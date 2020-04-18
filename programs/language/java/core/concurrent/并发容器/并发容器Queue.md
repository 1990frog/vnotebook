[TOC]

![](_v_images/20200416224417853_41126944.png)

# ConcurrentHashMap和CopyOnWriteArrayList
+ 同步的ArrayList（时代巨轮滚滚向前）
+ 绝大多数并发情况下，ConcurrentHashMap和CopyOnWriteArrayList的性能都更好（CopyOnWriteArrayList写多情况除外，Collection.synchronizedList更适合写多。而ConcurrentHashMap任何情景之下都强于其他方法）

# Map简介
+ HashMap
+ Hashtable
+ LinkedHashMap
+ TreeMap


# 并发队列Queue
阻塞队列
非阻塞队列

# 为什么要使用队列
+ 用队列可以在线程间传递数据：生产者消费者模式、银行转账
+ 考虑锁等线程安全问题的重任从“你”转移到了“队列”上

# 并发队列简介
+ Queue
+ BlockingQUeue

![](_v_images/20200213114134655_778768935.png)


# 什么是阻塞队列
+ 阻塞队列是具有阻塞功能的队列，所以它首先是一个队列，其次是具有阻塞功能
+ 通常，阻塞队列的一端是给生产者放数据用，另一端给消费者拿数据用。阻塞队列是线程安全的，所以生产者和消费者都可以是多线程的
+ take()方法：获取并移除队列的头结点，一旦如果执行take的时候，队列里无数据，则阻塞，直到队列里有数据
+ put()方法：插入元素。但是如果队列已满，那么就无法继续插入，则阻塞，知道队列里有了空闲空间
+ 是否有界（容量有多大）：这是一个非常重要的属性，无界队列意味着里面可以容纳非常多（Integer.MAX_VALUE，约为2的31次，是非常大的一个数，可以近似认为是无限容量）
+ 阻塞队列和线程池的关系：阻塞队列是线程池的重要组成部分

# BlockingQueue主要方法
+ put
+ take
+ add
+ remove
+ element
+ offer
+ poll
+ peek

# ArrayBlockingQueue
+ 有界
+ 指定容量
+ 公平：还可以指定是否要保证公平，如果想保证公平的话，那么等待了最长时间的线程会被优先处理，不过这会同时带来一定的性能消耗

![](_v_images/20200213114150607_1639988253.png)


# LinkedBlockingQueue
+ 无界
+ 容量Integer.MAX_VALUE
+ 内部结构：Node、两把锁。分析put方法

# PriorityBlockingQueue
+ 支持优先级
+ 自然顺序（而不是先进先出）
+ 无界队列
+ PriorityQueue的线程安全版本

# SynchronousQueue
+ 它的容量为0
+ 需要注意的是，SynchronousQueue的容量不是1而是0，因为SynchronousQueue不需要去持有元素，它所做的就是直接传递（direct handoff）
+ 效率很高
+ SynchronousQueue没有peek等函数，因为peek的含义是取出头结点，但是SynchronousQueue的容量是0，所以连头结点都没有，也就没有peek方法。同理，没有iterate相关方法
+ 是一个极好的用来直接传递的并发数据结构
+ SynchronousQueue是线程池Executors.newCachedThreadPool()使用的阻塞队列

# DelayQueue
+ 延迟队列，根据延迟时间排序
+ 元素需要实现Delayed接口，规定排序规则

# 非阻塞并发队列
+ 并发包中的非阻塞队列只有ConcurrentLinkedQueue这一种，顾名思义ConcurrentLinkedQueue是使用链表作为其数据结构的，使用CAS非阻塞算法来实现线程安全（不具备阻塞功能），适合用在对性能要求较高的并发场景。用到相对比较少一些
+ 看源码的offer方法的CAS思想，内有p.casNext方法，用了UNSAFE.compareAndSwapObject

# 如何选择适合自己的队列
+ 边界
+ 空间
+ 吞吐量