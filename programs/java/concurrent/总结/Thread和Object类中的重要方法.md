[TOC]

# yield
作用：释放cpu时间段，如果释放了还是Runnable状态

好多方法都用到yield

yield和sleep区别：是否随时可能再次被调度
sleep阻塞
yield保持竞争状态

主线程执行run：
```java
new CurrentThread().run()
```

wait
sleep
join
notify
notifyAll
stop
interupet
yield


# start()：
在使用 new 关键字创建一个线程后（New 状态），并不表现出任何的线程活动状态（非 New、Terminated 状态，可以使用 isAlive 方法检测线程的活动状态），CPU 也不会执行线程中的代码。
只有在 start() 方法执行后，才表示这个线程可运行了（Runnable 状态），至于何时真正运行还要看线程调度器的调度。
在线程死亡后，不要再次调用 start() 方法。只能对新建状态的线程调用且只能调用一次 start() 方法，否则将抛出 IllegalThreadStateException 异常。

# run()：
启动线程是 start() 方法，而不是 run() 方法。
如果直接调用 run() 方法，这个线程中的代码会被立即执行，多个线程就无法并发执行了。

# join()：
等待该线程完成的方法，其他线程将进入等待状态（Waiting 状态），通常由使用线程的程序（线程）调用，如将一个大问题分割为许多小问题，要等待所有的小问题处理后，再进行下一步操作。

# sleep()：
主动放弃占用的处理器资源，该线程进入阻塞状态（Blocked 状态），指定的睡眠时间超时后，线程进入就绪状态（Runnable），等待线程调度器的调用。

# yield()：
主动放弃占用的处理器资源，线程直接进入就绪状态（Runnable），等待线程调度器的调用。
可能的情况是当线程使用 yield 方法放弃执行后，线程调度器又将该线程调度执行。

# interrupt()：
没有任何强制线程终止的方法，这个方法只是请求线程终止，而实际上线程并不一定会终止，在调用 sleep() 方法时可能会出现 InterruptedException 异常，你可能会想在异常捕获后（try-catch语句中的catch）请求线程终止，而更好的选择是不处理这个异常，抛给调用者处理，所以这个方法并没有实际的用途，还有 isInterrupted() 方法检查线程是否被中断。

# setDaemon()：
设置守护进程，该方法必须在 start() 方法之前调用，判断一个线程是不是守护线程，可以使用 isDaemon() 方法判断。

# setPriority()：
设置线程的优先级，理论上来说，线程优先级高的线程更容易被执行，但也要结合具体的系统。
每个线程默认的优先级和父线程（如 main 线程、普通优先级）的优先级相同，线程优先级区间为 1~10，三个静态变量：MIN_PRIORITY = 1、NORM_PRIORITY = 5、MAX_PRIORITY = 10。
使用 getPriority() 方法可以查看线程的优先级。

# isAlive()：
检查线程是否处于活动状态，如果线程处于就绪、运行、阻塞状态，方法返回 true，如果线程处于新建和死亡状态，方法返回 false。
