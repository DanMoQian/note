![这里写图片描述](https://user-gold-cdn.xitu.io/2018/1/5/160c3d5d9b8b17cd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

https://github.com/doocs/jvm/blob/main/docs/01-jvm-memory-structure.md



![img](https://p0.meituan.net/travelcube/132ba6ba720f2bfc6c69b1ce490f7c87693987.jpg)



# Java对象在内存中实例化的过程

在讲 Java 对象在内存中的实例化过程前，先来说下在类的实例化过程中，内存会使用到的三个区域：栈区、堆区、方法区。

- 堆区：

  - 存储的全部都是对象，每个对象包含了一个与之对应的 class 类的信息。
  - jvm 只有一个堆区(steap)，它会被所有线程共享，堆中不存放基本数据类型和对象引用，它只存放对象本身。

- 栈区:

  - 每个线程都包含一个栈区，栈中只保存基本数据类型的值和对象以及基础数据的引用。
  - 每个栈中的数据（基本数据类型和对象的引用）都是私有的，其它栈是无法进行访问的。
  - 栈分为三个部分：基本类型变量区、执行环境上下文、操作指令区(存放操作指令)。

- 方法区：

  - 又被称为静态区，它跟堆一样，被所有的线程共享，方法区包含所有的 class 信息 和 static修饰的变量。
  - 方法区中包含的都是整个程序中永远唯一的元素，如：class、static变量。

  

  ![image-20201228185113514](C:\Users\DMQi\AppData\Roaming\Typora\typora-user-images\image-20201228185113514.png)

  ​														JDK1.8之后的堆空间



![image-20201228195027654](C:\Users\DMQi\AppData\Roaming\Typora\typora-user-images\image-20201228195027654.png)



## 简谈 JDK8 前后 JVM 内存变化

JDK8 之前对内存划分为:新生代（YOUNG）—老年代（Tenured）—永久代(PermGen)
**新生代：**
新生代又分为伊甸区（Eden） 存活区(Survivor)，其中存活区又分为两个大小空间一样的s0、s1，而且s0 和 s1 可以互相转化，存活区保存的一定是在伊甸区保存了很久的，并且经过好几次小的GC还存活下来的对象，存活区一定会有两块大小相等的空间。目的是一块存活区未来的晋升，另一块存活区是为了对象的回收。需要注意的是：**这两块存活区一定有一块是空的**。

**新生代中的 GC:**
新生代大小（PSYoungGen total 9216K）=eden大小（eden space 8192K）+1个survivor大小（from space 1024K）

HotSpot JVM把年轻代分为了三部分：1个Eden区和2个Survivor区（分别叫from和to）。默认比例为8（Eden）：1（一个survivor）,一般情况下，新创建的对象都会被分配到Eden区(一些大对象特殊处理),这些对象经过第一次Minor GC后，如果仍然存活，将会被移到Survivor区。对象在Survivor区中每熬过一次Minor GC，年龄就会增加1岁，当它的年龄增加到一定程度时，就会被移动到老年代中。
　　因为年轻代中的对象基本都是朝生夕死的(80%以上)，所以在年轻代的垃圾回收算法使用的是复制算法，复制算法的基本思想就是将内存分为两块，每次只用其中一块，当这一块内存用完，就将还活着的对象复制到另外一块上面。复制算法不会产生内存碎片。
在GC开始的时候，对象只会存在于*Eden区和名为“From”的Survivor区*，Survivor区“To”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到老年代中。

**为什么要设置两个Survivor区？**
设置两个Survivor区最大的好处就是解决了碎片化；
假设现在只有一个survivor区，我们来模拟一下流程：
刚刚新建的对象在Eden中，一旦Eden满了，触发一次Minor GC，Eden中的存活对象就会被移动到Survivor区。这样继续循环下去，下一次Eden满了的时候，问题来了，此时进行Minor GC，Eden和Survivor各有一些存活对象，如果此时把Eden区的存活对象硬放到Survivor区，很明显这两部分对象所占有的内存是不连续的，也就导致了内存碎片化。

碎片化带来的风险是极大的，严重影响Java程序的性能。堆空间被散布的对象占据不连续的内存，最直接的结果就是，堆中没有足够大的连续内存空间，接下去如果程序需要给一个内存需求很大的对象分配内存。。。画面太美不敢看。。。这就好比我们上学时背包里所有东西紧挨着放，最后就可能省出一块完整的空间放饭盒。如果每件东西之间隔一点空隙乱放，很可能最后就要手提一路了。

### 1.1、Java 中的数据类型

Java 中的数据类型有两种：

1、**基本类型**（primitive types）: 共有8种，即：int、short、long、byte、char、float、double、boolean（注意并没有 String 的基本类型），这8中类型的定义是通过诸如：int a = 5；long b = 22L;的形式来定义的，称为自动变量。注意：自动变量存的是字面值，不是类的实例，即不是类的引用，这里并没有类的存在；

如：int a = 5; 这里的 a 是一个指向 int 类型的引用，指向 5 这个字面值，这些字面值的数据由于大小可知，生存期可知（ 这些字面值固定定义在某个程序块里面，程序块退出后，字段值就消失了 ），出于追求速度的原因，这些字面值就存在于栈区中；

另外，栈有一个很重要的特殊性，就是存在栈中的数据可以共享。假设我们同时定义
　 　int a = 3;
　　 int b = 3；
　　 编译器先处理int a = 3；首先它会在栈中创建一个变量为a的引用，然后查找有没有字面值为3的地址，没找到，就开辟一个存放3这个字面值的地址，然后将a指向3的地址。接着处理int b = 3；在创建完b的引用变量后，由于在栈中已经有3这个字面值，便将b直接指向3的地址。这样，就出现了a与b同时均指向3的情况。

特别注意的是：这种字面值的引用与类对象的引用不同。假定两个类对象的引用同时指向一个对象，如果一个对象引用变量修改了这个对象的内部状态，那么另一个对象引用变量也即刻反映出这个变化。相反，通过字面值的引用来修改其值，不会导致另一个指向此字面值的引用的值也跟着改变的情况。如上例，我们定义完a与 b的值后，再令a=4；那么，b不会等于4，还是等于3。在编译器内部，遇到a=4；时，它就会重新搜索栈中是否有4的字面值，如果没有，重新开辟地址存放4的值；如果已经有了，则直接将a指向这个地址。因此a值的改变不会影响到b的值。

2、 **包装类数据**：如：String、Integer、Double等将相应的基本数据类型包装起来的类，这些数据全部存放在 堆 中， Java 用 `new()`语句来显示地告诉编译器，在运行时才根据需要动态创建，因此比较灵活，但缺点是要占用更多的时间。

### 1.2、类实例化时内存中发生的变化

首先我们先对下面的代码进行分析：

```java
public class People{
    String name; // 定义一个成员变量 name
    int age; // 成员变量 age
    Double height; // 成员变量 height
    void sing(){
        System.out.println("人的姓名："+name);
        System.out.println("人的年龄："+age);
        System.out.println("人的身高："+height);
    }
    
    public static void main(String[] args) {
        String name; // 定义一个局部变量 name
    	int age; // 局部变量 age
    	Double height; // 局部变量 height
        
        People people = new People() ; //实例化对象people
        people.name = "张三" ;       //赋值
        people.age = 18;             //赋值
        people.stuid = 180.0 ;   //赋值
        people.sing();              //调用方法sing
    }
}

```

**代码解析**：

这段代码首先定义三个成员变量：String name、int age、Double height 这三个变量都是只声明了没有初始化，然后定义了一个成员方法 sing();

在 main()方法里同样定义了三个一样的变量，只不过这些是局部变量；

在main() 函数里实例化对象 people , 内存中在堆区内会给实例化对象 people 分配一片地址，紧接着我们对实例化对象 people 进行了赋值。people 调用成员方法 sing() 。mian()函数打印输入人的姓名，人的年龄和人的身高，系统执行完毕。

**下面通过图解法展示实例化对象的过程中内存的变化：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200727150524513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RvbnlfX0phYQ==,size_16,color_FFFFFF,t_70)

在程序的执行过程中，首先类中的成员变量和方法体会进入到方法区，如图:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200727150545479.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RvbnlfX0phYQ==,size_16,color_FFFFFF,t_70)

程序执行到 main() 方法时，main()函数方法体会进入栈区，这一过程叫做进栈(压栈)，定义了一个用于指向 Person 实例的变量 person。如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200727150608130.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RvbnlfX0phYQ==,size_16,color_FFFFFF,t_70)

程序执行到 Person person = new Person(); 就会在堆内存开辟一块内存区间，用于存放 Person 实例对象，然后将成员变量和成员方法放在 new 实例中都是取成员变量&成员方法的地址值 如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200727150630116.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RvbnlfX0phYQ==,size_16,color_FFFFFF,t_70)

接下来对 person 对象进行赋值， person.name = “小二” ; perison.age = 13; person.height= 180.0;

先在栈区找到 person，然后根据地址值找到 new Person() 进行赋值操作。

如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020072715065299.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RvbnlfX0phYQ==,size_16,color_FFFFFF,t_70)

当程序走到 sing() 方法时，先到栈区找到 person这个引用变量，然后根据该地址值在堆内存中找到 new Person() 进行方法调用。

在方法体void speak()被调用完成后，就会立刻马上从栈内弹出（出站 )

最后，在main()函数完成后，main()函数也会出栈 如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200727150715386.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RvbnlfX0phYQ==,size_16,color_FFFFFF,t_70)
以上就是Java对象在内存中实例化的全过程。