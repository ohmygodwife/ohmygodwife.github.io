---
layout: post
title: "Java并发ThreadLocal"
date: 2019-05-14 17:06:17 +0800
description: ""
category: java/concurrency
tags: []
---

ThreadLocal使得每个线程使用自己的对象实例，彼此不会影响达到隔离的作用，从而就解决了对象在被共享访问带来线程安全问题。如果将同步机制和threadLocal做一个横向比较的话，同步机制就是通过控制线程访问共享对象的顺序，而threadLocal就是为每一个线程分配一个该对象，各用各的互不影响。 事实上，这就是一种“**空间换时间**”的方案，每个线程都会都拥有自己的“共享资源”无疑内存会大很多，但是由于不需要同步也就减少了线程可能存在的阻塞等待的情况，从而提高时间效率。

每个线程中有一个`ThreadLocalMap threadLocals`，key为各种`ThreadLocal<T>`，value为具体的值。

#### Entry数据结构

threadLocalMap内部维护了一个Entry类型的table数组。

![]({{"/assets/images/post/threadlocal.jpg" | absolute_url }})

上图中的实线表示强引用，虚线表示弱引用。如图所示，每个线程实例中可以通过threadLocals获取到threadLocalMap，而threadLocalMap实际上就是一个以threadLocal实例为key，任意对象为value的Entry数组。

当我们为threadLocal变量赋值，实际上就是以当前threadLocal实例为key，值为value的Entry往这个threadLocalMap中存放。

#### 内存泄露

需要注意的是Entry中的**key是弱引用**，当threadLocal外部强引用被置为null(`threadLocalInstance=null`)，那么系统GC的时候，根据可达性分析，这个threadLocal实例就没有任何一条链路能够引用到它，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现**key为null**的Entry，就没有办法访问这些key为null的Entry的value。

如果当前线程再迟迟不结束，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value，则value永远无法回收，造成内存泄漏。

当然，如果当前thread运行结束，threadLocal，threadLocalMap，Entry没有引用链可达，在垃圾回收的时候都会被系统进行回收。在实际开发中，会使用线程池去维护线程的创建和复用，比如固定大小的线程池，线程为了复用是不会主动结束的，所以，threadLocal的内存泄漏问题，是应该值得我们思考和注意的问题。

假设threadLocal使用的是强引用，在业务代码中执行`threadLocalInstance=null`操作，以清理掉threadLocal实例的目的，但是因为threadLocalMap的Entry强引用threadLocal，因此在gc的时候进行可达性分析，threadLocal依然可达，对threadLocal并不会进行垃圾回收，这样就无法真正达到业务逻辑的目的，出现逻辑错误。

因此，threadLocal使用Entry弱引用，尽管可能会出现内存泄漏的问题，但是在threadLocal的生命周期里（set，getEntry，remove）里，都会通过`expungeStaleEntry`，`cleanSomeSlots`，`replaceStaleEntry`这三个方法清理掉key为null的脏entry。

避免内存泄漏应该遵循的原则：

1. 每次使用完ThreadLocal，都调用它的remove()方法，清除数据。
2. 在使用线程池的情况下，没有及时清理ThreadLocal，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用ThreadLocal就跟加锁完要解锁一样，用完就清理。

####  set方法

 ThreadLocalMap采用散列表进行实现。散列冲突时，主要采用两种方式： **分离链表法**（separate chaining，散列值相同元素保存到一个链表）和**开放定址法**（open addressing，散列值单元被占用时，从冲突的数组单元开始，依次往后搜索空单元，如果到数组尾部，再从头开始搜索，即环形查找）。其中ThreadLocalMap使用**开放地址法**来处理散列冲突。因为，在ThreadLocalMap中的散列值分散的十分均匀，很少会出现冲突。并且ThreadLocalMap经常需要清除无用的对象，使用纯数组更加方便。

- hashCode是通过nextHashCode()方法实现的，该方法实际上总是用一个AtomicInteger加上0x61c88647来实现的。0x61c88647这个数是有特殊意义的，它能够**保证hash表的每个散列桶能够均匀的分布**，这是`Fibonacci Hashing`，正是能够均匀分布，所以threadLocal选择使用开放地址法来解决hash冲突的问题
- 通过与操作代替取模操作确定插入位置：`key.threadLocalHashCode & (len-1)`，因为哈希表大小总是为2的幂次方，所以相与等同于一个取模的过程，这样就可以通过Key分配到具体的哈希桶中去
- 在set方法的for循环中寻找和当前Key相同的可覆盖entry的过程中通过**replaceStaleEntry**方法解决脏entry的问题。如果当前table[i]为null的话，直接插入新entry后也会通过**cleanSomeSlots**来解决脏entry的问题
- size大于threshold（容量*加载因子）时，进行扩容：新建一个大小为原来数组长度的两倍的数组，然后遍历旧数组中的entry并将其插入到新的hash数组中，注意：在扩容的过程中针对脏entry的话会令value为null，以便能够被垃圾回收器能够回收，解决隐藏的内存泄漏的问题

####  getEntry方法

若当前定位的entry的key和查找的key相同的话就直接返回这个entry，否则的话就是在set的时候存在hash冲突的情况，需要通过getEntryAfterMiss做进一步处理。

通过nextIndex往后环形查找，如果找到和查询的key相同的entry的话就直接返回，如果在查找过程中遇到脏entry的话使用expungeStaleEntry方法进行处理。

#### remove方法

通过往后环形查找到与指定key相同的entry后，先通过clear方法将key置为null后，使其转换为一个脏entry，然后调用expungeStaleEntry方法将其value置为null，以便垃圾回收时能够清理，同时将table[i]置为null。

#### 适用场景 

 不能用于共享对象多线程访问的场景，只适用于**线程内访问对象**（如果共享会导致线程安全问题）的场景。如：

- Hibernate中通过ThreadLocal管理Session，不同的请求线程拥有自己的session，如果将session共享被其他线程访问，会带来线程安全问题
- SimpleDateFormat.parse有线程安全问题，通过ThreadLocal线程内访问

参考：

[并发容器之ThreadLocal](https://juejin.im/post/5aeeb22e6fb9a07aa213404a)

[一篇文章，从源码深入详解ThreadLocal内存泄漏问题](https://www.jianshu.com/p/dde92ec37bd1)







 

 

 

 

 

 

 