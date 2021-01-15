[TOC]

进阶必备，并发灵魂人物

# 为什么需要AQS
锁和协作类有共同点：闸门
事实上，不仅是ReentrantLock和Semaphore，包括CountDownLatch、ReentrantReadWriteLock都有这样类似的“协作”（或者叫“同步”）功能，其实，它们底层都用了一个共同的基类，这就是AQS
因为上面的那些协作类，它们有很多工作都是类似的，所以如果能提取出一个工具类，那么就可以直接用，对于ReentrantLock和Semaphore而言就可以屏蔽很多细节，只关注它们自己的“业务逻辑”就可以了

# Semaphore和AQS的关系
Semaphore内部有一个Sync类，Sync类继承了AQS，CountDownLatch也是一样

# AQS的比喻
比喻：群面、单面
安排就坐、叫号、先来后到等HR的工作就是AQS做的工作
面试官不会去关心两个面试者是不是号码相同冲突了，也不想去管面试者需要一个地方坐着休息，这些都交给HR去做
Semaphore：一个人面试结束之后，后一个人才能进来继续面试
CountDownLatch：群面，等待10人到齐
Semaphore、CountDownLatch这些同步工具类，要做的就只是写下自己的“要人”规则。比如是“出一个，进一个”，或者说”凑齐10人，一起面试”
剩下的脏话累活交给AQS来做

# 如果没有AQS
就需要每个协作工具自己实现：
+ 同步状态的原子性管理
+ 线程的阻塞与解除阻塞
+ 队列的管理

在并发场景下，自己正确且高效实现这些内容，是相当有难度的，所以我们用AQS来帮我们把这些脏话累活都搞定，我们只关注业务逻辑就够了

# AQS的作用
AQS是一个用于构建锁、同步器、协作工具类的工具类（框架）。有了AQS以后，更多的协作工具类都可以很方便得被写出来（比如等待线程用先进先出的队列操作，有一些标准来判断线程是否应该等待，帮我们处理多个地方的竞争问题，帮我们降低上下文开销提高吞吐量，AQS在设计的时候充分的考虑了这些场景，以及还考虑了性能问题）

# AQS的重要性、地位
AbstractQueuedSynchronizer是Doug Lea写的，从JDK1.5加入的一个基于FIFO等待队列实现的一个用于实现同步器的基础框架，我们用IDE看AQS的实现类，可以发现实现类有以下这些：
![](https://gitee.com/caijingquan/imagebed/raw/master/1610693557_20200203050527860_440403140.png)

# AQS的三要素
+ state
+ 控制线程抢锁和配合的FIFO队列
+ 期望协作工具类去实现的获取/释放等重要方法

# state状态
+ 这里的state的具体含义，会根据具体实现类的不同而不同，比如在Semaphore里，它表“剩余1的许可证的数量”，而在CountDownLatch里，它表示“还需要倒数的数量”
+ state是volatile修饰的，会被并发地修改，所以所有修改state的方法都需要保证线程安全，比如getState、setState以及compareAndSetState操作来读取和更新这个状态。这些方法都依赖于j.u.c.atomic包的支持

在ReentrantLock中，state用来表示“锁”的占有情况，包括可重入计数，当state的值为0的时候，标识改Lock不被任何线程所占有

# Queue
控制线程抢锁和配合的FIFO队列
这个队列用来存放“等待的线程”，AQS就是“排队管理器”，当多个线程争用同一把锁时，必须有排队机制将那些没能拿到锁的线程串在一起。当释放锁时，锁管理器就会挑选一个合适的线程来占有这个刚刚释放的锁
AQS会维护一个等待的线程队列，把线程都放到这个队列里（双向链表实现）
![](https://gitee.com/caijingquan/imagebed/raw/master/1610693559_20200203051740812_1628088130.png)

# 期望协作工具类去实现的获取/
这里的获取和释放方法，是利用AQS的协作工具类里最重要的方法，是由协作类自己去实现的，并且含义各不相同

获取方法
获取操作会依赖state变量，经常会阻塞（比如获取不到锁的时候）
在Semaphore中，获取就是acquire方法，作用是获取一个许可证
而在CountDownLatch里面，获取就是await方法，作用是“等待，直到倒数结束”

释放方法
释放操作不会阻塞
在Semaphore中，释放就是release方法，作用是释放一个许可证
CountDownLatch里面，获取就是countDown方法，作用是“倒数一个数”

其他方法
需要重写tryAcquire和tryRelease等方法




---
# java11为什么去掉unsafe
---

AbstractQueuedSynchronizer简称AQS，是一个用于构建锁和同步容器的框架。事实上concurrent包内许多类都是基于AQS构建，例如ReentrantLock，Semaphore，CountDownLatch，ReentrantReadWriteLock，FutureTask等。AQS解决了在实现同步容器时设计的大量细节问题

AQS使用一个FIFO的队列等待锁的线程，队列头结点称作“哨兵节点”或者“哑节点”，它不与任何线程关联。其他的节点与等待线程关联，每个节点维护一个等待状态waitStatus。

---

![](https://gitee.com/caijingquan/imagebed/raw/master/1610693559_20200424101725632_1526774809.png)

![](https://gitee.com/caijingquan/imagebed/raw/master/1610693560_20200424102411744_1671773247.png)

![](https://gitee.com/caijingquan/imagebed/raw/master/1610693561_20200424102515080_397295155.png)

![](https://gitee.com/caijingquan/imagebed/raw/master/1610693562_20200424103231580_1242172369.png)

![](https://gitee.com/caijingquan/imagebed/raw/master/1610693563_20200424103744113_790553162.png)

![](https://gitee.com/caijingquan/imagebed/raw/master/1610693563_20200424103853401_1366571782.png)