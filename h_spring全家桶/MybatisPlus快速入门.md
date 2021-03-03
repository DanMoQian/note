# MybatisPlus

## 简介

[MyBatis-Plus (opens new window)](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis (opens new window)](http://www.mybatis.org/mybatis-3/)的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

## 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

## 快速入门

地址：https://baomidou.com/guide/quick-start.html

使用第三方组件

1. 导入对应的依赖
2. 研究依赖如何配置
3. 代码如何编写
4. 提高扩展技术能力

==这是3.4.1版本的与低版本有所区别==

> 步骤

1. 创建数据库  ==mybatis_plus==

2. 创建==user==表

   ```sql
   DROP TABLE IF EXISTS user;
   
   CREATE TABLE user
   (
   	id BIGINT(20) NOT NULL COMMENT '主键ID',
   	name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
   	age INT(11) NULL DEFAULT NULL COMMENT '年龄',
   	email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
   	PRIMARY KEY (id)
   );
   #真实开发中，version(乐观锁)、deleted(逻辑删除)、gmt_create/gmt_modified
   ```


3. 初始化工程 `springboot`

   添加依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <optional>true</optional>
       </dependency>
       <dependency>
           <groupId>com.baomidou</groupId>
           <artifactId>mybatis-plus-boot-starter</artifactId>
           <version>3.4.1</version>
       </dependency>
   </dependencies>
   ```

4.  配置

   在 `application.yml` 配置文件中添加 mysql的相关配置：

   ```yml
   spring:
     datasource:
       driver-class-name: com.mysql.cj.jdbc.Driver
       #mysql8 要写时区
       url: jdbc:mysql://localhost:3306/springboot?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8
       username: root
       passwor: root
   ```

5. 在 Spring Boot 启动类中添加 `@MapperScan` 注解，扫描 Mapper 文件夹：

   ```java
   @SpringBootApplication
   //注意只写到mapper包一级就可以了，不要写到子包
   @MapperScan("com.mycode.mapper")
   public class MybatisPlusApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(MybatisPlusApplication.class, args);
       }
   
   }
   ```

6. 编写pojo类

   ```java
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class User {
   
       //private Integer id;
       private Long id;//采坑：数据库用的是BigInt要使用Long才行！！！
       private String name;
       private Integer age;
       private String email;
   
   }
   ```

7. 编写mapper类

   ```java
   @Repository
   public interface UserMapper extends BaseMapper<User> {
   }
   ```

8. 编写测试类

   ```java
   @SpringBootTest
   class MybatisPlusApplicationTests {
   
       @Autowired
       private UserMapper userMapper;
   
       @Test
       void contextLoads() {
           List<User> users = userMapper.selectList(null);
           users.forEach(System.out::println);
       }
   
   }
   ```

9. ==结果==

   ![image-20201216234905499](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201216234905499.png)



## 配置日志

在application.yml中添加一条配置

```yml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

`结果`

![image-20201217000237861](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201217000237861.png)

## CURD

### insert

- 主键生成策略

#### [](https://baomidou.com/guide/annotation.html#idtype)[@IdType(opens new window)](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/IdType.java)

|      值       |                             描述                             |
| :-----------: | :----------------------------------------------------------: |
|    `AUTO`     |            数据库ID自增==数据库表必须设置自增长==            |
|     NONE      | 无状态,该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT) |
|     INPUT     |                    insert前自行set主键值                     |
|  `ASSIGN_ID`  | 分配ID(主键类型为Number(Long和Integer)或String)(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextId`(默认实现类为`DefaultIdentifierGenerator`雪花算法) |
|  ASSIGN_UUID  | 分配UUID,主键类型为String(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextUUID`(默认default方法) |
|   ID_WORKER   |     分布式全局唯一ID 长整型类型(please use `ASSIGN_ID`)      |
|     UUID      |           32位UUID字符串(please use `ASSIGN_UUID`)           |
| ID_WORKER_STR |     分布式全局唯一ID 字符串类型(please use `ASSIGN_ID`)      |

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

    @TableId(type = IdType.ASSIGN_ID)
    private Long id;
    private String name;
    private Integer age;
    private String email;

}
```

```java
@Test
void contextLoads() {
    User user = new User();
    user.setName("张三");
    user.setAge(20);
    user.setEmail("zhangsan@gmail.com");
    int insert = userMapper.insert(user);
    System.out.println(insert);
}
```

### update

```java
@Test
public void updateTest(){
    User user = new User();
    user.setId(5L);
    user.setName("张三");
    user.setAge(1);
    user.setEmail("zhangsan@gmail.com");
    userMapper.updateById(user);
}
```

#### 自动填充

创建时间、修改时间！自动完成、不要手动更新。 `gmt_create`、`gmt_modified`几乎所有的表都要配置上

> 方式一：数据库级别

新增两个字段`gmt_create`、`gmt_modified`

![image-20201217133632411](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201217133632411.png)

```java
@Test
public void updateTest(){
    User user = new User();
    user.setId(5L);
    user.setName("张三");
    user.setAge(278);
    user.setEmail("zhangsan@gmail.com");
    userMapper.updateById(user);
}
```

![image-20201217134403330](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201217134403330.png)

==坑：多次执行该代码不会执行时间戳更新 具体原因不清楚==

> 方式二：代码级别

==方法一不推荐使用==

使用注解

```java
//字段添加填充内容 
@TableField(fill = FieldFill.INSERT)//在插入的时候填充
private Date gmtCreate;
@TableField(fill = FieldFill.INSERT_UPDATE)//在插入和更新的的时候填充
private Date gmtModified;
```

```java
@Slf4j
@Component //一定要把处理器放到IOC容器中
public class MeMetaObjectHandler implements MetaObjectHandler {

    //插入时的填充策略
    @Override
    public void insertFill(MetaObject metaObject) {
      log.info("start insert fill");
      this.setFieldValByName("gmtCreate", new Date(), metaObject);
      this.setFieldValByName("gmtModified", new Date(), metaObject);
    }

    //更新时的填充数据
    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("gmtModified", new Date(), metaObject);
    }
}
```

执行插入和更新操作之后：

![image-20201217144727998](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201217144727998.png)

#### 乐观锁

> 乐观锁：开放，不会出现问题，出现问题再次更新值
>
> 悲观锁：悲观，都要上锁，再去操作

乐观锁

- 取出记录时，获取当前version
- 更新时，带上version
- 执行更新时，set version = newVersion where version = version + 1

- 如果version不对，就更新失败

```mysql
-- 初始version为1
update user set name=‘zhangsan’,version = version + 1
where id = 1 and version = 1;
```

> 实现乐观锁

1. 修改表及其实体类 添加version字段

![image-20201217151356501](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201217151356501.png)

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

    @TableId(type = IdType.ASSIGN_ID)
    private Long id;
    private String name;
    private Integer age;
    private String email;
    @TableField(fill = FieldFill.INSERT)
    private Date gmtCreate;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date gmtModified;
    
    
    @Version
    private Integer version;//添加的version字段
}
```

2. 配置乐观锁类

```java

//更新新版应该这样使用
@Configuration
@MapperScan("com.mycode.mapper")
@EnableTransactionManagement
public class MyBatisPlusConfig {

    /**
     * 乐观锁
     * 需要设置 MybatisConfiguration#useDeprecatedExecutor = false 避免缓存出现问题(该属性会在旧插件移除后一同移除)
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        //乐观锁插件配置
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }

    @Bean
    public ConfigurationCustomizer configurationCustomizer() {
        return configuration -> configuration.setUseDeprecatedExecutor(false);
    }

}

/*@Configuration
@MapperScan("com.mycode.mapper")
@EnableTransactionManagement
public class MyBatisPlusConfig {
	//OptimisticLockerInterceptor 注意这是一个过时类但是用新的会报错，暂时先用旧的
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor(){
        return new OptimisticLockerInterceptor();
    }
}*/

/*
这种用法是错误的
@Configuration
@MapperScan("com.mycode.mapper")
@EnableTransactionManagement
public class MyBatisPlusConfig {

    @Bean
    public OptimisticLockerInnerInterceptor optimisticLockerInterceptor(){
        return new OptimisticLockerInnerInterceptor();
    }
}*/
```

执行测试：

```
@Test
public void OptimisticLockerTest(){
    User user = userMapper.selectById(1339461711110885377L);
    System.out.println(user);
    user.setName("王麻子");
    user.setAge(53);
    userMapper.updateById(user);
}
```

==注意：必须先查询再更新才能使用乐观锁，直接更新是不会使用乐观锁==

![image-20201217155056770](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201217155056770.png)

多线程测试：

```java
@Test
public void OptimisticLockerTest2(){
    //模拟多线程
    //第一个线程先查询
    User user = userMapper.selectById(1339461711110885377L);
    System.out.println(user);
    user.setName("王麻子");
    user.setAge(53);
    //第二个多线程再执行查询并先更新
    User user2 = userMapper.selectById(1339461711110885377L);
    System.out.println(user2);
    user2.setName("王麻子大爷");
    user2.setAge(99);
    userMapper.updateById(user2);
    //最后更新第一个线程会失败
    //可以使用自旋锁
    userMapper.updateById(user);
}
```

![image-20201217155620988](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201217155620988.png)

### select

```java
//批量查询
@Test
public void selectTest(){
    List<User> users = userMapper.selectBatchIds(Arrays.asList(1, 2, 3, 4));
    users.forEach(System.out::println);
}
```

```
//条件查询
@Test
public void selectByMap(){
    Map<String, Object> map = new HashMap<>();
    map.put("name", "jack");
    List<User> users = userMapper.selectByMap(map);
    System.out.println(users);
}
```

#### 分页查询

内置分页插件

1. 配置

```java
@Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        //乐观锁
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        //分页插件
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        //......直接在这里面new对应的插件
        
        return interceptor;
    }
```

插件

- 自动分页: PaginationInnerInterceptor
- 多租户: TenantLineInnerInterceptor
- 动态表名: DynamicTableNameInnerInterceptor
- 乐观锁: OptimisticLockerInnerInterceptor
- sql性能规范: IllegalSQLInnerInterceptor
- 防止全表更新与删除: BlockAttackInnerInterceptor

2. 测试

```java
//分页查询
@Test
public void pageTest(){
    Page<User> page = new Page<>(2, 5);//参数一 当前页 参数二 页面大小
    userMapper.selectPage(page, null);
    page.getRecords().forEach(System.out::println);
}
```



![image-20201217164547310](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201217164547310.png)

### delete

.......

### 逻辑删除

1. 修改表和实体类

![image-20201217170542106](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201217170542106.png)

```java
@TableLogic//逻辑删除
private Integer deleted;
```

2. 添加配置

```yml
global-config:
  db-config:
    logic-delete-field: 1
    logic-not-delete-value: 0
```

3. 测试

```java
//逻辑删除
@Test
public void deletedByLogic(){
    userMapper.deleteById(1L);
    userMapper.selectById(1L);
}
```

![image-20201217171230708](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201217171230708.png)



## 性能分析插件

日常开发中，会遇到慢sql。 通过测试，druid....

mp执行 SQL 分析打印

👉 [mybatis-plus-sample-crud(opens new window)](https://gitee.com/baomidou/mybatis-plus-samples/tree/master/mybatis-plus-sample-crud)

- p6spy 依赖引入

```xml
<!-- p6spy -->
<dependency>
    <groupId>p6spy</groupId>
    <artifactId>p6spy</artifactId>
    <version>3.9.1</version>
</dependency>
```

- 修改配置文件

```yml
spring:
  datasource:
  #修改
    #driver-class-name: com.mysql.cj.jdbc.Driver
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver
    #mysql8 要写时区
    #url: jdbc:mysql://localhost:3306/mybatis_plus?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8
    url: jdbc:p6spy:mysql://localhost:3306/mybatis_plus?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8
    username: root
    password: root
```

```properties
#p6spy
#3.2.1以上使用
modulelist=com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory,com.p6spy.engine.outage.P6OutageFactory
#3.2.1以下使用或者不配置
#modulelist=com.p6spy.engine.logging.P6LogFactory,com.p6spy.engine.outage.P6OutageFactory
# 自定义日志打印
logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger
#日志输出到控制台
appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger
# 使用日志系统记录 sql
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
# 设置 p6spy driver 代理
deregisterdrivers=true
# 取消JDBC URL前缀
useprefix=true
# 配置记录 Log 例外,可去掉的结果集有error,info,batch,debug,statement,commit,rollback,result,resultset.
excludecategories=info,debug,result,commit,resultset
# 日期格式
dateformat=yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
#driverlist=org.h2.Driver
# 是否开启慢SQL记录
outagedetection=true
# 慢SQL记录标准 2 秒
outagedetectioninterval=2
```

## 条件构造器

文档：https://baomidou.com/guide/wrapper.html#and



```java
@Autowired
private UserMapper userMapper;

@Test
public void test1(){
    QueryWrapper<User> userWrapper = new QueryWrapper<>();
    userWrapper.eq("name", "张三");
    List<User> users = userMapper.selectList(userWrapper);
    System.out.println(users);
}

@Test
public void test2(){
    QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
    userQueryWrapper.inSql("id", "select id from user where id < 3");
    List<User> userList = userMapper.selectList(userQueryWrapper);
    userList.forEach(System.out::println);
}

@Test
public void test3(){
    QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
    userQueryWrapper.and(i->i.eq("name", "张三").eq("age", 278));
    List<User> userList = userMapper.selectList(userQueryWrapper);
    userList.forEach(System.out::println);
}
```

## 代码自动生成器

- 依赖

```xml
<!--代码生成器-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.28</version>
    <scope>compile</scope>
</dependency>
```

- 执行类

```java
package com.mycode;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

// 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
public class CodeGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("DMQi");
        gc.setOpen(false);
        gc.setSwagger2(true); //实体属性 Swagger2 注解
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/vhr?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("root");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(scanner("vhr"));
        pc.setParent("com.mycode");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/src/main/resources/mapper/" + pc.getModuleName()
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        /*
        cfg.setFileCreate(new IFileCreate() {
            @Override
            public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
                // 判断自定义文件夹是否需要创建
                checkDir("调用默认方法创建的目录，自定义目录用");
                if (fileType == FileType.MAPPER) {
                    // 已经生成 mapper 文件判断存在，不想重新生成返回 false
                    return !new File(filePath).exists();
                }
                // 允许生成模板文件
                return true;
            }
        });
        */
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        // 配置自定义输出模板
        //指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setSuperEntityClass("");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // 公共父类
        strategy.setSuperControllerClass("");
        // 写于父类中的公共字段
        strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }

}
```



# 源码阅读

## mybatis 字段策略

参考：[MyBatis-Plus 配置 field-strategy 属性详解](https://www.lichenliang.top/2019/03/mybatis-plus-global-config-field-strategy.html)

思考：mybatis如何插入一个null,或者如何实现如果插入的时候忽略掉null值。

MyBatis-Plus 在 SpringBoot 的中有个配置项 field-strategy 。官方说明如下：

> 该策略约定了如何产出注入的sql,涉及`insert`,`update`以及`wrapper`内部的`entity`属性生成的 where 条件

在下面这个包中自动设置了field-strategy

```java
package com.baomidou.mybatisplus.enums;
```

```java
package com.baomidou.mybatisplus.enums;

/**
 * <p>
 * 字段策略枚举类
 * </p>
 *
 * @author hubin
 * @Date 2016-09-09
 */
public enum FieldStrategy {
    IGNORED(0, "忽略判断"),
    NOT_NULL(1, "非 NULL 判断"),
    NOT_EMPTY(2, "非空判断");
    /**
     * 主键
     */
    private final int key;
    /**
     * 描述
     */
    private final String desc;
    FieldStrategy(final int key, final String desc) {
        this.key = key;
        this.desc = desc;
    }
    //重点根据不同的key进行选择策略：忽略判断、非 NULL 判断、非空判断
    //默认是非 NULL 判断
    //可以在类上面加strategy，例子如下面
    //@TableField(value = "user_name", strategy = FieldStrategy.IGNORED)
	//private String userName;
    public static FieldStrategy getFieldStrategy(int key) {
        FieldStrategy[] fss = FieldStrategy.values();
        for (FieldStrategy fs : fss) {
            if (fs.getKey() == key) {
                return fs;
            }
        }
        return FieldStrategy.NOT_NULL;
    }
    public int getKey() {
        return this.key;
    }
    public String getDesc() {
        return this.desc;
    }
}
```

关注这个可以发现很多东西

![image-20210129201001353](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20210129201001353.png)



