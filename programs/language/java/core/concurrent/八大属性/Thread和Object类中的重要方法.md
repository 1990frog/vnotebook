[TOC]


# Thread
## sleep
<font color="red">主动放弃占用的处理器资源，该线程进入阻塞状态（Blocked 状态），指定的睡眠时间超时后，线程进入就绪状态（Runnable），等待线程调度器的调用。</font>
### 作用
sleep方法可以让线程进入Waiting状态，并且不占用cpu资源，但是不释放锁，直到规定时间后再执行，休眠期间如果被中断，会抛出异常并清除中断状态
我只想让线程在预期的时间执行，其他时候不要占用CPU资源，一旦调用sleep线程就会进入阻塞状态，就<font color="red">不会占用cpu资源</font>。直到它下次被调度起来之后
### 特点
+ 不释放锁（包括synchronize和lock）
+ 和wait不同，始终持有锁，而wait会释放锁
### sleep方法响应中断
1.抛出InterruptedException
2.清除中断状态
### 升级方案TimeUnit
```java
TimeUnit.SECONDS.sleep(100);
```
### sleep与wait对比
相同：
1. wait和sleep方法都可以使线程阻塞，对应线程状态是waiting或time_waiting。
2. wait和sleep方法都可以响应中断Thread.interrupt（）。
不同：
1. wait方法的执行必须在同步方法中进行，而sleep则不需要。
2. 在同步方法里执行sleep方法时，不会释放monitor锁，但是wait方法会释放monitor锁。
3. sleep方法短暂休眠之后会主动退出阻塞，而没有指定时间的wait方法则需要被其他线程中断后才能退出阻塞。
4. wait()和notify()，notifyAll()是Object类的方法，sleep()和yield()是Thread类的方法。

## join
等待该线程完成的方法，其他线程将进入等待状态（Waiting 状态），通常由使用线程的程序（线程）调用，如将一个大问题分割为许多小问题，要等待所有的小问题处理后，再进行下一步操作
## yield
作用：释放我的CPU时间片，但是不会释放锁，也不会陷入阻塞
定位：JVM不保证遵循，为了保证程序的稳定性，一般开发中不使用yield，但是这个方法，并发包的类中运用的场合比较多。
## yield和sleep区别
是否随时可能再次被调度：
+ sleep阻塞
+ yield保持竞争状态（好多方法都用到yield）
## currentThread
## start
在使用 new 关键字创建一个线程后（New 状态），并不表现出任何的线程活动状态（非 New、Terminated 状态，可以使用 isAlive 方法检测线程的活动状态），CPU 也不会执行线程中的代码。
只有在 start() 方法执行后，才表示这个线程可运行了（Runnable 状态），至于何时真正运行还要看线程调度器的调度。
在线程死亡后，不要再次调用 start() 方法。只能对新建状态的线程调用且只能调用一次 start() 方法，否则将抛出 IllegalThreadStateException 异常。
## run
启动线程是 start() 方法，而不是 run() 方法。
如果直接调用 run() 方法，这个线程中的代码会被立即执行，多个线程就无法并发执行了。
## interrupt
没有任何强制线程终止的方法，这个方法只是请求线程终止，而实际上线程并不一定会终止，在调用 sleep() 方法时可能会出现 InterruptedException 异常，你可能会想在异常捕获后（try-catch语句中的catch）请求线程终止，而更好的选择是不处理这个异常，抛给调用者处理，所以这个方法并没有实际的用途，还有 isInterrupted() 方法检查线程是否被中断
## stop
## suspend
## resume
## setDaemon
设置守护进程，该方法必须在 start() 方法之前调用，判断一个线程是不是守护线程，可以使用 isDaemon() 方法判断。
## setPriority
设置线程的优先级，理论上来说，线程优先级高的线程更容易被执行，但也要结合具体的系统。
每个线程默认的优先级和父线程（如 main 线程、普通优先级）的优先级相同，线程优先级区间为 1~10，三个静态变量：MIN_PRIORITY = 1、NORM_PRIORITY = 5、MAX_PRIORITY = 10。
使用 getPriority() 方法可以查看线程的优先级。
## isAlive
检查线程是否处于活动状态，如果线程处于就绪、运行、阻塞状态，方法返回 true，如果线程处于新建和死亡状态，方法返回 false。










# join方法作用、用法

作用：因为新的线程加入了我们，所以我们要等他执行完再出发
用法：main等待thread1执行完毕，注意谁等谁

join注意点
CountDownLatch或CyclicBarrier类

在join期间，线程处于哪种线程状态？
WAITING



主线程执行run：
```java
new CurrentThread().run()
```



