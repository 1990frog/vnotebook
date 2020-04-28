[TOC]

# ReentrantLock之fair与nonfair代码层级上来区分，差别就在lock与tryRelease上

# 构造函数
+ true：公平锁
+ false：非公平锁
```java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```
FairSync与NonFaiSync就对应AQS了
---
![](_v_images/20200404030934209_101758345.png)

# AQS
```java
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {

    /** Use serial ID even though all fields transient. */
    private static final long serialVersionUID = 3737899427754241961L;

    /**
     * Empty constructor for use by subclasses.
     */
    protected AbstractOwnableSynchronizer() { }

    /**
     * 当前拥有锁的线程
     */
    private transient Thread exclusiveOwnerThread;

    /**
     * 设置当前拥有锁的线程
     */
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }

    /**
     * 获取当前当前持有锁的线程
     */
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
```
子类两个：
AbstractQueuedSynchronizer
AbstractQueuedLongSynchronizer
# 源码解析
## 类结构
```java
class ReentrantLock{
    abstract static class Sync extends AbstractQueuedSynchronizer {
        // 交给子类实现（公平，非公平差异之处）
        abstract void lock();
        // 对应ReentrantLock的tryLock()方法，公平锁与非公平锁使用一种实现都是不公平的获取锁
        final boolean nonfairTryAcquire(int acquires);
        // 释放锁
        protected final boolean tryRelease(int releases);
        protected final boolean isHeldExclusively();
        final ConditionObject newCondition();
        final Thread getOwner();
        final int getHoldCount();
        final boolean isLocked();
        private void readObject(java.io.ObjectInputStream s)
    }
    static final class NonfairSync extends Sync {}
    static final class FairSync extends Sync {}
}
```

## tryLock方法（不分fair与nofair，反正就是抢）
对应Sync的nonfairTryAcquire方法，公平锁与非公平锁使用同一个方法
```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    // 获取状态，0对应锁已经释放，n对应被重入n次
    int c = getState();
    // 空锁
    if (c == 0) {
        // 通过CAS(Compare-And-Swap)更改锁状态，并将自己设置为锁的拥有者
        if (compareAndSetState(0, acquires)) {
            // 过户
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 我跟我自己抢
    else if (current == getExclusiveOwnerThread()) {
        // +1，好感度+1
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

# Fair lock
套娃开始：
大娃：FairSync
```java
final void lock() {
    acquire(1);
}
```
二娃：AbstractQueuedSynchronizer
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
三娃：AbstractQueuedSynchronizer
```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```


## FairSync
```java
/**
 * Sync object for fair locks
 */
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }

    /**
     * Fair version of tryAcquire.  Don't grant access unless
     * recursive call or no waiters or is first.
     */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        // 获取当前状态，0是没有锁，如果递归获取锁，那么该值（volatile）+1
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```
## NonfairSync
```java
/**
 * Sync object for non-fair locks
 */
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    /**
     * Performs lock.  Try immediate barge, backing up to normal
     * acquire on failure.
     */
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```