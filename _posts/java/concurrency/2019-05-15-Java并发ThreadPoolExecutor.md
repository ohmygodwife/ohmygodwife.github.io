---
layout: post
title: "Java并发ThreadPoolExecutor"
date: 2019-05-15 19:23:24 +0800
description: ""
category: java/concurrency
tags: []
---

线程很占用系统资源，如果对线程管理不善很容易导致系统问题。因此，在大多数并发框架中都会使用线程池来管理线程，使用线程池管理线程主要有如下好处：

1. **降低资源消耗**。通过复用已存在的线程和降低线程关闭的次数来尽可能降低系统性能损耗
2. **提升系统响应速度**。通过复用线程，省去创建线程的过程，因此整体上提升了系统的响应速度
3. **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性

#### 线程池创建

创建线程池可以指定的参数：

- corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的`prestartAllCoreThreads`方法，线程池会提前创建并启动所有基本线程。
- runnableTaskQueue（任务队列）：用于保存等待执行的任务的阻塞队列。可选：ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue、PriorityBlockingQueue。
- maximumPoolSize（线程池最大大小）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池不会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。
- ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字，Debug和定位问题时非常有帮助。
- RejectedExecutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。默认：AbortPolicy，表示无法处理新任务时抛出异常。其余可选策略：CallerRunsPolicy：只用调用者所在线程来运行任务，DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务，DiscardPolicy：不处理，丢弃掉。
- keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。
- TimeUnit（线程活动保持时间的单位）：可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。

#### 向线程池提交任务

通过`execute`方法提交没有返回值，无法判断任务知否被线程池执行成功。

使用`submit`方法来提交任务，会返回一个Future对象，可以通过这个Future对象来判断任务是否执行成功，通过Future的get方法来获取返回值，get方法会阻塞住直到任务完成，而使用`get(long timeout, TimeUnit unit)`方法则会阻塞一段时间后立即返回，这时有可能任务没有执行完。一旦任务执行结束，任务不能重新启动或取消，除非调用`runAndReset()`方法。

FutureTask实现了Future接口，FutureTask提供如下方法：

- 启动和取消异步任务
- 查询异步任务是否计算结束
- 获取最终的异步任务的结果

使用场景：当一个线程需要等待另一个线程把某个任务执行完后它才能继续执行。

#### 线程池关闭

`shutdown`和`shutdownNow`这两个方法的原理都是遍历线程池中所有的线程，然后依次中断线程。区别：

- shutdown只是将线程池的状态设置成`SHUTDOWN`状态，然后中断所有没有正在执行任务的线程。shutdown会等待运行的线程运行结束优雅退出。
- shutdownNow首先将线程池的状态设置为**STOP**,然后逐个调用线程的`interrupt`方法来中断线程，所以无法响应中断的任务可能永远无法终止，停止所有的正在执行和未执行任务的线程，并返回等待执行任务的列表

因此，`shutdown`方法会将正在执行的任务继续执行完，而`shutdownNow`会直接中断正在执行的任务。只要调用了其中一个，`isShutdown`方法就会返回true。当所有的任务都已关闭后，才表示线程池关闭成功，这时调用`isTerminaed`方法会返回true。

至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，**通常调用shutdown来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow**。

#### 线程池处理流程

![]({{"/assets/images/post/threadpoolexecutor.png" | absolute_url }})

1. 如果当前运行的线程少于核心线程池corePoolSize，则会创建新的线程来执行新的任务
2. 如果运行的线程个数等于或者大于corePoolSize，则会将提交的任务存放到阻塞队列workQueue中
3. 如果当前workQueue队列已满的话，则会创建新的线程来执行任务
4. 如果线程个数已经超过了最大线程池maximumPoolSize，则会使用饱和策略RejectedExecutionHandler来进行处理

任务的执行机制，完全交由`Worker类`，也就是进一步了封装了Thread。向线程池提交任务，无论为ThreadPoolExecutor的execute方法和submit方法，还是ScheduledThreadPoolExecutor的schedule方法，都是

1. 将任务移入到阻塞队列中
2. 通过addWork方法新建了Work类，并通过runWorker方法启动线程
3. 不断的从阻塞对列中获取异步任务执行交给Worker执行，直至阻塞队列中无法取到任务为止

#### 配置线程池策略

###### 任务性质

不同性质的任务可以用不同规模的线程池分开处理

**CPU密集型任务**配置尽可能少的线程数量，如配置Ncpu+1个线程的线程池

**IO密集型任务**则由于需要等待IO操作，线程并不是一直在执行任务，则配置尽可能多的线程，如2*Ncpu

**混合型的任务**，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。我们可以通过`Runtime.getRuntime().availableProcessors()`方法获得当前设备的CPU个数。

###### 任务优先级

优先级（高、中、低）不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先得到执行，注意：如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。

###### 任务执行时间

执行时间（长、中、短）不同的任务可以交给不同规模的线程池来处理，或者也可以使用优先级队列，让执行时间短的任务先执行。 

###### 任务依赖性

依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU。

###### 阻塞队列使用有界队列

有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点，比如几千。假如出现数据库连接问题的时候可以监控线程池及时发现。如果采用无界队列的话，一旦任务积压在阻塞队列中的话就会占用过多的内存资源，甚至会使得系统崩溃。

#### 线程池监控

###### 监控线程池属性

- taskCount：线程池需要执行的任务数量
- completedTaskCount：线程池在运行过程中已完成的任务数量。小于或等于taskCount
- largestPoolSize：线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过。如等于线程池的最大大小，则表示线程池曾经满了
- getPoolSize:线程池的线程数量。如果线程池不销毁的话，池里的线程不会自动销毁，所以这个大小只增不减
- getActiveCount：获取活动的线程数

###### 重写线程池方法

继承线程池并重写线程池的`beforeExecute`，`afterExecute`和`terminated`方法，我们可以在任务执行前，执行后和线程池关闭前干一些事情。如监控任务的平均执行时间，最大执行时间和最小执行时间等。这几个方法在线程池里默认是空方法。

#### ScheduledThreadPoolExecutor

给定延时后执行异步任务或者周期性执行任务，相对于任务调度的Timer来说，其功能更加强大，Timer只能使用一个后台线程执行任务，而ScheduledThreadPoolExecutor则可以通过构造函数来指定后台线程的个数。

ScheduledThreadPoolExecutor最大的特色是能够周期性执行异步任务，实际上是将提交的任务转换成的`ScheduledFutureTask`类，该类重写了run方法：

1. 先判断当前任务是否是周期性任务，如果不是的话（schedule方法调用）就直接调用`run()`方法
2. 如果是周期性任务（scheduleAtFixedRate和scheduleWithFixedDelay方法）执行完之后，就重设下一次任务执行的时间，并通过`runAndReset`将下一次待执行的任务放置到`DelayedWorkQueue`中

DelayedWorkQueue是一个基于堆的数据结构，类似于DelayQueue和PriorityQueue。在执行定时任务的时候，每个任务的执行时间都不同，所以DelayedWorkQueue的工作就是按照执行时间的升序来排列，执行时间距离当前时间越近的任务在队列的前面。堆结构在执行插入和删除操作时的最坏时间复杂度是 O(logN)。

参考：

[聊聊并发（三）Java线程池的分析和使用](http://www.infoq.com/cn/articles/java-threadPool)

[线程池之ScheduledThreadPoolExecutor](https://juejin.im/post/5aeec106518825670a10328a)