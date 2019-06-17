---
title: JVM内存模型学习
date: 2019-05-22 11:04:56
tags:  [java,底层] 
---

### JVM内存结构

Java代码是要运行在虚拟机上的，而虚拟机在执行Java程序的过程中会把所管理的内存划分为若干个不同的数据区域，这些区域都有各自的用途。其中有些区域随着虚拟机进程的启动而存在，而有些区域则依赖用户线程的启动和结束而建立和销毁。

<!--more-->

![JVM运行时内存结构](E:\tyg\study\blog\source\_posts\img\JVM运行时内存结构.png)

除了以上介绍的JVM运行时内存外，还有一块内存区域可供使用，那就是直接内存。Java虚拟机规范并没有定义这块内存区域，所以他并不由JVM管理，是利用本地方法库直接在堆外申请的内存区域。

JVM内存结构，由Java虚拟机规范定义。描述的是Java程序执行过程中，由JVM管理的不同数据区域。各个区域有其特定的功能。

### Java内存模型

Java堆和方法区的区域是多个线程共享的数据区域。也就是说，多个线程可能可以操作保存在堆或者方法区中的同一个数据。这也就是我们常说的“Java的线程间通过共享内存进行通信”。

Java内存模型  Java Memory Model（JMM）描述了一组规则或规范，这个规范定义了一个线程对共享变量的写入时对另一个线程是可见的。JMM定义了一些语法集，这些语法集映射到Java语言中就是volatile、synchronized等关键字。

《Java并发编程的艺术》。

### Java对象模型

HotSpot虚拟机中，设计了一个OOP-Klass Model。OOP（Ordinary Object Pointer）指的是普通对象指针，而Klass用来描述对象实例的具体类型。

![java中的对象存储模型](E:\tyg\study\blog\source\_posts\img\java中的对象存储模型.jpeg)

这就是一个简单的Java对象的OOP-Klass模型，即Java对象模型。



参考：[JVM内存结构 VS Java内存模型 VS Java对象模型](https://www.hollischuang.com/archives/2509)