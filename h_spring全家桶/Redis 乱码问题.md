Redis 乱码问题

```redis
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {


    @Bean(name="redisTemplate")
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, String> template = new RedisTemplate<>();
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        template.setConnectionFactory(factory);
        //key序列化方式
        template.setKeySerializer(redisSerializer);
        //value序列化
        template.setValueSerializer(redisSerializer);
        //value hashMap序列化
        template.setHashValueSerializer(redisSerializer);
        //key hashMap序列化
        template.setHashKeySerializer(redisSerializer);

        return template;
    }
}
```

> 序列化存储的值 但是无法存基本数字类型



```java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.*;
import org.springframework.data.redis.serializer.JdkSerializationRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {
/**
 * 注入 RedisConnectionFactory
 */
@Autowired
RedisConnectionFactory redisConnectionFactory;

@Bean
public RedisTemplate<String,Object> functionDomainRedisTemplate() {
    RedisTemplate<String,Object> redisTemplate = new RedisTemplate<>();
    initDomainRedisTemplate(redisTemplate, redisConnectionFactory);
    return redisTemplate;
}

/**
 * 设置数据存入 redis 的序列化方式
 *
 * @param redisTemplate
 * @param factory
 */
private void initDomainRedisTemplate(RedisTemplate<String,Object> redisTemplate, RedisConnectionFactory factory) {
    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.setHashKeySerializer(new StringRedisSerializer());
    redisTemplate.setHashValueSerializer(new JdkSerializationRedisSerializer());
    redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());
    redisTemplate.setConnectionFactory(factory);
}
/**
 * 实例化 HashOperations 对象,可以使用 Hash 类型操作
 *
 * @param redisTemplate
 * @return
 */
@Bean
public HashOperations<String, String, Object> hashOperations(RedisTemplate<String, Object> redisTemplate) {
    return redisTemplate.opsForHash();
}

/**
 * 实例化 ValueOperations 对象,可以使用 String 操作
 *
 * @param redisTemplate
 * @return
 */
@Bean
public ValueOperations<String, Object> valueOperations(RedisTemplate<String, Object> redisTemplate) {
    return redisTemplate.opsForValue();
}

/**
 * 实例化 ListOperations 对象,可以使用 List 操作
 *
 * @param redisTemplate
 * @return
 */
@Bean
public ListOperations<String, Object> listOperations(RedisTemplate<String, Object> redisTemplate) {
    return redisTemplate.opsForList();

}

/**
 * 实例化 SetOperations 对象,可以使用 Set 操作
 *
 * @param redisTemplate
 * @return
 */
@Bean
public SetOperations<String, Object> setOperations(RedisTemplate<String, Object> redisTemplate) {
    return redisTemplate.opsForSet();
}

/**
 * 实例化 ZSetOperations 对象,可以使用 ZSet 操作
 *
 * @param redisTemplate
 * @return
 */
@Bean
public ZSetOperations<String, Object> zSetOperations(RedisTemplate<String, Object> redisTemplate) {
    return redisTemplate.opsForZSet();
	}
}
```


### 总结

不建议更换redisTemplate默认的序列化策略，有乱码就让它乱着吧，反正知道正确的解码策略就不会影响程序的正常运行（不过通过php等其他语言去获取redis的值貌似不太好解决）

如果一定要更换策略，那么前往要注意，存储数据的类型要根据所选择的序列化策略去进行切换

项目案例源代码：[github/booklet-redis](https://github.com/liumapp/booklet/tree/master/booklet-redis)


作者：liumapp
链接：https://juejin.cn/post/6844903907244638222
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。