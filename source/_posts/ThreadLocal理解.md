---
title: ThreadLocal理解
tags:
  - JAVA
  - jdk源码
abbrlink: 362db54f-ThreadLocal
date: 2019-06-23 23:04:27
---

## 参考引用

[手撕面试题ThreadLocal！！！](https://mp.weixin.qq.com/s?__biz=MzU2NjIzNDk5NQ==&mid=2247486666&idx=1&sn=ee9d72b115411940940f00171986e0db&chksm=fcaed6d6cbd95fc08cf5525e7974efc7753f9f018f2a79fd5386b9f0132b90645394e85c3568&mpshare=1&scene=1&srcid=#rd)

## ThreadLocal是什么？
面试官：讲讲你对ThreadLocal的一些理解。

那么我们该怎么回答呢？你也可以思考下，下面看看零度的思考；
- ThreadLocal用在什么地方？

- ThreadLocal一些细节！

- ThreadLocal的最佳实践！

- 思考！

  

<!-- more -->

  

## ThreadLocal用在什么地方？
讨论ThreadLocal用在什么地方前，我们先明确下，如果仅仅就一个线程，那么都不用谈ThreadLocal的，ThreadLocal是用在多线程的场景的！
ThreadLocal归纳下来就2类用途：
- 保存线程上下文信息，在任意需要的地方可以获取！
> 1、由于ThreadLocal的特性，同一线程在某地方进行设置，在随后的任意地方都可以获取到。从而可以用来保存线程上下文信息。
2、常用的比如每个请求怎么把一串后续关联起来，就可以用ThreadLocal进行set，在后续的任意需要记录日志的方法里面进行get获取到请求id，从而把整个请求串起来。
3、还有比如Spring的事务管理，用ThreadLocal存储Connection，从而各个DAO可以获取同一Connection，可以进行事务回滚，提交等操作。
备注：ThreadLocal的这种用处，很多时候是用在一些优秀的框架里面的，一般我们很少接触，反而下面的场景我们接触的更多一些！
- 线程安全的，避免某些情况需要考虑线程安全必须同步带来的性能损失！
> ThreadLocal为解决多线程程序的并发问题提供了一种新的思路。但是ThreadLocal也有局限性，我们来看看阿里规范：
![](http://ww1.sinaimg.cn/large/8515e8c2ly1g4bgqpmiv3j20u004wdg7.jpg)
每个线程往ThreadLocal中读写数据是线程隔离，互相之间不会影响的，所以ThreadLocal无法解决共享对象的更新问题！
由于不需要共享信息，自然就不存在竞争问题了，从而保证了某些情况下线程的安全，以及避免了某些情况需要考虑线程安全必须同步带来的性能损失！
这类场景阿里规范里面也提到了：
![](http://ww1.sinaimg.cn/large/8515e8c2ly1g4bgtmiyk7j20u008rt92.jpg)

## ThreadLocal一些细节
ThreaLocal使用示例代码：
```
public class ThreadLocalTest {
    private static ThreadLocal<Integer> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {

        new Thread(() -> {
            try {
                for (int i = 0; i < 100; i++) {
                    // 给该线程的设置值
                    threadLocal.set(i);
                    System.out.println(Thread.currentThread().getName() + "====" + threadLocal.get());
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } finally {
                threadLocal.remove();
            }
        }, "threadLocal1").start();


        new Thread(() -> {
            try {
                for (int i = 0; i < 100; i++) {
                    System.out.println(Thread.currentThread().getName() + "====" + threadLocal.get());
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } finally {
                threadLocal.remove();
            }
        }, "threadLocal2").start();
    }
}
```
代码运行结果：

![](http://ww1.sinaimg.cn/large/8515e8c2ly1g4bgvby07kj20dp0723yi.jpg)

从运行的结果我们可以看到`threadLocal1`进行`set()`值对`threadLocal2`并**没有任何影响**！

- `Thread`、`ThreadLocalMap`、`ThreadLocal`总览图：
![](http://ww1.sinaimg.cn/large/8515e8c2ly1g4bgym7f89j20tq0m00t2.jpg)

- `Thread`类有属性变量`threadLocals`（类型是`ThreadLocal.ThreadLocalMap`），也就是说每个线程有一个自己的`ThreadLocalMap` ，所以每个线程往这个`ThreadLocal`中读写隔离的，并且是互相不会影响的。
![](http://ww1.sinaimg.cn/large/8515e8c2ly1g4bgz6eds7j20nd040weh.jpg)

- 一个`ThreadLocal`只能存储一个`Object`对象，如果需要存储多个`Object`对象那么就需要多个`ThreadLocal`。
如图：
![](http://ww1.sinaimg.cn/large/8515e8c2ly1g4bh2dmh7cj20ts0m2dgg.jpg)
看到上面的几个图，大概思路应该都清晰了，我们`Entry`的`key`指向`ThreadLocal`用虚线表示弱引用 ，下面我们来看看`ThreadLocalMap`:
![](http://ww1.sinaimg.cn/large/8515e8c2ly1g4bh3liojaj20na0c0jrt.jpg)

- java对象的引用包括 ： **强引用**，**软引用**，**弱引用**，**虚引用**。因为这里涉及到 **弱引用**，简单说明下：
> 弱引用也是用来描述非必需对象的，当JVM进行垃圾回收时，无论内存是否充足，该对象仅仅被弱引用关联，那么就会被回收。当仅仅只有ThreadLocalMap中的Entry的key指向ThreadLocal的时候，ThreadLocal会进行回收的！ThreadLocal被垃圾回收后，在ThreadLocalMap里对应的Entry的键值会变成null，但是Entry是强引用，那么Entry里面存储的Object，并没有办法进行回收，所以ThreadLocalMap 做了一些额外的回收工作。
![](http://ww1.sinaimg.cn/large/8515e8c2ly1g4bh6khf5kj20u00dhdgl.jpg)
虽然做了但是也会存在内存泄漏风险（我没有遇到过，网上很多类似场景，所以会提到后面的ThreadLocal最佳实践！）

## ThreadLocal的最佳实践！
很多时候，我们都是用在线程池的场景，程序不停止，线程基本不会销毁！
由于线程的生命周期很长，如果我们往ThreadLocal里面set了很大很大的Object对象，虽然set、get等等方法在特定的条件会调用进行额外的清理，**但是ThreadLocal被垃圾回收后，在ThreadLocalMap里对应的Entry的键值会变成null，但是后续在也没有操作set、get等方法了**。

所以最佳实践，应该在我们不使用的时候，主动调用remove方法进行清理。
![](http://ww1.sinaimg.cn/large/8515e8c2ly1g4bha0clrkj20u004xmxi.jpg)
这里把ThreadLocal定义为static还有一个好处就是，由于ThreadLocal有强引用在，那么在ThreadLocalMap里对应的Entry的键会永远存在，那么执行remove的时候就可以正确进行定位到并且删除！

## 思考
如果面试的时候，可以把上面的内容都可以讲到，个人觉得就非常好了，回答的就挺完美了。但是如果你可以进行下面的回答，那么就更完美了。

对于ThreadLocal，我在看Netty源码的时候，还了解过FastThreadLocal，xxxxx一些列内容，那就是一个升级了。
![](http://ww1.sinaimg.cn/large/8515e8c2ly1g4bhbtior6j20u00ijwf4.jpg)
在我本地进行测试，FastThreadLocal的吞吐量是jdkThreadLocal的3倍左右。

