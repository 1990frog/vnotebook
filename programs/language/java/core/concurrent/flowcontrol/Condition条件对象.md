[TOC]

# 概览
+ 需要将其包裹在Lock中
+ 当线程1需要等待某个条件的时候，它就去执行condition.await()方法，一旦执行了await()方法，线程就会进入阻塞状态
+ 然后通常会有另外一个线程，假设是线程2，去执行对应的条件，知道这个条件达成的时候，线程2就会去执行condition.signal()方法，这时JVM就会从被阻塞的线程里找，找到那些等待该condition的线程，当线程1就会收到可执行信号的时候，它的线程状态就会变成Runnable可执行状态

# 接口
可以将其看成Lock版本的wait与notify
```java
interface Condition{
    void await() throws InterruptedException;
    void awaitUninterruptibly();
    long awaitNanos(long nanosTimeout) throws InterruptedException;
    boolean await(long time, TimeUnit unit) throws InterruptedException;
    boolean awaitUntil(Date deadline) throws InterruptedException;
    void signal();
    void signalAll();
}
```

# demo
```java
public class ConditionDemo1 {
    private ReentrantLock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    void method1() throws InterruptedException {
        lock.lock();
        try{
            System.out.println("条件不满足，开始await");
            condition.await();
            System.out.println("条件满足了，开始执行后续的任务");
        }finally {
            lock.unlock();
        }
    }

    void method2() {
        lock.lock();
        try{
            System.out.println("准备工作完成，唤醒其他的线程");
            condition.signal();
        }finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ConditionDemo1 conditionDemo1 = new ConditionDemo1();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    conditionDemo1.method2();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
        conditionDemo1.method1();
    }
}
```

# signalAll()和signal()区别
+ signalAll()会唤醒所有的正在等待的线程
+ 但是signal()是公平的，只会唤起那个等待时间最长的线程

# 注意点
+ 实际上，如果说Lock用来代替synchronized，那么Condition就是用来代替相应的Object.wait/notify的，所以在用法和性质上，几乎都一样
+ await方法会自动释放持有的Lock锁，和Object.wait一样，不需要自己手动先释放锁
+ 调用await的时候，必须持有锁，否则会抛出异常，和Object.wait一样