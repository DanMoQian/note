# SpringCloud

## 配置

版本号以伦敦地铁站命名 且首字母越大版本也越大 ga代表稳定版

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mycode</groupId>
    <artifactId>springCloud</artifactId>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>springcloud-api</module>
        <module>springcloud-provider-dept-9001</module>
        <module>springcloud-consumer-dept-80</module>
        <module>springcloud-eureka-7001</module>
    </modules>

    <packaging>pom</packaging>

    <properties>
        <junit.version>4.12</junit.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-dependencies -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR9</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.7.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--数据库-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.47</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.1.23</version>
            </dependency>
            <!--springboot启动器-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter</artifactId>
                <version>2.4.1</version>
            </dependency>
            <!--mybatis-->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>2.1.4</version>
            </dependency>
            <!--junit-->
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.12</version>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>1.2.17</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
                <version>1.2.3</version>
            </dependency>
            <!--lombok-->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.18.16</version>
            </dependency>
        </dependencies>
    </dependencyManagement>


</project>
```

## RESTTemplate

http请求的组件

- 配置类

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

}
```

- 使用

```java
@RestController
public class DeptController {

    @Autowired
    private RestTemplate restTemplate;

    private static final String DEPT_PROVIDE_URL_PRE = "http://localhost:9001/dept";

    //http://localhost:9001/dept/query/list

    @PostMapping("/add")
    public Boolean addDept(@RequestBody Dept dept){
        return restTemplate.postForObject(DEPT_PROVIDE_URL_PRE + "/add", dept, Boolean.class);
    }

    @GetMapping("/query/{id}")
    public Dept queryById(@PathVariable("id") Long id){
        return restTemplate.getForObject(DEPT_PROVIDE_URL_PRE + "/query/"+id, Dept.class);
    }

    @GetMapping("/query/list")
    public List<Dept> queryAll(){
        return restTemplate.getForObject(DEPT_PROVIDE_URL_PRE + "/query/list", List.class);
    }
}
```

## Eureka

### 服务注册中心

- 导包

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

- 配置

```yml
server:
  port: 7001
  #Eureka配置
eureka:
  instance:
    # 服务端的实例名称
    hostname: localhost
  client:
    # 表示是否向服务中心注册自己
    register-with-eureka: false
    # false 表示自己就是注册中心
    fetch-registry: false
    # 服务地址 及监控页面
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

### 服务提供者

- 导包

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>

```

- 配置

```yml
#注册到eureka
eureka:
  client:
    service-url:
    # 注意defaultZone要小写不能大写 源码里面大写是常量
      defaultZone: http://localhost:7001/eureka
```

- 开启服务注册

```java
@SpringBootApplication
@EnableEurekaClient//开启取eureka注册
public class springCloudProviderDept {
    public static void main(String[] args) {
        SpringApplication.run(springCloudProviderDept.class, args);
    }
}
```

- 监控
  - 导包

```xml
<!--actuator监控-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

			- - 配置

```yml
info:
  app.name: danpeng-springcloud
  company.name: www.baidu.com
```

- - discovery

  ```java
  @Autowired
  private DiscoveryClient client;
  
  /**
       * 服务注册与发现
       */
      @GetMapping("/discovery")
      public Object discovery(){
          List<String> services = client.getServices();
          logger.info("dept-provider-->"+ services.toString());
  
          List<ServiceInstance> instances = client.getInstances("SPRINGCLOUD-PROVIDER-DEPT");
          for (ServiceInstance instance : instances) {
              logger.info(instance.getUri().toString());
              logger.info(instance.getPort()+"");
              logger.info(instance.getHost());
              logger.info(instance.getServiceId());
          }
  
          return this.client;
      }
  ```

## ribbon

### 服务消费方以及负载均衡

ribbon负载均衡

![image-20201223195557431](C:\Users\DMQi\AppData\Roaming\Typora\typora-user-images\image-20201223195557431.png)

- 导包

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
<!--erueka-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

- 配置

```yml
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka/, http://127.0.0.1:7002/eureka/, http://127.0.0.1:7003/eureka/
    # 不注册自己
    register-with-eureka: false
```

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    //负载均衡
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

}
```

```java
private static final String DEPT_PROVIDE_URL_PRE = "http://SPRINGCLOUD-PROVIDER-DEPT";//负载均衡,在注册中心去拿访问的地址
```

#### 均衡策略

``AbstractLoadBalancerRule``抽象类

![image-20201224150758207](C:\Users\DMQi\AppData\Roaming\Typora\typora-user-images\image-20201224150758207.png)

- 轮询（默认）
- 随机
- 过滤掉挂掉的服务器
- ------等等

自定义负载均衡

```java
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RuleConfig {
    
    @Bean
    public IRule iRule(){
        return new MyRule();
    }

}
```

```java
package com.config;


import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.Server;

import java.util.List;
import java.util.concurrent.ThreadLocalRandom;

/**
 * 负载均衡策略
 * 轮询然后服务连续提供5次
 */
public class MyRule extends AbstractLoadBalancerRule {

    private int count = 0;

    private int curIndex = 0;
    
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;

        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            List<Server> upList = lb.getReachableServers();
            List<Server> allList = lb.getAllServers();

            int serverCount = allList.size();
            if (serverCount == 0) {
                /*
                 * No servers. End regardless of pass, because subsequent passes
                 * only get more restrictive.
                 */
                return null;
            }

            //int index = chooseRandomInt(serverCount);
            //server = upList.get(index);
            //====================================================
            if (count < 5){
                server = upList.get(curIndex);
                count++;
            }else {
                count = 0;
                curIndex++;
                if (curIndex >= upList.size()){
                    curIndex = 0;
                }
                server = upList.get(curIndex);
            }
            //=====================================================


            if (server == null) {
                /*
                 * The only time this should happen is if the server list were
                 * somehow trimmed. This is a transient condition. Retry after
                 * yielding.
                 */
                Thread.yield();
                continue;
            }

            if (server.isAlive()) {
                return (server);
            }

            // Shouldn't actually happen.. but must be transient or a bug.
            server = null;
            Thread.yield();
        }

        return server;

    }

    protected int chooseRandomInt(int serverCount) {
        return ThreadLocalRandom.current().nextInt(serverCount);
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
        // TODO Auto-generated method stub

    }
}

```

## feign

社区版 ribbon

用注解和接口来配置 类似zoopkeper+Dobble



## hystrix

When everything is healthy the request flow can look like this:

![img](https://github.com/Netflix/Hystrix/wiki/images/soa-1-640.png)

When one of many backend systems becomes latent it can block the entire user request:

![img](https://github.com/Netflix/Hystrix/wiki/images/soa-2-640.png)

With high volume traffic a single backend dependency becoming latent can cause all resources to become saturated in seconds on all servers.

Every point in an application that reaches out over the network or into a client library that might result in network requests is a source of potential failure. Worse than failures, these applications can also result in increased latencies between services, which backs up queues, threads, and other system resources causing even more cascading failures across the system.

![img](https://github.com/Netflix/Hystrix/wiki/images/soa-3-640.png)

These issues are exacerbated when network access is performed through a third-party client — a “black box” where implementation details are hidden and can change at any time, and network or resource configurations are different for each client library and often difficult to monitor and change.

Even worse are transitive dependencies that perform potentially expensive or fault-prone network calls without being explicitly invoked by the application.

Network connections fail or degrade. Services and servers fail or become slow. New libraries or service deployments change behavior or performance characteristics. Client libraries have bugs.

All of these represent failure and latency that needs to be isolated and managed so that a single failing dependency can’t take down an entire application or system.



- 导包

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

- 注解使用

1. 对某个方法

```java
@GetMapping("/query/{id}")
@HystrixCommand(fallbackMethod = "hystrixQueryById")
public Dept queryById(@PathVariable("id") Long id){
    Dept dept = deptService.queryById(id);
    if (dept == null){
        throw  new RuntimeException("没有找到");
    }
    return dept;
}
public Dept hystrixQueryById(@PathVariable("id") Long id){
        return new Dept()
                .setDeptno(id)
                .setDname("no name")
                .setDbSource("no such datasource");
}
```

2. 对整个类：

----------google

> 服务熔断： 服务端 	某个服务超时或者异常，引起熔断

> 服务降级：客户端		再整体负载考虑， 将部分服务停掉，再客户端提供默认值