# java注解

***注解\***目前非常的流行，很多主流框架都支持注解，而且自己编写代码的时候也会尽量的去用注解，一时方便，而是代码更加简洁。

```
 注解的语法比较简单，除了@符号的使用之外，它基本与Java固有语法一致。Java SE5内置了三种标准注解：

 @Override，表示当前的方法定义将覆盖超类中的方法。

 @Deprecated，使用了注解为它的元素编译器将发出警告，因为注解@Deprecated是不赞成使用的代码，被弃用的代码。

 @SuppressWarnings，关闭不当编译器警告信息。

 上面这三个注解多少我们都会在写代码的时候遇到。Java还提供了4中注解，专门负责新注解的创建。
123456789
```

**@Target**

表示该注解可以用于什么地方，可能的ElementType参数有：

CONSTRUCTOR：构造器的声明

FIELD：域声明（包括enum实例）

LOCAL_VARIABLE：局部变量声明

METHOD：方法声明

PACKAGE：包声明

PARAMETER：参数声明

TYPE：类、接口（包括注解类型）或enum声明

**@Retention**

表示需要在什么级别保存该注解信息。可选的RetentionPolicy参数包括：

SOURCE：注解将被编译器丢弃

CLASS：注解在class文件中可用，但会被VM丢弃

RUNTIME：VM将在运行期间保留注解，因此可以通过反射机制读取注解的信息。

**@Document**

将注解包含在Javadoc中

**@Inherited**

允许子类继承父类中的注解

　　定义一个注解的方式：

```
@Target(ElementType.METHOD)
 @Retention(RetentionPolicy.RUNTIME)
public @interface Test {

 }12345
```

@Override注解

```
@Override  注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}12345
```

除了@符号，注解很像是一个接口。定义注解的时候需要用到元注解，上面用到了@Target和@RetentionPolicy，它们的含义在上面的表格中已近给出。

```
 在注解中一般会有一些元素以表示某些值。注解的元素看起来就像接口的方法，唯一的区别在于可以为其制定默认值。没有元素的注解称为标记注解，上面的@Test就是一个标记注解。 

 注解的可用的类型包括以下几种：所有基本类型、String、Class、enum、Annotation、以上类型的数组形式。元素不能有不确定的值，即要么有默认值，要么在使用注解的时候提供元素的值。而且元素不能使用null作为默认值。注解在只有一个元素且该元素的名称是value的情况下，在使用注解的时候可以省略“value=”，直接写需要的值即可。 
123
```

**自定义一个注解：**

```
package com.Annatation;

import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.ElementType;


@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
     public int id();
     public String description() default "no description";
}1234567891011121314
```

注解解析器以及Test

```
package com.Annatation;

public class PasswordUtils {
      @UseCase(id=47 ,description="passwords must contain at least one numeric")
    public boolean validatePassword(String password){
        return(password.matches("\\w*\\d\\w*"));
    }


      @UseCase(id =48)
      public String encryptPassword(String password){
          return new StringBuilder(password).reverse().toString();
      }

}




package com.Annation;

import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.junit.Test;

import com.Annatation.PasswordUtils;
import com.Annatation.UseCase;

public class Test1 {

    @Test
    public void TestUseCaseAnnotation(){
        PasswordUtils pu=new PasswordUtils();
        System.out.println(pu.encryptPassword("sssss"));
    }

    @Test
    public void Test2(){
        List<Integer> useCases =new ArrayList<Integer>();
        Collections.addAll(useCases,47,48,49,50);
        trackUseCases(useCases,PasswordUtils.class);
    }

/***********注解处理器***************/

    public static void trackUseCases(List<Integer> useCases,Class<?> c1){
          for(Method m:c1.getDeclaredMethods()){  
            //  getDeclaredMethods    including public, protected, default (package) access, and private methods, but excluding inherited methods. 
             UseCase uc=m.getAnnotation(UseCase.class);
             if(uc !=null){
                 System.out.println("Found Use Case:"+uc.id()+" "+uc.description());

                 useCases.remove(new Integer(uc.id()));
             }
          }
          for(int i:useCases){
              System.out.println("Warning :Missing use case-"+i);
          }
    }
}123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263
```

**理解：**
先通过反射获取工具类的所有方法再通过getAnnotation方法获取UseCase注解

**result：**

```
Found Use Case:48 no description
Found Use Case:47 passwords must contain at least one numeric
Warning :Missing use case-49
Warning :Missing use case-501234
```

**Java注解的基础知识点（见下面导图）**

![这里写图片描述](https://img-blog.csdn.net/20171010090341394?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzMzNjYyMjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)