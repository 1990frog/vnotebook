[TOC]

Lock和synchronized，这两个是最常见的锁，它们都可以达到线程安全的目的，但是在使用上和功能上又有较大的不同

# Synchronized
优点：
自动释放锁
缺点：
+ 效率低锁释放情况少、试图获得锁时不能设定超时、不能中断一个正在试图获得锁的线程
+ 加锁和释放的时机单一，每个锁仅有单一的条件（某个对象），可能是不够的
+ 无法知道是否成功获取到锁

补充： Java并发编程这个领域中synchronized关键字一直都是元老级的角色，很久之前很多人都会称它为 “重量级锁” 。但是，在JavaSE 1.6之后进行了主要包括为了减少获得锁和释放锁带来的性能消耗而引入的 `偏向锁` 和 `轻量级锁` 以及其它各种优化之后变得在某些情况下并不是那么重了。
synchronized的底层实现主要依靠 `Lock-Free` 的队列，基本思路是 自旋后阻塞，竞争切换后继续竞争锁，稍微牺牲了公平性，但获得了高吞吐量。
在线程冲突较少的情况下，可以获得和CAS类似的性能；而线程冲突严重的情况下，性能远高于CAS。

# Lock接口（还有个Lock类呢）
+ Lock并不是用来代替synchronized的，而是当使用synchronized不适合或不足以满足要求的时候，来提供高级功能的。
+ Lock接口最常见的实现类是ReentrantLock
+ 通常情况下，Lock只允许一个线程来访问这个共享资源。不过有的时候，一些特殊的实现也允许并发访问，比如：ReadWriteLock里面的ReadLock

Lock接口源码：
```java
public interface Lock {
    // lock方法就是最普通的获取锁。如果锁已被其他线程获取，则进行等待
    void lock();
    // 如果当前线程未被中断，则获取锁
    void lockInterruptibly() throws InterruptedException;
    // 用来尝试获取锁，如果当前锁没有被其他线程占用，则获取成功，则返回true，否则返回false，代表获取锁失败
    boolean tryLock();
    // 与trylock差不多，加个超时
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    // 释放锁
    void unlock();
    Condition newCondition();
}
```
## lock() 阻塞
1. lock方法就是最普通的获取锁。如果锁已被其他线程获取，则进行等待（阻塞）
2. lock方法不会像synchronized一样在异常时自动释放锁，因此最佳实践是，在finally中释放锁，以保证发生异常时锁一定被释放
3. lock方法不能被中断，这会带来很大的隐患：一旦陷入死锁，lock()就会陷入永久等待
## tryLock() 非阻塞
1. tryLock()用来尝试获取锁，如果当前锁没有被其他线程占用，则获取成功，则返回true，否则返回false，代表获取锁失败
2. 相比于lock，这样的方法显然功能更强大了，我们可以根据是否能获取锁来决定后续程序的行为，该方法会立即返回，即便在拿不到锁时不会一直在那等
## tryLock(long time,TimeUnit unit)
尝试获取锁，超时就放弃
## lockInterruptibly()
如果当前线程未被中断，那就一直尝试获取锁。<font color="red">在等待锁的过程中，线程就可以被中断</font>
## unlock()
释放锁
先解锁再写业务逻辑
# 可见性保证
Lock的加解锁和synchronized有同样的内存语义，也就是说，下一个加锁前可以看到所有前一个解锁后发生的所有语句



