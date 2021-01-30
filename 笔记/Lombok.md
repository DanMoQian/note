# Lombok

#### @Builder 构造器模式

通过 @Builder 构造对象

```
@Builder
public class PersonDTO {
    private String name;
    private Integer age;
}

  @PostMapping("/test1/{id1}")
    public String test1(
            @PathVariable(name = "id1") Integer id,
            @RequestParam String name,
            @RequestBody PersonDTO personDTO) {

        PersonDTO gabriel = PersonDTO.builder().name("Gabriel").age(18).build();
        System.out.println(gabriel);

        return "Gabriel";
    }  
```

注意：

\1. 如果使用了 @Builder 注解，就不能使用无参构造函数了去构建对象，也不能使用 Setter 方法赋值，只能使用 Builder 方式去构建对象

原因：@Builder 会为Bean生成一个 private 的无参构造函数

\2. 如果使用了 @Builder 注解，还想使用无参构造函数，则必须显示声明一个无参构造函数



```
@Builder
@Setter
@NoArgsConstructor
public class PersonDTO {
    private String name;
    private Integer age;
}
```

\3. 如果使用 Builder 模式构建了对象，并且直接返回前端是会报错的，如下示例：



```java
    @PostMapping("/test1/{id1}")
    public PersonDTO test1(
            @PathVariable(name = "id1") Integer id,
            @RequestParam String name,
            @RequestBody PersonDTO personDTO) {

        PersonDTO gabriel = PersonDTO.builder().name("gabriel").age(18).build();

        return gabriel;
    }
```

调用后报错：

```
常原因：org.springframework.http.converter.HttpMessageNotWritableException: No converter found for return value of type: class com.gx.missyou.dto.PersonDTO
2020-08-04 20:29:33.037  WARN 10576 --- [nio-8080-exec-3] .m.m.a.ExceptionHandlerExceptionResolver : Resolved [org.springframework.http.converter.HttpMessageNotWritableException: No converter found for return value of type: class com.gx.missyou.dto.PersonDTO]
```

`原因：因为使用 @Builder 注解后，Bean是没有 Getter 方法的，但是序列化一定是需要有 Getter 方法才可以，因此要解决该问题，需要添加 Getter 方法`

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@Builder
@Getter
public class PersonDTO {
    private String name;
    private Integer age;
}
```