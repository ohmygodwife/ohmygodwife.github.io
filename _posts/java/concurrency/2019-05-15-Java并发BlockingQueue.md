---
layout: post
title: "Java并发BlockingQueue"
date: 2019-05-15 15:09:25 +0800
description: ""
category: java/concurrency
tags: []
---

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：

- 在队列空时，获取元素的线程会等待队列变为非空。
- 当队列满时，存储元素的线程会等待队列可用。

阻塞队列常用于**生产者和消费者**的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。

| 方法\处理方式 | 抛出异常  |        返回特殊值         | 一直阻塞 |      超时退出      |
| :-----------: | :-------: | :-----------------------: | :------: | :----------------: |
|   插入方法    |  add(e)   | offer(e)，队列满返回false |  put(e)  | offer(e,time,unit) |
|   移除方法    | remove()  |  poll()，队列空返回null   |  take()  |  poll(time,unit)   |
|   检查方法    | element() |  peek()，队列空返回null   |  不可用  |       不可用       |

#### ArrayBlockingQueue

**数组**实现的**有界**阻塞队列。ArrayBlockingQueue一旦创建，容量不能改变。队头元素是队列中存在时间最长的数据元素，而队尾数据是当前队列最新的数据元素。

###### 公平性

严格按照线程等待的绝对时间顺序，即最先等待的线程能够最先访问到ArrayBlockingQueue。而非公平性则是指访问ArrayBlockingQueue的顺序不是遵守严格的时间顺序，有可能存在：一旦ArrayBlockingQueue可以被访问时，长时间阻塞的线程依然无法访问到ArrayBlockingQueue。**如果需要保证公平性，通常会降低吞吐量**。通过设置构造函数第二个参数为true，可以得到支持公平性的ArrayBlockingQueue。

访问者的公平性是使用**可重入锁（ReentrantLock ）**实现的。

#### LinkedBlockingQueue

**链表**实现的**有界**阻塞队列。与ArrayBlockingQueue相比起来具有**更高的吞吐量**，为了防止LinkedBlockingQueue容量迅速增长，损耗大量内存。通常在创建LinkedBlockingQueue对象时，需要指定其容量，如果未指定，容量等于Integer.MAX_VALUE。

#### PriorityBlockingQueue

支持优先级排序的**无界**阻塞队列。默认情况下元素采用自然顺序进行排序，也可以通过自定义类实现compareTo()方法来指定元素排序规则，或者初始化时通过构造器参数Comparator来指定排序规则。

#### DelayQueue

存放实现了`Delayed`接口的数据的无界阻塞队列，使用PriorityQueue来实现。只有当数据对象的延时时间达到时才能插入到队列进行存储。如果当前所有的数据都还没有达到创建时所指定的延时期，则队列没有队头，并且线程通过poll等方法获取数据元素则返回null。

所谓数据延时期满时，则是通过Delayed接口的`getDelay(TimeUnit.NANOSECONDS)`来进行判定，如果该方法返回的是小于等于0则说明该数据元素的延时期已满。

###### 应用场景

- 缓存系统的设计：保存缓存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从DelayQueue中获取元素时，表示缓存有效期到了。
- 定时任务调度。保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到任务就开始执行，从比如TimerQueue就是使用DelayQueue实现的。

#### SynchronousQueue

不存储元素的阻塞队列。每个插入操作必须等待另一个线程进行相应的删除操作，因此，SynchronousQueue实际上没有存储任何数据元素，因为只有线程在删除数据时，其他线程才能插入数据，同样的，如果当前有线程在插入数据时，线程才能删除数据。SynchronousQueue也可以通过构造器参数来为其指定公平性。

可以将SynchronousQueue看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合于传递性场景，比如在一个线程中使用的数据，传递给另外一个线程使用，SynchronousQueue的吞吐量**高于**LinkedBlockingQueue 和 ArrayBlockingQueue。

#### LinkedTransferQueue

链表实现的无界阻塞队列。比其他BlockingQueue增加了`transfer(E e)/tryTransfer(E e,long timeout,imeUnit unit)`方法，如果当前有线程（消费者）正在调用`take()`方法或者可延时的`poll()`方法进行消费数据时，生产者线程可以调用`transfer`方法将数据传递给消费者线程。如果当前没有消费者线程的话，`transfer`方法是必须等到有消费者线程消费数据时，生产者线程才能够返回。而`tryTransfer`方法能够立即返回结果退出。

#### LinkedBlockingDeque

链表实现的**有界**双向阻塞队列。所谓双向队列指的你可以从队列的两端插入和移出元素。双端队列因为多了一个操作队列的入口，在多线程同时入队时，也就减少了一半的竞争。

相比其他阻塞队列，Deque多了**从队头移除和从队尾插入**的功能，其余接口可以等价使用，比如BlockingQueue的`add`方法和BlockingDeque的`addLast`方法的功能是一样的。

#### ArrayBlockingQueue与LinkedBlockingQueue实现原理比较

**相同点：**ArrayBlockingQueue和LinkedBlockingQueue都是通过**condition通知机制**来实现可阻塞式插入和删除元素，并满足线程安全的特性。

**不同点**：ArrayBlockingQueue插入和删除数据，只采用了一个lock；而LinkedBlockingQueue则是在插入和删除分别采用了`putLock`和`takeLock`，这样可以降低线程由于线程无法获取到lock而进入WAITING状态的可能性，从而提高了线程并发执行的效率。

参考：

[聊聊并发（七）——Java中的阻塞队列](http://www.infoq.com/cn/articles/java-blocking-queue)

[并发容器之ArrayBlockingQueue和LinkedBlockingQueue实现原理详解](https://juejin.im/post/5aeebdb26fb9a07aa83ea17e)