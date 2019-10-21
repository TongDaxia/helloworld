---
title: Netty学习笔记-2-reactor
date: 2019-10-21
tags: [java,网络编程] 
description: netty最核心的就是reactor线程，对应项目中使用广泛的NioEventLoop。
---



###### reactor线程

netty中最核心的东西莫过于两种类型的reactor线程，可以看作netty中两种类型的发动机，驱动着netty整个框架的运转

一种类型的reactor线程是boos线程组，专门用来接受新的连接，然后封装成channel对象扔给worker线程组；还有一种类型的reactor线程是worker线程组，专门用来处理连接的读写。

不管是boos线程还是worker线程，所做的事情均分为以下三个步骤

1. 轮询注册在selector上的IO事件
2. 处理IO事件
3. 执行异步task

对于boos线程来说，第一步轮询出来的基本都是 accept  事件，表示有新的连接，而worker线程轮询出来的基本都是read/write事件，表示网络的读写事件



### 服务端启动

服务端启动过程是在用户线程中开启，第一次 添加异步任务的时候启动boos线程 被启动，netty将处理新连接的过程封装成一个channel，对应的pipeline会按顺序处理新建立的连接 。

































