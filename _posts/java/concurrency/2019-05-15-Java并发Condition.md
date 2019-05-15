---
layout: post
title: "Java并发Condition"
date: 2019-05-15 17:54:03 +0800
description: ""
category: java/concurrency
tags: []
---

在线程间实现通信的往往会应用到Object的几个方法，比如`wait()，wait(long timeout)，wait(long timeout, int nanos)与notify()，notifyAll()`几个方法实现等待/通知机制。同样的， 在Java Lock体系下依然会有同样的方法实现等待/通知机制。

Object的wait和notify是与对象monitor机制（详见：Java并发synchronized）配合完成线程间的等待/通知机制，而Condition与Lock配合完成等待通知机制，前者是Java底层级别的，后者是语言级别的，具有更高的可控制性和扩展性。两者除了在使用方式上不同外，在功能特性上还是有很多的不同：

1. Condition能够支持不响应中断，而通过使用Object方式不支持
2. Condition能够支持一个同步队列**多个等待队列**（new 多个Condition对象），而Object方式只能支持一个同步队列一个等待队列
3. Condition能够支持超时时间的设置，而Object不支持

#### 实现原理

创建一个Condition对象是通过`lock.newCondition()`，而这个方法实际上是会new出一个**ConditionObject**对象，该类是AQS（AbstractQueuedSynchronizer）的一个内部类。

AQS内部维护了一个同步队列，如果是独占式锁的话，所有获取锁失败的线程的尾插入到**同步队列**，同样的，Condition内部也是使用同样的方式，内部维护了一个**等待队列**，所有调用condition.await方法的线程会加入到等待队列中，并且线程状态转换为等待状态。

当调用Condition.await()方法后会使得当前获取lock的线程进入到**等待队列**，如果该线程能够从await()方法返回的话一定是该线程获取了与condition相关联的lock。

调用Condition的signal方法可以将**等待队列**中等待时间最长的节点移动到**同步队列**中，使得该节点能够有机会获得lock。signalAll将时间等待队列中的每一个节点都移入到同步队列中。

![]({{"/assets/images/post/condition.png" | absolute_url }})

1. 线程awaitThread先通过lock.lock()方法获取锁成功后调用了condition.await方法进入**等待队列**
2. 另一个线程signalThread通过lock.lock()方法获取锁成功后调用了condition.signal或者signalAll方法，使得线程awaitThread能够有机会从**等待队列**移入到**同步队列**中，从而使得线程awaitThread能够有机会获取lock，然后从await方法中退出，继续执行后续操作
3. 如果awaitThread获取lock失败会直接进入到**同步队列**

参考：

[详解Condition的await和signal等待/通知机制](https://juejin.im/post/5aeea5e951882506a36c67f0)





