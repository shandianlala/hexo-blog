---
title: Thread类阅读笔记
tags:
  - JAVA
  - jdk源码
abbrlink: 9c02c64f
date: 2017-12-04 21:53:21
---
Thread类在工作的过程中接触的少，也一直没花时间去细细阅读，Thread类是一个很重要的类，下面从Thread类的源码开刀喽。
<!-- more -->
```
public
class Thread implements Runnable {
    /* Make sure registerNatives is the first thing <clinit> does. */
    private static native void registerNatives();
    static {
        registerNatives();
    }

    private char        name[];         //线程名
    private int         priority;       //线程的优先级
    private Thread      threadQ;
    private long        eetop;

    /* Whether or not to single_step this thread. */
    private boolean     single_step;

    /* Whether or not the thread is a daemon thread. 是否为守护线程*/
    private boolean     daemon = false;

    /* JVM state 是否*/
    private boolean     stillborn = false;

    /* What will be run. 要执行的任务*/
    private Runnable target;

    /* The group of this thread */
    private ThreadGroup group;

    /* The context ClassLoader for this thread */
    private ClassLoader contextClassLoader;

    /* The inherited AccessControlContext of this thread */
    private AccessControlContext inheritedAccessControlContext;

    、、、、
}
```
## 相关属性
Thread类实现了Runnable接口，在Thread类中，有一些比较关键的属性，比如name是表示Thread的名字，可以通过Thread类的构造器中的参数来指定线程名字，priority表示线程的优先级（最大值为10，最小值为1，默认值为5），daemon表示线程是否是守护线程，target表示要执行的任务。
## 方法介绍
### start()方法用synchronized关键字修饰
```
public synchronized void start() {}
```
- synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种：
    1. **代码块：**被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象；
    2. **方法：**被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象；
    3. **静态：**的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象；
    4. **类：**其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。
start()用来启动一个线程，当调用start方法后，系统才会开启一个新的线程来执行用户定义的子任务，在这个过程中，会为相应的线程分配需要的资源。
### run()方法
    ```
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
    ```
    run()方法是不需要用户来调用的，当通过start方法启动一个线程之后，当线程获得了CPU执行时间，便进入run方法体去执行具体的任务。注意，继承Thread类必须重写run方法，在run方法中定义具体要执行的任务。
### sleep()方法,有两个重载的版本。
    ```
  - public static native void sleep(long millis) throws InterruptedException;
  - public static void sleep(long millis, int nanos)
    throws InterruptedException {
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }

        sleep(millis);
    }
    ```
    sleep相当于让线程睡眠，交出CPU，让CPU去执行其他的任务。但是有一点要非常注意，sleep方法不会释放锁，也就是说如果当前线程持有对某个对象的锁，则即使调用sleep方法，其他线程也无法访问这个对象。下面举个例子简单说明。
    - 新建线程类SThread，继承Thread类。
    ```
    package club.sdll.blog.thread;
    public class SThread extends Thread {
        private static Object object = new Object();
        private static int count = 1; 
        @Override
        public void run() {
            synchronized(object) {
                System.out.println("start count  ===  " + count);
                System.out.println("我是线程  ===  " + Thread.currentThread().getName());
                System.out.println(Thread.currentThread().getName() + "睡眠开始 === " + count);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "睡眠结束 === " + count);
                count++;
                System.out.println("end   count  ===  " + count);
            }
        }
    }

    ```
    Test测试类。
    ```
    package club.sdll.blog.thread;
    public class Test {
        public static void main(String[] args) {
            SThread cat = new SThread();
            cat.setName("小猫");
            SThread dog = new SThread();
            dog.setName("小狗");
            cat.start();
            dog.start();
        }
    }

    ```
    - 有**syschronized同步锁**的情况测试结果：
        ![](http://ozux0lqfa.bkt.clouddn.com/%E6%9C%89%E5%90%8C%E6%AD%A5%E9%94%81%E7%9A%84%E7%BB%93%E6%9E%9C.png)
        从上面输出结果可以看出，当**小猫thread**进入睡眠状态之后，**小狗thread**并没有去执行具体的任务。只有当**小猫thread**执行完之后，此时**小猫thread**释放了对象锁`object`，**小狗thread**才开始执行。当线程睡眠时间满后，不一定会立即得到执行，因为此时可能CPU正在执行其他的任务。所以说调用sleep方法相当于让线程进入阻塞状态。
    - 没有同步锁**synchronized(object) {}**的结果：两个线程同时都在争抢时间片
        ![](http://ozux0lqfa.bkt.clouddn.com/%E6%B2%A1%E6%9C%89%E5%90%8C%E6%AD%A5%E9%94%81%E7%9A%84%E7%BB%93%E6%9E%9C.png)
### yield()方法
调用yield方法会让当前线程交出CPU权限，让CPU去执行其他的线程。它跟sleep方法类似，同样不会释放锁。但是yield不能控制具体的交出CPU的时间，另外，yield方法只能让拥有相同优先级的线程有获取CPU执行时间的机会。注意，调用yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取CPU执行时间，这一点是和sleep方法不一样的。
### join()方法
假如在main线程中，调用thread.join方法，则main方法会等待thread线程执行完毕或者等待一定的时间。如果调用的是无参join方法，则等待thread执行完毕，如果调用的是指定了时间参数的join方法，则等待一定的时间。
```
package club.sdll.blog.thread;
import java.io.IOException;
public class Test2 {
    public static void main(String[] args) throws IOException  {
        System.out.println("进入线程“"+Thread.currentThread().getName()+"”");
        Test2 test2 = new Test2();
        SThread thread = test2.new SThread();
        thread.start();
        try {
            System.out.println("线程“"+Thread.currentThread().getName()+"”等待");
            thread.join();
            System.out.println("线程“"+Thread.currentThread().getName()+"”继续执行");
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    } 
     
    class SThread extends Thread{
        @Override
        public void run() {
            System.out.println("进入线程“"+Thread.currentThread().getName()+"”");
            try {
                Thread.currentThread().sleep(5000);
                System.out.println("线程“"+Thread.currentThread().getName() + "”休眠5秒");
            } catch (InterruptedException e) {
                // TODO: handle exception
            }
            System.out.println("离开线程“"+Thread.currentThread().getName()+"”");
        }
    }
}

```
控制台打印结果：
![](http://ozux0lqfa.bkt.clouddn.com/join%E6%96%B9%E6%B3%95%E6%B5%8B%E8%AF%95.png)

### 未完待续

**参考文章：**
https://www.cnblogs.com/franson-2016/p/5498221.html
    