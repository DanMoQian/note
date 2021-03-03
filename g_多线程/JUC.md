# JUC

## 1. 什么是JUC

源码+官方文档

![image-20201225121309985](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201225121309985.png)

java工具包

> java不能开启多线程

Thread.start();

```java
private native void start0();
```

调用的是本地方法

## 2. 线程和进程

> 进程是一个应用，可以有多个线程

> 并发 一个CPU核在一段时间执行多个线程
>
> 并行 多个CPU核分别执行各自的线程

> 线程的状态：
>
> 6种

Thread.State

```java
public enum State {
    /**
     * Thread state for a thread which has not yet started.
     */
    NEW,

    /**
     * Thread state for a runnable thread.  A thread in the runnable
     * state is executing in the Java virtual machine but it may
     * be waiting for other resources from the operating system
     * such as processor.
     */
    RUNNABLE,

    /**
     * Thread state for a thread blocked waiting for a monitor lock.
     * A thread in the blocked state is waiting for a monitor lock
     * to enter a synchronized block/method or
     * reenter a synchronized block/method after calling
     * {@link Object#wait() Object.wait}.
     */
    BLOCKED,

    /**
     * Thread state for a waiting thread.
     * A thread is in the waiting state due to calling one of the
     * following methods:
     * <ul>
     *   <li>{@link Object#wait() Object.wait} with no timeout</li>
     *   <li>{@link #join() Thread.join} with no timeout</li>
     *   <li>{@link LockSupport#park() LockSupport.park}</li>
     * </ul>
     *
     * <p>A thread in the waiting state is waiting for another thread to
     * perform a particular action.
     *
     * For example, a thread that has called <tt>Object.wait()</tt>
     * on an object is waiting for another thread to call
     * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
     * that object. A thread that has called <tt>Thread.join()</tt>
     * is waiting for a specified thread to terminate.
     */
    WAITING,

    /**
     * Thread state for a waiting thread with a specified waiting time.
     * A thread is in the timed waiting state due to calling one of
     * the following methods with a specified positive waiting time:
     * <ul>
     *   <li>{@link #sleep Thread.sleep}</li>
     *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
     *   <li>{@link #join(long) Thread.join} with timeout</li>
     *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
     *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
     * </ul>
     */
    TIMED_WAITING,

    /**
     * Thread state for a terminated thread.
     * The thread has completed execution.
     */
    TERMINATED;
}
```

> wait/sleep 区别

1. 来自不同的类

wait => object

sleep => Thread

2. 关于锁的释放

wait 会释放锁 sleep 不会

3. 使用范围不同

wait 在同步代码块

sleep 任意地方

## 3. LOCK锁

> 传统synchronized

```java
public class ASaleTicket {

    public static void main(String[] args) {

        Ticket ticket = new Ticket();
        for (int i = 0; i < 3; i++) {
            new Thread(()->{
                while (ticket.getTicket() > 0){
                    ticket.sale();
                }
            }
            ).start();
        }
    }
}
@Data
class Ticket{
    private int ticket = 10000;
    public synchronized void sale(){
        if (ticket > 0){
            System.out.println(Thread.currentThread().getName() + "--》" + --ticket);
        }
    }
}
```

> LOCK

![image-20201225125221441](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201225125221441.png)

![image-20201225125446711](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201225125446711.png)

公平锁：不能插队

非公平锁： 可以插队（默认）



- 使用

![image-20201225125643245](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201225125643245.png)

三部曲：

```java
private Lock lock = new ReentrantLock();
lock.lock();
lock.unlock();
```



> synchronized和LOCK区别

![image-20201225131200562](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201225131200562.png)

> 锁是什么

## 4. 生产者和消费者

> synchronized实现

```java
package com.mycode;

public class CConsumerAdProvide {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.add();
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.dec();
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.dec();
            }
        }).start();
    }
}

//等待 业务 通知
class Data {
    private int num = 0;

    public synchronized void add() {
        while (num != 0) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num++;
        System.out.println(Thread.currentThread().getName() + "-->" + num);
        this.notifyAll();

    }

    public synchronized void dec() {

        while (num == 0) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num--;
        System.out.println(Thread.currentThread().getName() + "-->" + num);
        this.notifyAll();
    }
}
```

![image-20201225135042048](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201225135042048.png)

> LOCK实现

```java
package com.mycode;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class DConsumerAdProvide {


    public static void main(String[] args) {
        DData data = new DData();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.add();
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.dec();
            }
        }).start();
    }

}

//
// 上锁 等待 业务 通知 放锁
class DData {
    private int num = 0;
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    public void add() {
        try {
            lock.lock();
            while (num != 0) {
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num++;
            System.out.println(Thread.currentThread().getName() + "-->" + num);
            condition.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public synchronized void dec() {
        try {
            lock.lock();
            while (num == 0) {
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num--;
            System.out.println(Thread.currentThread().getName() + "-->" + num);
            condition.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

> condition可以实现精准唤醒

```java
package com.mycode;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * A->B->C->A
 */
public class DConsumerAdProvide {


    public static void main(String[] args) {
        DData data = new DData();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printA();
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printB();
            }
        }).start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printC();
            }
        }).start();
    }

}

//
// 上锁 等待 执行 通知 放锁
class DData {
    private int num = 1;
    Lock lock = new ReentrantLock();
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();
    Condition condition3 = lock.newCondition();

    public void printA() {
        try {
            lock.lock();
            while (num != 1) {
                try {
                    condition1.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "-->" + "A");
            num = 2;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void printB() {
        try {
            lock.lock();
            while (num != 2) {
                try {
                    condition2.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "-->" + "B");
            num = 3;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void printC() {
        try {
            lock.lock();
            while (num != 3) {
                try {
                    condition3.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "-->" + "C");
            num = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

## 5. 8锁现象

1. synchronized

```java
package com.mycode.lock8;

import java.util.concurrent.TimeUnit;

public class Test1 {

    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(()->phone.sendMsg()).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->phone.call()).start();
        new Thread(()->phone.sayHello()).start();
    }

}

class Phone{
    public synchronized void sendMsg(){
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("sendMsg");
    }
    public synchronized void call(){
        System.out.println("call");
    }

    public void sayHello(){
        System.out.println("hello");
    }
}
```

> 结果：1s后打印sayhello 然后3s打印sendMsg马上打印call 

> 静态sychronized锁的是Class对象
>
> sychronized锁的是调用者 this
>
> 两个锁不是同一个，不要认为锁了对象就能锁调用者



## 6.集合不安全

List

```java
public class Demo01 {

    /**
     * ConcurrentModificationException 并发修改异常
     * 多线程解决方案
     * 1. List list = new Vector();
     * 2. List list = Collections.synchronizedList(new ArrayList<>());
     * 3. List list = new CopyOnWriteArrayList();
     * 写入时复制， 避免并发情况下导致数据覆盖的问题
     *
     * @param args
     */
    public static void main(String[] args) {
        List list = new CopyOnWriteArrayList();
        for (int i = 0; i < 10; i++) {
            new Thread(()-> {
                list.add(new StringBuilder(UUID.randomUUID().toString()).substring(0, 5));
                System.out.println(list.toString());
            }).start();
        }


    }

}
```

## 7. Callable

```java
public class Test02 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Task task = new Task();
        //使用futurTask来开启和获得返回值
        FutureTask futureTask = new FutureTask(task);
        new Thread(futureTask).start();
        Integer i = (Integer) futureTask.get();
        System.out.println(i);

    }

}

class Task implements Callable<Integer>{

    @Override
    public Integer call() throws Exception {
        System.out.println("call() running");
        return 1024;
    }
}
```

## 8. CountDownLatch&Semaphore&

减法计数器

```java
public class MyCountDownLatch {

    public static void main(String[] args) {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                countDownLatch.countDown();
            }).start();
            System.out.println(i + "go out");
        }

        try {
            countDownLatch.await();//等待减数器为0
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("close door");

    }
}
```

```java
public class SemaphoreTest {

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(7);
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "获取了信号");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName() + "失去了信号");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            }, String.valueOf(i)).start();
        }
    }

}
```

## 9.读写锁

```java
package com.mycode.readWriteLock;

import com.mycode.OpenThread;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class RWL {

    public static void main(String[] args) {
        Cash cash = new Cash();
        for (int i = 0; i < 10; i++) {
            final String temp = i + "";
            new Thread(()->{
                cash.push(temp, "aaa");
            }).start();
        }

        for (int i = 0; i < 10; i++) {
            final String temp = i + "";
            new Thread(()->{
                cash.get(temp);
            }).start();
        }

        new OpenThread(()->cash.push("aaa", "bbb"), 10);
    }

}

class Cash{
    private volatile Map<String, String> map = new HashMap();

     private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void push(String key, String value){
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "写入开始");
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "写入结束");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }

    public String get(String key){
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "读取开始");
            String s = map.get(key);
            System.out.println(Thread.currentThread().getName() + "读取结束");
            return s;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
        return null;
    }

}
```

## 10 阻塞队列 BlockingQueue

![image-20201225214755331](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201225214755331.png)同步队列SynchronousQueue

```java
package com.mycode.queue;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

public class SynchronousQueueTest {

    public static void main(String[] args) {
        final BlockingQueue queue = new SynchronousQueue();
        new Thread(()->{
            try {
                System.out.println("放");
                queue.put("a");
                System.out.println("放");
                queue.put("b");
                System.out.println("放");
                queue.put("c");
                System.out.println("放");
                queue.put("d");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(3);
                System.out.println(queue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(queue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(queue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(queue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

    }
}
```

## 11.线程池

> 3大方法

```java
package com.mycode.pool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPool {
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newSingleThreadExecutor();//只有一个
        ExecutorService threadPool1 = Executors.newFixedThreadPool(5);//只有5个
        ExecutorService threadPool2 = Executors.newCachedThreadPool();//会变化

        try {
            for (int i = 0; i < 10; i++) {
                threadPool2.execute(()->{
                    System.out.println(Thread.currentThread().getName() + "执行了");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool1.shutdown();
            threadPool.shutdown();
            threadPool2.shutdown();
        }

    }
}
```

> 7大参数

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

//本质调用ThreadPoolExecutor
public ThreadPoolExecutor(int corePoolSize,//核心线程池大小
                              int maximumPoolSize,//最大线程
                              long keepAliveTime,//超时释放
                              TimeUnit unit,//超时单位
                              BlockingQueue<Runnable> workQueue,//阻塞队列
                              ThreadFactory threadFactory,//线程工厂
                              RejectedExecutionHandler handler//拒绝策略) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

![image-20201225231804721](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201225231804721.png)

![image-20201225232400513](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201225232400513.png)

> corePoolSize保证随时都开启，BlockingQueue满就开启maximumPoolSize，maximumPoolSize也满就RejectedExecutionHandler拒绝策略。keepAliveTime超时等待是指最大线程池一旦在这段时间没有使用就释放资源



> 四种拒绝策略

![image-20201225233437753](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201225233437753.png)

> 手写线程池

```java
package com.mycode.pool;

import java.util.concurrent.*;

/**
 * 拒绝策略
 * new ThreadPoolExecutor.AbortPolicy()超过最大的处理就抛出异常
 * new ThreadPoolExecutor.CallerRunsPolicy()哪来的回哪去
 * new ThreadPoolExecutor.DiscardPolicy()抛弃
 * new ThreadPoolExecutor.DiscardOldestPolicy()尝试插队，当当前正在执行的线程快完了，就尝试竞争插队
 */
public class ThreadPool {
    public static void main(String[] args) {
        //ExecutorService threadPool = Executors.newSingleThreadExecutor();//只有一个
        //ExecutorService threadPool1 = Executors.newFixedThreadPool(5);//只有5个
        //ExecutorService threadPool2 = Executors.newCachedThreadPool();//会变化
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(5),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardOldestPolicy()
        );
        try {
            for (int i = 0; i < 100; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName() + "执行了");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }

    }
}
```

> 最大线程定义 （调优）

```java
//最大线程定义
//1. CPU密集型 当前Cpu有几个就定义几个  效率最高
Runtime.getRuntime().availableProcessors()
//2. IO密集型 大于当前项目的大型任务数 因为IO开启会浪费资源
```

## 12.函数式接口

![image-20201226000045776](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201226000045776.png)	



> fuction 接口
>
> function.apply   有参数和返回值

```java
package com.mycode.fuction;

import java.util.function.Function;

public class FunctionTest {

    public static void main(String[] args) {
        Function function = str -> str;
        String ss = (String) function.apply("ss");
        System.out.println(ss);
    }

}
```



> predicate
>
> predicate.test 有参数和布尔类型返回值

```
package com.mycode.fuction;

import java.util.function.Predicate;

public class PredicatTest {

    public static void main(String[] args) {
        Predicate<String> predicate = str-> str.isEmpty();
        System.out.println(predicate.test("abc"));
    }

}
```

> Consumer
>
> consumer.accept  有参数无返回值

```java
package com.mycode.fuction;

import java.util.function.Consumer;

public class ConsumerTest {
    public static void main(String[] args) {
        Consumer<String> consumer = (str)->{
            System.out.println(str);
        };
       consumer.accept("aaa");
    }
}
```

> Supplier 接口
>
> Supplier.get()无参数有返回值

```java
class SupplierTest{
    public static void main(String[] args) {
        Supplier<String> stringSupplier = ()->{return "aaa";};
        System.out.println(stringSupplier.get());
    }
}
```

## 13.Stream流

```java
package com.mycode.fuction;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

//只要id为偶数 年龄大于23 用户名转为大写 倒序排列的第一个用户
public class StreamTest {

    public static void main(String[] args) {
        User u1 = new User(1, "a", 21);
        User u2 = new User(2, "b", 22);
        User u3 = new User(3, "c", 23);
        User u4 = new User(4, "d", 24);
        User u5 = new User(6, "e", 26);
        List<User> list = Arrays.asList(u1, u2, u3, u4, u5);

        list.stream().filter(u->u.getId() % 2 == 0)
                .filter(u->u.getAge()>23)
                .map((uu1)->uu1.getName().toUpperCase())
                .sorted((uu1, uu2)->uu2.compareToIgnoreCase(uu1))
                .limit(1)
                .forEach(System.out::println);
    }

}
```

## 14.ForkJoin

采用双端队列的方式

多个线程

任务窃取：执行完的线程可以去帮助未执行完的线程完成部分任务

```java
package com.mycode;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;
import java.util.stream.LongStream;

public class ForkJoinTest {

    private static final long TOTAL = 10_0000_0000L;

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        long start = System.currentTimeMillis();
        forkJoinSum();
        long end = System.currentTimeMillis();
        System.out.println("共花费"+(end-start)+"ms");
    }

    public static void toSum(){
        long sum = 0;
        for (long i = 1; i <= TOTAL; i++) {
            sum += i;
        }
        System.out.println(sum);
    }

    public static void forkJoinSum() throws ExecutionException, InterruptedException {
        ForkJoinDemo forkJoinDemo = new ForkJoinDemo(1, 10_0000_0000);
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> submit = forkJoinPool.submit(forkJoinDemo);
        Long aLong = submit.get();
        System.out.println(aLong);
    }
    
    public static void streamSum(){
        System.out.println(LongStream.rangeClosed(0, TOTAL).parallel().sum());
    }

}

//分治算法 
class ForkJoinDemo extends RecursiveTask<Long> {
    private long start;
    private long end;
    private static final long TEMP = 10000L;
    
    public ForkJoinDemo(long start, long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        long count = 0;
        if (end - start < TEMP){
            for (long i = start; i <= end; i++) {
                count += i;
            }
            return count;
        }else {
            long mid = (start+end) / 2;
            ForkJoinDemo forkJoinDemo = new ForkJoinDemo(start, mid);
            forkJoinDemo.fork();
            ForkJoinDemo forkJoinDemo1 = new ForkJoinDemo(mid + 1, end);
            forkJoinDemo1.fork();
            return forkJoinDemo.join()+forkJoinDemo1.join();
        }
    }
}
```

## 15. 异步回调

```java
package com.mycode.future;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

public class FutureDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        /*CompletableFuture completableFuture = CompletableFuture.runAsync(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "runAsync");
        });

        TimeUnit.SECONDS.sleep(3);
        System.out.println("main run...");
        completableFuture.get();*/
        CompletableFuture<Integer> integerCompletableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "supplyAsync");
            int error = 1024/0;
            return 1024;
        });
        System.out.println(integerCompletableFuture.whenComplete((t, u) -> {
            System.out.println("t=>"+ t);
            System.out.println("u=>"+ u);
        }).exceptionally((e)->{
            System.out.println(e.getMessage());
            return 404;
        }).get());
    }

}
```

## 16. JMM

> volatile

volatile是java虚拟机提供``轻量级的同步机制``

1. 保证可见性
2. 不保证原子性
3. 禁止指令重排

> JMM  java内存模型

 ![image-20201226143922583](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20201226143922583.png)

## 17.Volatile



## 18. 彻底玩转单例模式



## 19.深入理解CAS



## 20. 原子引用



## 21.各种锁的理解



## 22. 死锁

