[TOC]

# 考虑的要素
粒度
竞争强弱（造成自旋）

# cas的快速路径与慢速路径，还有锁的
更新原子变量的快速（非竞争）路径不会比获取锁的快速路径慢，并且通常会非常快，而它的慢速路径肯定比锁的慢速路径快，因为它不需要挂起或重新调度线程

# Compare And Swap
CAS包括了3个操作数——需要读写的内存位置V、进行比较的值A和要写入的新值B。当且仅当V的值等于A时，CAS才会通过原子方式用新值B来更新V的值，否则不会执行任何操作。无论位置V的值是否等于A，都将返回V原有的值。（这种变化形式被称为比较并设置，无论操作是否成功都会返回。）

CAS的含义是：我认为V的值应该为A，如果是，那么将V的值更新为B，否则不修改并告诉V的值实际为多少

CAS是一项乐观的技术，它希望能成功执行更新操作，并且如果有另一个线程在最近一次检查后更新了该变量，那么CAS能检测到这个错误

当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其他线程都将失败，然而，失败的线程并不会被挂起（这与获取锁的情况不同：当获取锁失败时，线程将被挂起），而是被告知在这次竞争中失败，并可以再次尝试。

CAS的典型使用模式是：首先从V中读取值A，并根据A计算新值B，然后再通过CAS以原子方式将V中的值由A变成B（只要在这期间没有任何线程将V的值修改为其他值）。由于CAS能检测到来自其他线程的干扰，因此即使不适用锁也能够实现原子的读——改——写操作序列。

# 什么是CAS
+ 并发
+ 我认为V的值是A，如果是的话那我就把它改成B，如果不是A（说明被别人修改了），那我就不修改了，避免多人同时修改导致出错
+ CAS有三个操作数：内存值V、预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，才将内存值修改为B，否则什么都不做。最后返回现在的V值
+ CPU特殊指令，例如比较并交换指令
+ 非阻塞算法
+ 优点粒度特别细，不存在死锁，自旋

更好的volatile变量

## Unsafe
```java
/**
 * Atomically update Java variable to <tt>x</tt> if it is currently
 * holding <tt>expected</tt>.
 * @return <tt>true</tt> if successful
 */
public final native boolean compareAndSwapObject(Object o, long offset,
                                                 Object expected,
                                                 Object x);

/**
 * Atomically update Java variable to <tt>x</tt> if it is currently
 * holding <tt>expected</tt>.
 * @return <tt>true</tt> if successful
 */
public final native boolean compareAndSwapInt(Object o, long offset,
                                              int expected,
                                              int x);

/**
 * Atomically update Java variable to <tt>x</tt> if it is currently
 * holding <tt>expected</tt>.
 * @return <tt>true</tt> if successful
 */
public final native boolean compareAndSwapLong(Object o, long offset,
                                               long expected,
                                               long x);
```
compareAndSwap是通过c调用cpu原子指令实现的，其返回值为boolean，只需要考虑执行成功或失败两种情景（每次只有1个尝试能成功）

# 偏移量
计算机汇编语言中的偏移量定义为：把存储单元的实际地址与其所在段的段地址之间的距离称为段内偏移，也称为“有效地址或偏移量”。

# 应用场景
+ 乐观锁
+ 并发容器
+ 原子类

# CAS和synchronized适用场景
1. 对于资源竞争较少的情况，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户态内核态间的切换操作额外浪费消耗cpu资源；而CAS基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少，因此可以获得更高的性能。
2. 对于资源竞争严重的情况，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于synchronized。

# 模拟
demo1：
```java
/**
 * 描述：
 * 模拟CAS操作，等价代码
 */
public class SimulatedCAS {
    private volatile int value;

    public synchronized int compareAndSwap(int expectedValue, int newValue) {
        int oldValue = value;
        if (oldValue == expectedValue) {
            value = newValue;
        }
        return oldValue;
    }
}
```
demo2：
```java
public final int getAndIncrement() {    
    for (;;) {    
        int current = get();    
        int next = current + 1;    
        if (compareAndSet(current, next))    
            return current;    
    }    
}     
```

# 分析在Java中是如何利用CAS实现原子操作的？
+ AtomicInteger加载Unsafe工具，用来直接操作内存数据
+ 用Unsafe来实现底层操作
+ 用volatile修身value字段，保证可见性

# 源码
让我们来看看AtomicInteger是如何通过cas实现并发下的累加操作的，以AtomicInteger的getAndAdd方法为突破口。

## 1.getAndAdd方法
```java
public final int getAndAdd(int delta){
    return unsafe.getAndAddInt(this,valueOffset,delta);
}
```
可以看出，这里使用Unsafe这个类，这里需要简要介绍一下Unsafe类：

## 2.Unsafe类
Unsafe是CAS的核心类。Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门，JDK中有一个类Unsafe，它提供了硬件级别的原子操作。
```java
/**
 * Atomically update Java variable to <tt>x</tt> if it is currently
 * holding <tt>expected</tt>.
 * @return <tt>true</tt> if successful
 */
public final native boolean compareAndSwapInt(Object o, long offset,
                                              int expected,
                                              int x);
```
## 3.AtomicInteger加载Unsafe工具，用来直接操作内存数据
```java
public class AtomicInteger extends Number{
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static{
        try{
            //通过unsafe类的objectFieldOffset()方法，获取value变量在对象内存中的偏移
            //通过该偏移量valueOffset，unsafe类的内部方法可以获取到变量value对其进行取值或赋值操作
            valueOffset = unsafe.objectFieldOffset(AtomicInteger.class.getDeclaredField("value"));
        }catch(Exception e){throw new Error(ex);}
    }

    private volatile int value;
    public final int get(){return value;}
}
```
在AtomicInteger数据定义的部分，我们还获取了unsafe实例，并且定义了valueOffset。再看到static块，懂类加载过程的都知道，static块的加载发生于类加载的时候，是最先初始化的，这时候我们调用unsafe的objectFieldOffset从Atomic类文件中获取value的偏移量，那么valueOffset其实就是记录value的偏移量的。

valueOffset表示的是变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的原值的，这样我们就能通过unsafe来实现CAS了。value是用volatile修饰的，保证了多线程之间看到value值是同一份。

## 4.接下来继续看Unsafe的getAndAddInt方法的实现
```java
//Unsafe的getAndAddInt方法分析：自旋+CAS，在这个过程中，通过compareAndSwapInt比较并更新value值，如果更新失败，重新获取，然后再次更新，直到更新成功。
public final int getAndAddInt(Object var1,long var2,int var4){
    int var5;
    do{
        //通过调用unsafe的getIntVolatile(var1,var2)，这个是native方法，其实就是获取var1中，var2偏移量处的值,就是我们前面提到的valueOffset，这样我们就从内存里获取到现在valueOffset处的值了。
        var5 = this.getIntVolatile(var1,var2);
    //其实换成compareAndSwapInt(obj,offset,expect,update)比较清楚，意思就是如果obj内的value和expect相等，就证明没有其他线程改变这个变量，那么就更新它为update，如果这一步的CAS没有成功，那就采用自旋的方式继续进行CAS操作。
    }while(!this.compareAndSwapInt(var1,var2,var5,var5+var4))
    return var5;
}
```
# CAS缺点
+ ABA问题：解决办法，版本号
+ 自旋时间过长

# CAS与Lock的对比，业务情景















