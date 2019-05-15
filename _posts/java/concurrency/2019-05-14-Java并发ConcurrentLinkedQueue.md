---
layout: post
title: "Java并发ConcurrentLinkedQueue"
date: 2019-05-14 09:56:06 +0800
description: ""
category: java/concurrency
tags: []
---

ArrayList不是线程安全的，Vector是线程安全（类似HashMap线程不安全，HashTable线程安全），vector线程安全的方式，是非常粗暴的在方法上用synchronized独占锁，将多线程执行变成串行化，并发性能低下。

ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列，它采用先进先出的规则对节点进行排序，添加一个元素的时候，它会添加到队列的尾部；获取一个元素时，它会返回队列头部的元素。它采用了`wait－free`算法来实现，该算法在[Michael & Scott算法](http://www.cs.rochester.edu/u/scott/papers/1996_PODC_queues.pdf)上进行了一些修改。

#### 结构

- ConcurrentLinkedQueue由head节点和tail节点组成
- 每个节点（Node）由节点元素（item）和指向下一个节点的引用（next）组成，两者都是用volatile进行修饰的，以保证内存可见性
- 节点与节点之间通过next关联起来，形成链表结构的队列
- 初始状态下head指向一个item为null的Node，tail节点等于head节点

#### 入队

![]({{"/assets/images/post/concurrentlinekedqueue-offer.jpg" | absolute_url }})

```java
if (tail.next == null) {
    cas(tail.next, e); //直接插入入队节点，无需更新tail节点
} else {
    cas(tail.next.next, e);
    cas(tail, e); // 更新tail节点
}
```

入队主要做两件事情：

- 定位尾节点：tail节点不总是指向队列的最后一个节点，也可能是倒数第二个。所以每次入队都必须先通过tail节点来找到尾节点，尾节点可能就是tail节点，也可能是tail节点的next节点。再将入队节点设置成当前队列尾节点的下一个节点
- 更新tail节点：如果tail节点的next节点不为空，将入队节点设置成tail节点

如果让tail节点永远作为队列的尾节点，这样实现代码量非常少，而且逻辑非常清楚和易懂。但是这么做有个缺点就是每次都需要使用循环CAS更新tail节点。

如果能减少CAS更新tail节点的次数，就能提高入队的效率，所以doug lea使用hops变量来控制并减少tail节点的更新频率，并不是每次节点入队后都将 tail节点更新成尾节点，而是当 tail节点和尾节点的距离大于等于常量HOPS的值（默认等于1）时才更新tail节点。

tail和尾节点的距离越长使用CAS更新tail节点的次数就会越少，但是距离越长带来的负面效果就是每次入队时定位尾节点的时间就越长，因为循环体需要多循环一次来定位出尾节点，但是这样仍然能提高入队的效率，因为从本质上来看它通过增加对volatile变量的读操作来减少了对volatile变量的写操作，而对volatile变量的写操作开销要远远大于读操作，所以入队效率会有所提升。

注意：入队方法永远返回`true`，所以不要通过返回值判断入队是否成功！

#### 出队

![]({{"/assets/images/post/concurrentlinekedqueue-poll.jpg" | absolute_url }})

```java
item = head.getItem();
if (item != null) {
    cas(head.item, null);
    return item; //当前head不为null，直接弹出
} else {
    item = head.next.getItem();
    cas(head.next.item, null);
    cas(head, head.next.next); //更新head节点
    return item;
}
```

类似入队，并不是每次出队都更新head节点，当head节点里有元素，直接弹出head节点的元素，而不更新head节点，只有当head节点里没有元素时，出队操作才会更新head节点。这种做法也是通过hops变量来减少使用CAS更新head节点的消耗，从而提高出队效率。 

1. 获取头节点的元素，判断头节点元素是否为空
2. 如果为空，表示另外一个线程已经进行了一次出队操作将该节点的元素取走
3. 如果不为空，则使用CAS的方式将头节点的引用设置成null
4. 如果CAS成功，则直接返回头节点的元素
5. 如果不成功，表示另外一个线程已经进行了一次出队操作更新了head节点，导致元素发生了变化，需要重新获取头节点
6. 如果p的下一个节点也为空，说明这个队列已经空了；如果下一个元素不为空，则将头节点的下一个节点设置成头节点。

参考：

[聊聊并发（六）ConcurrentLinkedQueue的实现原理分析](http://www.infoq.com/cn/articles/ConcurrentLinkedQueue)

