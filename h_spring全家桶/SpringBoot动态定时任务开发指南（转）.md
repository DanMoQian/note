# [SpringBoot动态定时任务开发指南（转）](http://wuwenliang.net/2019/12/03/SpringBoot动态定时任务开发指南/)

By [SnoWalker](http://wuwenliang.net/about)

链接：http://wuwenliang.net/2019/12/03/SpringBoot%E5%8A%A8%E6%80%81%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97/

一般情况下，如果想在Spring Boot中使用定时任务，我们只需要@EnableScheduling开启定时任务支持，在需要调度的方法上添加@Scheduled注解。这样就能够在项目中开启定时调度功能了，并且这种方法支持通过cron表达式灵活的控制执行周期和频率。

上述的方式好处是快捷，轻量，缺点是周期一旦指定，想要更改必须要重启应用，如果我们想要动态的对定时任务的执行周期进行变更，甚至动态的增加定时调度任务则上述方式就不适用了。

本文我将讲解如何在Spring 定时任务的基础上进行扩展，实现动态定时任务。



## 需求

1. 动态增加定时任务
2. 热更新定时任务的执行周期（动态更新cron表达式）

## 方案1：仅实现动态变更任务周期

> 首先介绍的方案1能够实现动态变更已有任务的执行频率/周期。

首先建立一个Spring Boot应用，这里不再展开。

建立一个任务调度类，实现接口SchedulingConfigurer，标记为Spring的一个bean。注意一定要添加注解 **@EnableScheduling** 开启定时任务支持。

```java
@EnableScheduling
@Component
public class DynamicCronHandler implements SchedulingConfigurer {

    private static final String DEFAULT_CRON = "0/5 * * * * ?";
    private String taskCron = DEFAULT_CRON;

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        scheduledTaskRegistrar.addTriggerTask(()->{
            LOGGER.info("执行任务");
        }, triggerContext -> {
            // 刷新cron
            CronTrigger cronTrigger = new CronTrigger(taskCron);
            Date nextExecDate = cronTrigger.nextExecutionTime(triggerContext);
            return nextExecDate;
        });
    }
```

scheduledTaskRegistrar.addTriggerTask接受两个参数，分别为需要调度的任务实例（Runnable实例），Trigger实例，这里通过lambda方式注入，需要实现nextExecutionTime回调方法，返回下次执行时间。

通过该回调方法，在Runnable中执行业务逻辑代码，在Trigger修改定时任务的执行周期。

```
    public DynamicCronHandler setTaskImplement(Runnable taskImplement) {
        this.taskImplement = taskImplement;
        return this;
    }

    public DynamicCronHandler setTaskCron(String taskCron) {
        this.taskCron = taskCron;
        return this;
    }

    public DynamicCronHandler taskCron(String taskCron) {
        System.out.println("更新cron=" + taskCron);
        this.taskCron = taskCron;
        return this;
    }

    ...省略getter...

}
```

编写一个测试类，进行测试：

```
@RequestMapping("execute")
@ResponseBody
public String executeTask(@RequestParam(value = "cron", defaultValue = "0/10 * * * * ?") String cron) {
    LOGGER.info("cron={}", cron);
    dynamicCronHandler.taskCron(cron);
    return "success";
}
```

暴露一个http接口，接受参数cron，启动应用并访问/execute，首次传入参数cron=0/1 ?，表示每秒执行一次任务。日志如下：

```
2019-12-03 15:32:40.001  INFO 7196 --- [TaskScheduler-1] c.s.s.t.d.d.DynamicCronHandler           : 执行任务
2019-12-03 15:32:41.001  INFO 7196 --- [TaskScheduler-1] c.s.s.t.d.d.DynamicCronHandler           : 执行任务
2019-12-03 15:32:42.001  INFO 7196 --- [TaskScheduler-1] c.s.s.t.d.d.DynamicCronHandler           : 执行任务
2019-12-03 15:32:43.001  INFO 7196 --- [TaskScheduler-1] c.s.s.t.d.d.DynamicCronHandler           : 执行任务
2019-12-03 15:32:44.001  INFO 7196 --- [TaskScheduler-1] c.s.s.t.d.d.DynamicCronHandler           : 执行任务
```

可以看到每秒执行一次。

更改cron的值为0/5 ?，观察到控制台输出发生变化：

```
更新cron=0/5 * * * * ?
2019-12-03 15:33:30.001  INFO 7196 --- [TaskScheduler-1] c.s.s.t.d.d.DynamicCronHandler           : 执行任务
2019-12-03 15:33:35.001  INFO 7196 --- [TaskScheduler-1] c.s.s.t.d.d.DynamicCronHandler           : 执行任务
2019-12-03 15:33:40.001  INFO 7196 --- [TaskScheduler-1] c.s.s.t.d.d.DynamicCronHandler           : 执行任务
```

此时定时任务执行频率更新为5秒一次，表明通过SchedulingConfigurer.configureTasks回调，动态的更新了定时任务执行频率。

### 思考

到目前为止，实现了动态变更定时任务的执行频率，但是不能实现动态的提交定时任务。方案二就是为了解决这个疑问而实现的，

## 方案二：动态提交定时任务并更新任务执行频率

首先建立一个DynamicTaskScheduler类，内容如下：

```
@Scope(value = "singleton")
@Component
@EnableScheduling
public class DynamicTaskScheduler {

    private ScheduledFuture<?> future;

    @Autowired
    private ThreadPoolTaskScheduler threadPoolTaskScheduler;

    @Bean
    public ThreadPoolTaskScheduler threadPoolTaskScheduler() {
        return new ThreadPoolTaskScheduler();
    }

    public void startCron(Runnable task, String cron) {
        stopCron();
        future = threadPoolTaskScheduler.schedule(
                task, new CronTrigger(cron)
        );
    }

    public void stopCron() {
        if (future != null) {
            future.cancel(true);
            System.out.println("stopCron()");
        }
    }
}
```

这里通过startCron提交一个新的任务，通过cron表达式进行调度，在开始之前进行判断是否关闭老的，必须关闭老的才能开启新的。

通过stopCron对老任务进行关闭。

编写一个测试方法测试该动态任务调度类。

```
@RequestMapping("execute1")
@ResponseBody
public String executeTask1(@RequestParam(value = "cron", defaultValue = "0/10 * * * * ?") String cron) {
    LOGGER.info("cron={}", cron);
    dynamicTaskScheduler.startCron(
            () -> {
                LOGGER.info("模拟执行作业,cron={}", cron);
            },
            cron
    );
    return "success";
}
```

启动方法中初始化一个 ThreadPoolTaskScheduler 实例。

```
@SpringBootApplication
public class SnowalkerTestDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SnowalkerTestDemoApplication.class, args);
    }

    @Bean
    public ThreadPoolTaskScheduler threadPoolTaskScheduler() {
        ThreadPoolTaskScheduler executor = new ThreadPoolTaskScheduler();
        executor.setPoolSize(20);
        executor.setThreadNamePrefix("taskExecutor-");
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(60);
        return executor;
    }

}
```

运行启动类，访问测试接口/execute1，先传入cron=0/1 ?，表示每秒执行一次任务。日志如下：

更改cron的值为0/5 ?，观察到控制台输出发生变化：

可以看到这种方式同样实现了动态的变更定时任务执行频率，相比上述的方法，该方式更加灵活，能够动态的增加任务到线程池中进行调度，我们可以定义一个Map保存future，从而实现创建并维护多个定时任务，具体可以参考这篇文章 [ThreadPoolTaskScheduler的使用，定时任务开启与关闭](https://blog.csdn.net/qq_32711309/article/details/84944534) ，思路如下：

1. 自定义Task类，实现Runnable，定义属性name
2. 定义一个ConcurrentHashMap,KEY=name,value=ScheduledFuture
3. 通过 **ScheduledFuture<?> schedule(Runnable task, Trigger trigger)** 进行任务调度时，传入自定义Task，构造/setter 注入任务名称（全局唯一）, 并将该task实现类设置到步骤2的map中，key=name，value=当前通过schedule调度返回的ScheduledFuture
4. 停止该任务时，通过name在map中找到ScheduledFuture实例，调用scheduledFuture.cancel(true);方法停止任务即可

核心代码如下：

> 任务存储Map（启动类 成员变量）一启动就初始化

```
public static ConcurrentHashMap<String, ScheduledFuture> map = new ConcurrentHashMap<>();
```

> 启动任务

```
@Component
@Scope("prototype")
public class DynamicTask {


    @Autowired
    private ThreadPoolTaskScheduler threadPoolTaskScheduler;
    private ScheduledFuture future;

    public void startCron() {
        cron = "0/1 * * * * ?";
        System.out.println(Thread.currentThread().getName());
        String name = Thread.currentThread().getName();
        future = threadPoolTaskScheduler.schedule(new myTask(name), new CronTrigger(cron));
        App.map.put(name, future);
    }
```

> 停止任务

```
    public void stop() {
        if (future != null) {
            future.cancel(true);
        }
    }
}
```

> 自定义Task定义

```
public class MyTask implements Runnable {
    private String name;

    myTask(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println("test" + name);
    }
}
```

> 测试接口

```
@Autowired
private DynamicTask task;

@RequestMapping("/start")
public void test() throws Exception {
    // 开启定时任务，对象注解Scope是多利的。
    task.startCron();

}

@RequestMapping("/stop")
public void stop() throws Exception {
    // 提前测试用来测试线程1进行对比是否关闭。
    ScheduledFuture scheduledFuture = App.map.get("http-nio-8081-exec-2");
    scheduledFuture.cancel(true);
    // 查看任务是否在正常执行之前结束,正常返回true
    boolean cancelled = scheduledFuture.isCancelled();
    while (!cancelled) {
        scheduledFuture.cancel(true);
    }
}
```

```java
package ltd.dmqi.task;

import ltd.dmqi.TaskApplication;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Scope;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
import org.springframework.scheduling.support.CronTrigger;
import org.springframework.stereotype.Component;

import java.util.concurrent.ScheduledFuture;

@Component
@Scope("prototype")
public class DynamicTask {
 
    private static final Logger logger = LoggerFactory.getLogger(DynamicTask.class);

    private String cron;
    @Autowired
    private ThreadPoolTaskScheduler threadPoolTaskScheduler;
    private ScheduledFuture future;
 
    public void startCron() {
        cron = "0/1 * * * * ?";
        System.out.println(Thread.currentThread().getName());
        String name = Thread.currentThread().getName();
        future = threadPoolTaskScheduler.schedule(new myTask(name), new CronTrigger(cron));
        TaskApplication.map.put(name, future);
    }
 
    public void stop() {
        if (future != null) {
            future.cancel(true);
        }
    }
 
    private class myTask implements Runnable {
        private String name;
 
        myTask(String name) {
            this.name = name;
        }
 
        @Override
        public void run() {
            System.out.println("test" + name);
        }
    }
 
}
```

## 小结

以上就是SpringBoot动态定时任务相关的讲解，这种方式在轻量级环境下能够很好的工作。如果我们的定时任务要求分布式，高可用，则需要引入额外的组件，如果有必要则需要引入如ejob，xxl-job，quartz等定时调度组件。

## 参考资料

[SpringBoot中并发定时任务的实现、动态定时任务的实现](https://segmentfault.com/a/1190000018788967)

[使用ThreadPoolTaskScheduler实现定时关单](https://my.oschina.net/kevin2kelly/blog/1548237)

[ThreadPoolTaskScheduler的使用，定时任务开启与关闭](https://blog.csdn.net/qq_32711309/article/details/84944534)