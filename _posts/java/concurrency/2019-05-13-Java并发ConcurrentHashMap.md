---
layout: post
title: "Java并发ConcurrentHashMap"
date: 2019-05-13 20:34:11 +0800
description: ""
category: java/concurrency
tags: []
---

Hashmap线程不安全，多线程环境下，使用Hashmap进行put操作会引起死循环；而HashTable对基本所有方法采用synchronized进行控制，高并发的情况下，每次只有一个线程能够获取对象监视器锁，并发性能低下。

ConcurrentHashMap使用锁分段技术，将数据分成一段一段的存储，给每一段配一把锁，当一个线程占用锁访问其中某个段数据时，不影响其他段数据的访问。

#### Java 1.7

Java 1.7版本的ConcurrentHashMap采用`Segment` + `HashEntry`的方式实现。

![]({{"/assets/images/post/concurrenthashmap-1.7.png" | absolute_url }})

###### 初始化

`ConcurrentHashMap`初始化时，计算出`Segment`数组的大小`ssize`和每个`Segment`中`HashEntry`数组的大小`cap`，并初始化`Segment`数组的第一个元素；其中`ssize`大小为2的幂次方，默认为16，`cap`大小也是2的幂次方，最小值为2，最终结果根据根据初始化容量`initialCapacity`进行计算。

其中`Segment`在实现上继承了`ReentrantLock`，这样就自带了锁的功能。 

###### put实现

当执行`put`方法插入数据时，根据key的hash值，在`Segment`数组中找到相应的位置，如果相应位置的`Segment`还未初始化，则通过CAS进行赋值，接着执行`Segment`对象的`put`方法通过加锁机制插入数据。当线程A和线程B同时执行相同`Segment`对象的`put`方法时：

1. 线程A执行`tryLock()`方法成功获取锁，则把`HashEntry`对象插入到相应的位置
2. 线程B获取锁失败，则执行`scanAndLockForPut()`方法，在`scanAndLockForPut`方法中，会通过重复执行`tryLock()`方法尝试获取锁，在多处理器环境下，重复次数为64，单处理器重复次数为1，当执行`tryLock()`方法的次数超过上限时，则执行`lock()`方法挂起线程B
3. 当线程A执行完插入操作时，会通过`unlock()`方法释放锁，接着唤醒线程B继续执行

###### get实现

`get`操作实现非常简单和高效，通过hash值，找到对应的`Segment`，再定位到对应的`HashEntry`。

`get`操作的高效之处在于整个`get`过程**不需要加锁**，除非读到的值是空的才会加锁重读，不需要加锁的原因是它的`get`方法里将要使用的共享变量都定义成volatile，如用于统计当前`Segment`大小的count字段和用于存储值的`HashEntry`的value。

定义成volatile的变量，能够在线程之间保持可见性，能够被多线程同时读，并且保证不会读到过期的值，但是只能被单线程写（有一种情况可以被多线程写，就是写入的值不依赖于原值），在get操作里只需要读不需要写共享变量count和value，所以可以不用加锁。根据Java内存模型的happens before原则，对volatile字段的写入操作先于读操作，即使两个线程同时修改和获取volatile变量，get操作也能拿到最新的值，这是用volatile替换锁的经典应用场景。 

###### size实现

因为`ConcurrentHashMap`是可以并发插入数据的，所以在准确计算元素时存在一定的难度，一般的思路是：统计每个`Segment`对象中的元素个数，然后进行累加。但是这种方式计算出来的结果并不一样的准确的，因为在计算后面几个`Segment`的元素个数时，已经计算过的`Segment`同时可能有数据的插入或者删除，在1.7的实现中，采用了**先乐观，再悲观**的方式：

先采用不加锁的方式，连续计算元素的个数，最多计算3次：

- 如果前后两次计算结果相同，则说明计算出来的元素个数是准确的
- 如果前后两次计算结果都不同，则给每个`Segment`进行加锁，再计算一次元素的个数

####  Java 1.8

Java 1.8版本放弃了`Segment`臃肿的设计，取而代之的是采用`Node` + `CAS` + `Synchronized`来保证并发安全进行实现，底层数据结构改变为采用**数组+链表+红黑树**的数据形式。由于synchronzied做了很多的优化，包括偏向锁，轻量级锁，因此，使用synchronized相较于ReentrantLock的性能会持平甚至在某些情况更优。

![]({{"/assets/images/post/concurrenthashmap-1.8.png" | absolute_url }})

第一次调用`put`方法时才会调用`initTable()`初始化`Node`数组。

###### put实现

ConcurrentHashMap是一个哈希桶数组，如果不出现哈希冲突的时候，每个元素均匀的分布在哈希桶数组中。当出现哈希冲突的时候，是**标准的链地址的解决方式**，将hash值相同的节点构成链表的形式，称为“拉链法”；为了防止拉链过长，当链表的长度大于8的时候会将链表转换成红黑树。

Node数组中的每个元素实际上是单链表的头结点或者红黑树的根节点。

1. 根据key的hash值定位到要插入的桶，即插入Node数组的索引处
2. 如果当前Node数组还未初始化，先将Node数组进行初始化操作
3. 如果这个位置是null的，那么使用CAS操作直接放入
4. 如果这个位置存在结点，说明发生了hash碰撞，首先判断这个节点的类型。如果该节点fh==MOVED（代表forwardingNode,数组正在进行扩容）的话，说明正在进行扩容
5. 如果是链表节点（fh>0）,则得到的结点就是hash值相同的节点组成的链表的头节点。需要依次向后遍历确定这个新加入的值所在位置。如果遇到key相同的节点，则只需要覆盖该结点的value值即可。否则依次向后遍历，直到链表尾插入这个结点，插入完节点之后再次检查链表长度，如果长度大于8，就把这个链表转换成红黑树
6. 如果这个节点的类型是TreeBin的话，直接调用红黑树的插入方法进行插入新的节点
7. 对当前容量大小进行检查，如果超过了临界值（实际大小*加载因子）就需要扩容 

###### get实现

1. 当前的hash桶数组节点即Node[i]是否为查找的节点，若是则直接返回
2. 若不是，则继续再看当前是不是树节点？通过看节点的hash值是否为小于0，如果小于0则为树节点。如果是树节点在红黑树中查找节点
3. 如果不是树节点，那就只剩下为链表的形式的一种可能性了，就向后遍历查找节点，若查找到则返回节点的value即可，若没有找到就返回null

###### size实现

使用一个`volatile`类型的变量`baseCount`记录元素的个数，当插入新数据或则删除数据时，会通过`addCount()`方法更新`baseCount`。

1. 初始化时`counterCells`为空，在并发量很高时，如果存在两个线程同时执行`CAS`修改`baseCount`值，则失败的线程会继续执行方法体中的逻辑，使用`CounterCell`记录元素个数的变化
2. 如果`CounterCell`数组`counterCells`为空，调用`fullAddCount()`方法进行初始化，并插入对应的记录数，通过`CAS`设置cellsBusy字段，只有设置成功的线程才能初始化`CounterCell`数组
3. 如果通过`CAS`设置cellsBusy字段失败的话，则继续尝试通过`CAS`修改`baseCount`字段，如果修改`baseCount`字段成功的话，就退出循环，否则继续循环插入`CounterCell`对象
4. 因为元素个数保存`baseCount`中，部分元素的变化个数保存在`CounterCell`数组中，通过累加`baseCount`和`CounterCell`数组中的数量，即可得到元素的总个数

###### 总结

- 不采用segment而采用node，**锁住node来实现减小锁粒度**，同时防止拉链过长导致性能下降，当链表长度大于8的时候**采用红黑树**的设计 
- 采用synchronized而不是ReentrantLock
- 设计了MOVED状态，当resize过程中线程2还在put数据，线程2会帮助resize
- 使用3个CAS操作来确保node的一些操作的原子性，这种方式代替了锁
- sizeCtl的不同值来代表不同含义，起到了控制的作用

参考：

[谈谈ConcurrentHashMap1.7和1.8的不同实现](https://www.jianshu.com/p/e694f1e868ec)

[并发容器之ConcurrentHashMap(JDK1.8版本)](https://juejin.im/post/5aeeaba8f265da0b9d781d16)

