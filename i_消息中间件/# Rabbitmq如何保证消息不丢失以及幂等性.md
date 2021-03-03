# # [Rabbitmq如何保证消息不丢失](https://www.cnblogs.com/-wenli/p/13046535.html)以及幂等性

### 1.mq原则

数据不能多，也不能少，不能多是说消息不能重复消费；不能少，就是说不能丢失数据。如果mq传递的是非常核心的消息，支撑核心的业务，那么这种场景是一定不能丢失数据的。

### 2.丢失数据场景

丢数据一般分为三种，一种是mq把消息丢了，一种就是消费时将消息丢了。下面从rabbitmq和kafka分别说一下，丢失数据的场景，
**A:生产者弄丢了数据**
生产者将数据发送到rabbitmq的时候，可能在传输过程中因为网络等问题而将数据弄丢了。
**B:rabbitmq自己丢了数据**
如果没有开启rabbitmq的持久化，那么rabbitmq一旦重启，那么数据就丢了。所依必须开启持久化将消息持久化到磁盘，这样就算rabbitmq挂了，恢复之后会自动读取之前存储的数据，一般数据不会丢失。除非极其罕见的情况，rabbitmq还没来得及持久化自己就挂了，这样可能导致一部分数据丢失。
**C：消费端弄丢了数据**
如果一个消费者应用在消费的时候，刚消费到，还没处理,如进程挂了，比如重启了，rabbitmq认为你都消费了，这数据就丢了。

### 3.如何解决

**A:生产者丢失消息**

①：可以选择使用rabbitmq提供是事物功能，就是生产者在发送数据之前开启事物，然后发送消息，如果消息没有成功被rabbitmq接收到，那么生产者会受到异常报错，这时就可以回滚事物，然后尝试重新发送；如果收到了消息，那么就可以提交事物。

特别说明：AMQP 协议中的事务仅仅是指生产者发送消息给 broker 这一系列流程处理的事务机制，并不包含消费端的处理流程。

```java
  channel.txSelect();//开启事物
  try{
      //发送消息
  }catch(Exection e){
      channel.txRollback()；//回滚事物
      //重新提交
  }
```

**缺点：**rabbitmq事物已开启，就会变为同步阻塞操作，生产者会阻塞等待是否发送成功，太耗性能会造成吞吐量的下降。

②：可以开启confirm模式。在生产者哪里设置开启了confirm模式之后，每次写的消息都会分配一个唯一的id，然后如何写入了rabbitmq之中，rabbitmq会给你回传一个ack消息，告诉你这个消息发送OK了；如果rabbitmq没能处理这个消息，会回调你一个nack接口，告诉你这个消息失败了，你可以进行重试。而且你可以结合这个机制知道自己在内存里维护每个消息的id，如果超过一定时间还没接收到这个消息的回调，那么你可以进行重发。

```yml

rabbitmq:
    username: guest
    password: guest
    host: 127.0.0.1
    #在消息生产者这一端开启开启确认回调
    publisher-confirms: true
    publisher-returns: true
```



```java
    //开启confirm
    channel.confirm();
    //发送成功回调
    public void ack(String messageId){
      
    }

    // 发送失败回调
    public void nack(String messageId){
        //重发该消息
    }
```

```java
package org.javaboy.vhr.config;
@Configuration
public class RabbitConfig {
    public final static Logger logger = LoggerFactory.getLogger(RabbitConfig.class);
    @Autowired
    CachingConnectionFactory cachingConnectionFactory;
    @Autowired
    MailSendLogService mailSendLogService;

    @Bean
    RabbitTemplate rabbitTemplate() {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(cachingConnectionFactory);
        //消息发送成功回调
        rabbitTemplate.setConfirmCallback((data, ack, cause) -> {
            String msgId = data.getId();
            if (ack) {
                logger.info(msgId + ":消息发送成功");
                mailSendLogService.updateMailSendLogStatus(msgId, 1);//修改数据库中的记录，消息投递成功
            } else {
                logger.info(msgId + ":消息发送失败");
            }
        });
        //消息发送失败回调
        rabbitTemplate.setReturnCallback((msg, repCode, repText, exchange, routingkey) -> {
            logger.info("消息发送失败");
        });
        return rabbitTemplate;
    }

    //入职邮件发送
    @Bean
    Queue mailQueue() {
        return new Queue(MailConstants.MAIL_QUEUE_NAME, true);
    }

    @Bean
    DirectExchange mailExchange() {
        return new DirectExchange(MailConstants.MAIL_EXCHANGE_NAME, true, false);
    }

    @Bean
    Binding mailBinding() {
        return BindingBuilder.bind(mailQueue()).to(mailExchange()).with(MailConstants.MAIL_ROUTING_KEY_NAME);
    }

    //奖惩邮件发送
    @Bean
    Queue empecMailQueue() {
        return new Queue(MailConstants.EMPEC_MAIL_QUEUE_NAME, true);
    }
    @Bean
    DirectExchange empecMailExchange() {
        return new DirectExchange(MailConstants.EMPEC_MAIL_EXCHANGE_NAME, true, false);
    }
    @Bean
    Binding empecMailBinding() {
        return BindingBuilder.bind(empecMailQueue()).to(empecMailExchange()).with(MailConstants.EMPEC_MAIL_ROUTING_KEY_NAME);
    }
}
```

**二者不同**
事务机制是同步的，你提交了一个事物之后会阻塞住，但是confirm机制是异步的，发送消息之后可以接着发送下一个消息，然后rabbitmq会回调告知成功与否。
一般在生产者这块避免丢失，都是用confirm机制。
**B:rabbitmq自己弄丢了数据**
设置消息持久化到磁盘。设置持久化有两个步骤：
①创建queue的时候将其设置为持久化的，这样就可以保证rabbitmq持久化queue的元数据，但是不会持久化queue里面的数据。
②发送消息的时候讲消息的deliveryMode设置为2，这样消息就会被设为持久化方式，此时rabbitmq就会将消息持久化到磁盘上。
必须要同时开启这两个才可以。

而且持久化可以跟生产的confirm机制配合起来，只有消息持久化到了磁盘之后，才会通知生产者ack，这样就算是在持久化之前rabbitmq挂了，数据丢了，生产者收不到ack回调也会进行消息重发。
**C:消费者弄丢了数据**

rabbitmq有手动ack机制与自动ack机制来解决消费者弄丢数据：
如果使用rabbitmq提供的ack机制，首先关闭rabbitmq的自动ack，使用手动ack，每次在确保处理完这个消息之后，在代码里手动调用ack。这样就可以避免消息还没有处理完就ack。

但是ack机制在异常情况下可能造成重复消费：当消费者异常断掉连接，但并未挂掉，broker 会得知， 此时broker 尚未获得 ack，那么消息会被重新放入其他队列，这样就导致数据被重复消费了。

应用层解决重复的方式：

- **1.** 专门的 Map 存储：用来存储每个消息的执行状态（用 msgid 区分），执行成功之后更新 Map，有另外消息重复消费的时候，读取 Map 数据判断 msgid 对应的执行状态，已消费则不执行。
- **2.** 业务逻辑判断：消息执行完会更改某个实体状态，判断实体状态是否更新，如果更新，则不进行重复消费。

总结：AMQP 提供的是“**至少一次交付**”（at-least-once delivery），异常情况下，消息会被重复消费，此时业务要实现幂等性（重复消息处理）



## 幂等性

```java
package org.javaboy.mailserver.receiver;
@Component
public class MailReceiver {

    public static final Logger logger = LoggerFactory.getLogger(MailReceiver.class);

    @Autowired
    JavaMailSender javaMailSender;
    @Autowired
    MailProperties mailProperties;
    @Autowired
    TemplateEngine templateEngine;
    @Autowired
    StringRedisTemplate redisTemplate;

    @RabbitListener(queues = MailConstants.MAIL_QUEUE_NAME)
    public void handler(Message message, Channel channel) throws IOException {
        Employee employee = (Employee) message.getPayload();
        MessageHeaders headers = message.getHeaders();
        Long tag = (Long) headers.get(AmqpHeaders.DELIVERY_TAG);
        String msgId = (String) headers.get("spring_returned_message_correlation");
        //处理等幂信问题
        if (redisTemplate.opsForHash().entries("mail_log").containsKey(msgId)) {
            //redis 中包含该 key，说明该消息已经被消费过
            logger.info(msgId + ":消息已经被消费");
            channel.basicAck(tag, false);//确认消息已消费
            return;
        }
        //收到消息，发送邮件
        MimeMessage msg = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(msg);
        try {
            helper.setTo(employee.getEmail());
            helper.setFrom(mailProperties.getUsername());
            helper.setSubject("入职欢迎");
            helper.setSentDate(new Date());
            Context context = new Context();
            context.setVariable("name", employee.getName());
            context.setVariable("posName", employee.getPosition().getName());
            context.setVariable("joblevelName", employee.getJobLevel().getName());
            context.setVariable("departmentName", employee.getDepartment().getName());
            String mail = templateEngine.process("mail", context);
            helper.setText(mail, true);
            javaMailSender.send(msg);
            redisTemplate.opsForHash().put("mail_log", msgId, "javaboy");
            channel.basicAck(tag, false);
            logger.info(msgId + ":邮件发送成功");
        } catch (MessagingException e) {
            channel.basicNack(tag, false, true);
            e.printStackTrace();
            logger.error("邮件发送失败：" + e.getMessage());
        }
    }
}
```