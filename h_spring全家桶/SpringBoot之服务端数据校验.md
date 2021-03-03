# [SpringBoot之服务端数据校验](https://www.cnblogs.com/myitnews/p/12348454.html)

对于任何一个应用而言，客户端做的数据有效性验证都不是安全有效的，而数据验证又是一个企业级项目架构上最为基础的功能模块，这时候就要求我们在服务端接收到数据的时候也对数据的有效性进行验证。为什么这么说呢？往往我们在编写程序的时候都会感觉后台的验证无关紧要，毕竟客户端已经做过验证了，后端没必要在浪费资源对数据进行验证了，但恰恰是这种思维最为容易被别人钻空子。毕竟只要有点开发经验的都知道，我们完全可以模拟 HTTP 请求到后台地址，模拟请求过程中发送一些涉及系统安全的数据到后台，后果可想而知....

验证分两种：对封装的Bean进行验证  或者  对方法简单参数的验证。

## 封装的Bean进行验证 

**说明：SpringBoot 中使用了 Hibernate-validate 校验框架作为支持**

### 一、引入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
<!-- 这两个springboot包里面都包含hibernate-validator包，只要引入一个即可 -->
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 二、hibernate-validator常用注解

- @Valid：被注释的元素是一个对象，需要检查此对象的所有字段值
- @Validated ：是@Valid 的一次封装，是Spring提供的校验机制使用。@Valid不提供分组功能@Null 被注释的元素必须为 null
- `@NotNull`：被注释的元素必须不为 null   `除了字符串内置类型判断，最好加上这个注解防止null导致不进行其他判断`
- @Pattern(value) ：被注释的元素必须符合指定的正则表达式
- @Size(min, max) ：集合元素的数量必须在min和max之间
- @CreditCardNumber(ignoreNonDigitCharacters=)： 字符串必须是信用卡号，按照美国的标准验证
- @Email： 字符串必须是Email地址
- @Length(min, max) ：检查字符串的长度
- `@NotBlank` ： 只能用于字符串不为null，并且字符串trim()以后length要大于0
- @NotEmpty ： 字符串不能为null, 集合必须有元素
- @Range(min, max) ：数字必须大于min, 小于max
- @SafeHtml(whitelistType=,additionalTags=) ：字符串必须是安全的html, classpath中要有jsoup包
- @URL(protocol=,host=, port=, regexp=, flags=) ： 字符串必须是合法的URL
- @AssertTrue ：被注释的元素必须为 true
- @AssertFalse ：被注释的元素必须为 false
- @DecimalMax(value=, inclusive=) ：值必须小于等于(inclusive=true)/小于(inclusive=false)属性指定的值，也可以注释在字符串类型的属性上。
- @DecimalMin(value=, inclusive=) ：值必须大于等于(inclusive=true)/小于(inclusive=false)属性指定的值，也可以注释在字符串类型的属性上。
- @Digits (integer, fraction) ：数字格式检查。integer指定整数部分的最大长度，fraction指定小数部分的最大长度
- @Future ： 时间必须是未来的
- @Past ： 时间必须是过去的
- @Max(value=) ： 值必须小于等于value指定的值。不能注解在字符串类型属性上。
- @Min(value=) ： 值必须小于等于value指定的值。不能注解在字符串类型属性上。
- @ScriptAssert(lang=, script=, alias=) ： 要有Java Scripting API 即JSR 223("Scripting for the JavaTM Platform")的实现

### 三、@Valid和@Validated的区别

@Valid是使用Hibernate validation的时候。

@Validated是使用Spring Validator校验机制（在spring-context依赖下）。

java的JSR303声明了@Valid这类接口，而Hibernate-validator对其进行了实现，@Validation对@Valid进行了二次封装，在使用上并没有区别，但在分组、注解位置、嵌套验证等功能上有所不同。

**1. 注解位置上**

@Validated：用在类型、方法和方法参数上。但不能用于成员属性（field）。

@Valid：可以用在方法、构造函数、方法参数和成员属性（field）上。

如果@Validated注解在成员属性上，则会报 不适用于field错误。

**2. 分组校验**

@Validated：`提供分组功能`，可以在参数验证时，根据不同的分组采用不同的验证机制。

@Valid：`没有分组功能`。

(1) 定义分组接口

```
public interface IGroupA extends Default {
}

public interface IGroupB extends Default{
}
```

> 注意:在声明分组的时候尽量加上 extend javax.validation.groups.Default 否则,在你声明@Validated(Update.class)的时候,就会出现你在默认没添加groups = {}的时候的校验组@Email(message = "邮箱格式不对"),会不去校验,因为默认的校验组是groups = {Default.class}。

(2) 定义需要校验的参数bean

```
public class Student implements Serializable {
    @NotBlank(message = "用户名不能为空")
    private String name;
    //只在分组为IGroupB的情况下进行验证
    @Min(value = 18, message = "年龄不能小于18岁", groups = {IGroupB.class})
    private Integer age;
    @Pattern(regexp = "^((13[0-9])|(14[5,7,9])|(15([0-3]|[5-9]))|(166)|(17[0,1,3,5,6,7,8])|(18[0-9])|(19[8|9]))\\d{8}$", message = "手机号格式错误")
    private String phoneNum;
    @Email(message = "邮箱格式错误")
    private String email;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getPhoneNum() {
        return phoneNum;
    }

    public void setPhoneNum(String phoneNum) {
        this.phoneNum = phoneNum;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

(3) 检验分组为IGroupA的情况

```
@RestController
public class CheckController {

    @PostMapping("/stu")
    public String addStu(@Validated({IGroupA.class})  Student studentBean){
        return "add student success";
    }
}
```

很明显，这里对IGroupA是不起作用的，修改为@Validated({IGroupB.class})或@Validated({IGroupA.class, IGroupB.class})。

说明：
对一个参数需要多种验证方式时，也可通过分配不同的组达到目的。

**3. 组序列**

默认情况下 不同级别的约束验证是无序的，但是在一些情况下，顺序验证却是很重要。

一个组可以定义为其他组的序列，使用它进行验证的时候必须符合该序列规定的顺序。在使用组序列验证的时候，如果序列前边的组验证失败，则后面的组将不再给予验证。

(1) 定义组序列

```
@GroupSequence({Default.class, IGroupA.class, IGroupB.class})
public interface IGroup {
}
```

(2) 需要校验的Bean，分别定义IGroupA对age进行校验，IGroupB对email进行校验：

```
public class Student implements Serializable {
    @NotBlank(message = "用户名不能为空")
    private String name;
    //只在分组为IGroupB的情况下进行验证
    @Min(value = 18, message = "年龄不能小于18岁", groups = {IGroupA.class})
    private Integer age;
    @Pattern(regexp = "^((13[0-9])|(14[5,7,9])|(15([0-3]|[5-9]))|(166)|(17[0,1,3,5,6,7,8])|(18[0-9])|(19[8|9]))\\d{8}$", message = "手机号格式错误")
    private String phoneNum;
    @Email(message = "邮箱格式错误", groups = IGroupB.class)
    private String email;
}
```

(3) 测试

```
@RestController
public class CheckController {

    @PostMapping("/stu")
    public String addStu(@Validated({IGroup.class})  Student studentBean){
        return "add student success";
    }
}
```

测试发现，如果age出错，那么对组序列在IGroupA后的IGroupB不进行校验。

**4. 嵌套校验**

一个待验证的pojo类，其中还包含了待验证的对象，需要在待验证对象上注解@Valid，才能验证待验证对象中的成员属性，这里不能使用@Validated。

(1) 需要约束的bean

```
public class TeacherBean {
    @NotEmpty(message = "老师姓名不能为空")
    private String teacherName;
    @Min(value = 1, message = "学科类型从1开始计算")
    private int type;
}
public class Student implements Serializable {
    @NotBlank(message = "用户名不能为空")
    private String name;
    //只在分组为IGroupB的情况下进行验证
    @Min(value = 18, message = "年龄不能小于18岁")
    private Integer age;
    @Pattern(regexp = "^((13[0-9])|(14[5,7,9])|(15([0-3]|[5-9]))|(166)|(17[0,1,3,5,6,7,8])|(18[0-9])|(19[8|9]))\\d{8}$", message = "手机号格式错误")
    private String phoneNum;
    @Email(message = "邮箱格式错误")
    private String email;

    @NotNull(message = "任课老师不能为空")
    @Size(min = 1, message = "至少有一个老师")
    private List<TeacherBean> teacherBeans;
}
```

上面这样写，对teacherBeans只校验了NotNull, 和 Size，并没有对teacher信息里面的字段进行校验，如果需要多里面的属性也进行校验，可以在teacherBeans中加上 @Valid

```
@Valid
@NotNull(message = "任课老师不能为空")
@Size(min = 1, message = "至少有一个老师")
private List<TeacherBean> teacherBeans;
```

### 四、验证结果接收

可以Controller的方法入参上添加BindingResult参数，用于接收校验后的验证结果，如果多个校验对象，那么每个@Validated后面跟着的BindingResult就是这个@Validated的验证结果，顺序不能乱。

```
@Validated People p, BindingResult result, @Validated Person p2
```

可能用到的操作：

```java
// result是BindingResult 实例
if(result.hasErrors()){
    List<ObjectError> allErrors = result.getAllErrors();
    for(ObjectError error : allErrors){
        FieldError fieldError = (FieldError)error;
        // 属性
        String field = fieldError.getField();
        // 错误信息
        String message = fieldError.getDefaultMessage();
        System.out.println(field + ":" + message);
    }
}
```

### 五、统一异常处理

```java
	@ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    public String MethodArgumentNotValidExceptionHandler(MethodArgumentNotValidException e) {
        ObjectError objectError = e.getBindingResult().getAllErrors().get(0);
        return objectError.getDefaultMessage();
    }
```

## 自己测试如下

- 可以添加多个校验方式：比如@URL校验无法校验空，直接再加一个@NotBlank

```java
@NotBlank(message = "URL不能为空")
@URL(regexp = "(https?|ftp|file)://[-A-Za-z0-9+&@#/%?=~_|!:,.;]+[-A-Za-z0-9+&@#/%=~_|]", message = "URL不合法")
private String url;         //URL
```

- Integer

```java
@Min(value = 0, message = "浏览量不能为负数")
@NotNull
private Integer visits;     //浏览量
```

> 原因是源码都放行了null

```java
public class PositiveValidatorForInteger implements ConstraintValidator<Positive, Integer> {
   @Override
   public boolean isValid(Integer value, ConstraintValidatorContext context) {
      // null values are valid
      if ( value == null ) {
         return true;
      }
      return NumberSignHelper.signum( value ) > 0;
   }
}
```

## 对简单参数进行校验


> ```java
> @RequestParam 请求参数
> @GetMapping("/test1/{id1}")
>     public String test1(@PathVariable(name = "id1") Integer id, @RequestParam String name) {
>         System.out.println("路径参数" + id + ": name: " + name);
> 
>         return "Gabriel";
>     }
>  
> PathVariable
> @GetMapping("/url/{age}")
>     public String URL1(@PathVariable @Range(min = 0, max = 150, message = "超过最大年龄") int age){
>         return "success";
>     }
> ```



## 自定义校验

业务需求总是比框架提供的这些简单校验要复杂的多，我们可以自定义校验来满足我们的需求。自定义 spring validation 非常简单，主要分为两步。

1 自定义校验注解
我们尝试添加一个“字符串不能包含空格”的限制。

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {CannotHaveBlankValidator.class})<1>
public @interface CannotHaveBlank {

    // 默认错误消息
    String message() default "不能包含空格";

    // 分组
    Class<?>[] groups() default {};

    // 负载
    Class<? extends Payload>[] payload() default {};

    // 指定多个时使用
    @Target({FIELD, METHOD, PARAMETER, ANNOTATION_TYPE})
    @Retention(RUNTIME)
    @Documented
    @interface List {
        CannotHaveBlank[] value();
    }

}
```

我们不需要关注太多东西，使用 spring validation 的原则便是便捷我们的开发，例如 payload，List ，groups，都可以忽略。

<1> 自定义注解中指定了这个注解真正的验证者类。

2 编写真正的校验者类

```java
public class CannotHaveBlankValidator implements <1> ConstraintValidator<CannotHaveBlank, String> {

	@Override
    public void initialize(CannotHaveBlank constraintAnnotation) {
    }
    
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context <2>) {
        //null 时不进行校验
        if (value != null && value.contains(" ")) {
	        <3>
            // 获取默认提示信息
            String defaultConstraintMessageTemplate = context.getDefaultConstraintMessageTemplate();
            System.out.println("default message :" + defaultConstraintMessageTemplate);
            // 禁用默认提示信息
            context.disableDefaultConstraintViolation();
            // 设置提示语
            context.buildConstraintViolationWithTemplate("can not contains blank").addConstraintViolation();
            return false;
        }
        return true;
    }
}
```

<1> 所有的验证者都需要实现 ConstraintValidator 接口，它的接口也很形象，包含一个初始化事件方法，和一个判断是否合法的方法。

```java
public interface ConstraintValidator<A extends Annotation, T> {
	void initialize(A constraintAnnotation);
	boolean isValid(T value, ConstraintValidatorContext context);
}
```

<2> ConstraintValidatorContext 这个上下文包含了认证中所有的信息，我们可以利用这个上下文实现获取默认错误提示信息，禁用错误提示信息，改写错误提示信息等操作。

<3> 一些典型校验操作，或许可以对你产生启示作用。

值得注意的一点是，自定义注解可以用在 `METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER` 之上，ConstraintValidator 的第二个泛型参数 T，是需要被校验的类型。

## 手动校验

可能在某些场景下需要我们手动校验，即使用校验器对需要被校验的实体发起 validate，同步获得校验结果。理论上我们既可以使用 Hibernate Validation 提供 Validator，也可以使用 Spring 对其的封装。在 spring 构建的项目中，提倡使用经过 spring 封装过后的方法，这里两种方法都介绍下：

**Hibernate Validation**：

```
Foo foo = new Foo();
foo.setAge(22);
foo.setEmail("000");
ValidatorFactory vf = Validation.buildDefaultValidatorFactory();
Validator validator = vf.getValidator();
Set<ConstraintViolation<Foo>> set = validator.validate(foo);
for (ConstraintViolation<Foo> constraintViolation : set) {
    System.out.println(constraintViolation.getMessage());
}
```

由于依赖了 Hibernate Validation 框架，我们需要调用 Hibernate 相关的工厂方法来获取 validator 实例，从而校验。

在 spring framework 文档的 Validation 相关章节，可以看到如下的描述：

> Spring provides full support for the Bean Validation API. This includes convenient support for bootstrapping a JSR-303/JSR-349 Bean Validation provider as a Spring bean. This allows for a javax.validation.ValidatorFactory or javax.validation.Validator to be injected wherever validation is needed in your application. Use the LocalValidatorFactoryBean to configure a default Validator as a Spring bean:

> bean id=”validator” class=”org.springframework.validation.beanvalidation.LocalValidatorFactoryBean”

> The basic configuration above will trigger Bean Validation to initialize using its default bootstrap mechanism. A JSR-303/JSR-349 provider, such as Hibernate Validator, is expected to be present in the classpath and will be detected automatically.

上面这段话主要描述了 spring 对 validation 全面支持 JSR-303、JSR-349 的标准，并且封装了 LocalValidatorFactoryBean 作为 validator 的实现。值得一提的是，这个类的责任其实是非常重大的，他兼容了 spring 的 validation 体系和 hibernate 的 validation 体系，也可以被开发者直接调用，代替上述的从工厂方法中获取的 hibernate validator。由于我们使用了 springboot，会触发 web 模块的自动配置，LocalValidatorFactoryBean 已经成为了 Validator 的默认实现，使用时只需要自动注入即可。

```java
@Autowired
Validator globalValidator; <1>

@RequestMapping("/validate")
public String validate() {
    Foo foo = new Foo();
    foo.setAge(22);
    foo.setEmail("000");

    Set<ConstraintViolation<Foo>> set = globalValidator.validate(foo);<2>
    for (ConstraintViolation<Foo> constraintViolation : set) {
        System.out.println(constraintViolation.getMessage());
    }

    return "success";
}
```

<1> 真正使用过 Validator 接口的读者会发现有两个接口，一个是位于 javax.validation 包下，另一个位于 org.springframework.validation 包下，注意我们这里使用的是前者 javax.validation，后者是 spring 自己内置的校验接口，LocalValidatorFactoryBean 同时实现了这两个接口。

<2> 此处校验接口最终的实现类便是 LocalValidatorFactoryBean。

## 基于方法校验

```java
@RestController
@Validated <1>
public class BarController {

    @RequestMapping("/bar")
    public @NotBlank <2> String bar(@Min(18) Integer age <3>) {
        System.out.println("age :" + age);
        return "";
    }

    @ExceptionHandler(ConstraintViolationException.class)
    public Map handleConstraintViolationException(ConstraintViolationException cve){
        Set<ConstraintViolation<?>> cves = cve.getConstraintViolations();<4>
        for (ConstraintViolation<?> constraintViolation : cves) {
            System.out.println(constraintViolation.getMessage());
        }
        Map map = new HashMap();
        map.put("errorCode",500);
        return map;
    }

}
```

<1> 为类添加 @Validated 注解

<2> <3> 校验方法的返回值和入参

<4> 添加一个异常处理器，可以获得没有通过校验的属性相关信息

基于方法的校验，个人不推荐使用，感觉和项目结合的不是很好。

## 使用校验框架的一些想法

理论上 spring validation 可以实现很多复杂的校验，你甚至可以使你的 Validator 获取 ApplicationContext，获取 spring 容器中所有的资源，进行诸如数据库校验，注入其他校验工具，完成组合校验（如前后密码一致）等等操作，但是寻求一个易用性和封装复杂性之间的平衡点是我们作为工具使用者应该考虑的，我推崇的方式，是仅仅使用自带的注解和自定义注解，完成一些简单的，可复用的校验。而对于复杂的校验，则包含在业务代码之中，毕竟如用户名是否存在这样的校验，仅仅依靠数据库查询还不够，为了避免并发问题，还是得加上唯一索引之类的额外工作，不是吗？