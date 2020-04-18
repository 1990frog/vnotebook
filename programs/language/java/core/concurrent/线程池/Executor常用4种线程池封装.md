[TOC]

静态工厂模式

# 常用线程池构造参数
|   Parameter   |   FixedThreadPool    | CachedThreadPool  | ScheduledThreadPool |   SingleThreaded    |
| ------------- | -------------------- | ----------------- | ------------------- | ------------------- |
| corePoolSize  | n                   | 0                 | contructor-arg      | 1                   |
| maxPoolSize   | n | Integer.MAX_VALUE | Integer.MAX_VALUE   | 1                   |
| keepAliveTime | 0 seconds            | 60 seconds        | 0 seconds           | 0 seconds           |
| workQueue     | LinkedBlockingQueue  | SynchronousQueue  | DelayedWorkQueue    | LinkedBlockingQueue |

# newFixedThreadPool
定长线程池（core==max），queue无容量限制（易内存泄露）

参数
+ corePoolSize/maxPoolSize：自定义
+ keepAliveTime：不设置
+ workQueue：LinkedBlockingQueue无界队列

源码
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

注意
由于传进去的LinkedBlockingQueue是没有容量上限的，所以当请求数越来越多，并且无法及时处理完毕的时候，也就是请求堆积的时候，会容易造成占用大量的内存，可能会导致OOM(OutOfMemoryError)

测试：
```
设置VM options: -Xmx8m -Xms8m
```

# newSingleThreadExecutor
单线程池，queue无容量限制

+ 单线程的线程池：它只会用唯一的工作线程来执行任务
+ 它的原理和FixedThreadPool是一样的，但是此时的线程数量被设置为了1

参数
+ corePoolSize/maxPoolSize：1
+ keepAliveTime：不设置
+ workQueue：LinkedBlockingQueue无界队列

源码
```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>(),
                                threadFactory));
}
```

注意
可以看出，这里和刚才的newFixedThreadPool的原理基本一样，只不过把线程数直接设置成了1，所以这也会导致同样的问题，也就是当请求堆积的时候，可能会占用大量的内存

# CachedThreadPool
可缓存线程池，无界线程池，自带回收机制

没有线程容量的限制，当分配新任务就会创建一个新的线程，超过 keepAliveTime 60s还没有新的任务，过期自动销毁多余线程池

参数
+ corePoolSize：0
+ maxPoolSize：Integer.MAX_VALUE
+ keepAliveTime：60秒
+ workQueue：SynchronousQueue零容量，立即交换队列

源码
```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>(),
                                  threadFactory);
}
```

注意
+ 可缓存线程池
+ 无界线程池，具有自动回收多余线程的功能（60s就挂了）
+ 这里的弊端在于第二个参数maximumPoolSize被设置为Integer.MAX_VALUE，这可能会创建数量非常多的线程，甚至导致OOM

# ScheduledThreadPool
支持定时及周期性任务执行的线程池

参数
+ command：Runnable
+ initialDelay：延迟首次执行的时间
+ period：两次执行之间的时间间隔
+ unit：时间单位

源码
```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

public static ScheduledExecutorService newScheduledThreadPool(
        int corePoolSize, ThreadFactory threadFactory) {
    return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
}
```
ScheduledExecutorService.java
```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue());
}
```

# workStealingPool是jdk1.8加入的（有点难哦ForkJoinPool）
+ 这个线程池和之前的都有很大不同
+ 子任务
+ 窃取

不保障顺序，如果任务是递归的，会产生子任务适合这种线程池