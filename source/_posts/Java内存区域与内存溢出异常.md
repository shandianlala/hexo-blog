---
title: Java内存区域与内存溢出异常
tags:
  - java虚拟机
abbrlink: 21b75d7a-memory-overflow-exception
date: 2018-09-22 22:34:14
---

### 第二章 Java内存区域与内存溢出异常

#### 2.1 概述
- 因为有java虚拟机的存在，java程序员不需要为每一个new出来的对象去编写`delete/free`代码，因为虚拟机自动内存管理机制不容易出现内存溢出的问题。


<!-- more -->

#### 2.2 运行时数据区
- *2.2.1* 程序计数器
    - 每条线程都要一个独立的程序计数器，为了线程切换后能够切换到正确的执行位置。
    - 此内存区域是唯一一个在java虚拟机规范中没有规定任何**OutOfMerroyError**情况的区域
- *2.2.2* java虚拟机栈
    - Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的
生命周期与线程相同。
    - Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧（Stack  Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。
    - 局部变量表存放了编译期可知的各种基本数据类型、对象引用、returnAddress类型。
- *2.2.3* 本地方法栈
    - 虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务 。
- *2.2.4* java堆
    - Java堆(Java Heap)是Java虚拟机所管理的内存中最大的一块。
    - 此内存区域的唯一目的就是存放对象实例，**几乎**所有的对象实例都在这里分配内存。(不是绝对哦)。
    - Java堆中还可以细分为：新生代和老年代；再细致一点的有Eden空间、From Survivor空间、To Survivor空间等。
    - 无论如何划分，都与存放内容无关，无论哪个区域，存储的都仍然是对象实例，进一步划分的目的是为了更好地回收内存，或者更快地分配内存。
    - Java堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可。
- *2.2.5* 方法区
   - 方法区（MethodArea）与**Java堆**一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据；别名叫做Non-Heap(非堆)。
   - 不需要连续的内存、可以选择固定大小或者可拓展、可以选择不实现垃圾收集。
   - 该区域的内存回收主要是针对常量池的回收和对类型的卸载(条件相当严苛)，回收“成绩”难以令人满意。
- *2.2.6* 运行时常量池
    - 是方法区的一部分。
    - Class文件除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池，用于存储编译期间生成的各种字面量和符号引用，在类加载后进入方法区的`运行时常量池`。
    - 运行时常量池相对Class文件常量池的另外一个重要的特征是具备动态性，运行期间也可能将新的常量放入池中。
- *2.2.7* 直接内存

#### 2.3　HotSpot虚拟机对象探秘
- *2.3.1* 对象的创建
    - 为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来。两种分配方式：**指针碰撞**(Bump the Pointer)、**空闲列表**(Free List)；在使用Serial、ParNew等带Compact过程的收集器时，系统采用的分配算法是指针碰撞，而使用CMS这种基于Mark-Sweep算法的收集器时，通常采用空闲列表。
    - 采用CAS配上失败重试的方式保证更新操作的原子性
    - 在Java堆中预先分配一小块内存，称为本地线程分配缓冲(Thread Local Allocation Buffer,TLAB)
    - JIT（just in time）即时编译器；
- *2.3.2* 对象的内存布局
    - 在HotSpot虚拟机中,对象在内存中存储的布局可以分为3块区域：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）。
    - 对象头的另一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是属于哪个类的实例。如果对象是一个Java数组，那在对象头中还必须有一块记录数组长度的数据。
    - 实例数据部分是对象真正存储的有效信息，也是在程序代码中定义的各种类型的字段类容。从父类继承和子类中定义的都需要记录下来。
    - 对其补充并不是必然存在的，起着占位符的作用。
- *2.3.3* 对象的访问定位
    - Java程序需要通过栈上的reference数据来操作堆上的具体对象，访问堆中的对象取决于虚拟机的实现而定的，目前主流的访问方式有：使用句柄、直接指针两种。
    - 使用句柄：java堆中会划出一块内存来作为句柄池，reference中存储的就是对象的句柄地址。
    - 直接指针：最大的好处的速度更快，节省了一次指针定位的开销。