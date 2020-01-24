[TOC]

# 什么是cas
+ 并发
+ 我认为V的值是A，如果是的话那我就把它改成B，如果不是A（说明被别人修改了），那我就不修改了，避免多人同时修改导致出错
+ CAS有三个操作数：内存值V、预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，才将内存值修改为B，否则什么都不做。最后返回现在的V值
+ CPU特殊指令

# CAS等价代码

# 应用场景
+ 乐观锁
+ 并发容器
+ 原子类

# 分析在Java中是如何利用CAS实现原子操作的？
+ AtomicInteger加载Unsafe工具，用来直接操作内存数据
+ 用Unsafe来实现底层操作
+ 用volatile修身value字段，保证可见性

# 源码
让我们来看看AtomicInteger是如何通过cas实现并发下的累加操作的，以AtomicInteger的getAndAdd方法为突破口。

1.getAndAdd方法
```java
public final int getAndAdd(int delta){
    return unsafe.getAndAddInt(this,valueOffset,delta);
}
```
可以看出，这里使用Unsafe这个类，这里需要简要介绍一下Unsafe类：

2.Unsafe类
Unsafe是CAS的核心类。Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门，JDK中有一个类Unsafe，它提供了硬件级别的原子操作。

3.AtomicInteger加载Unsafe工具，用来直接操作内存数据
```java
public class AtomicInteger extends Number{
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static{
        try{
            valueOffset = unsafe.objectFieldOffset(AtomicInteger.class.getDeclaredField("value"));
        }catch(Exception e){throw new Error(ex);}
    }

    private volatile int value;
    public final int get(){return value;}
}
```
在AtomicInteger数据定义的部分，我们还获取了unsafe实例，并且定义了valueOffset。再看到static块，懂类加载过程的都知道，static块的加载发生于类加载的时候，是最先初始化的，这时候我们调用unsafe的objectFieldOffset从Atomic类文件中获取value的偏移量，那么valueOffset其实就是记录value的偏移量的。

valueOffset表示的是变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的原值的，这样我们就能通过unsafe来实现CAS了。value是用volatile修饰的，保证了多线程之间看到value值是同一份。

4.接下来继续看Unsafe的getAndAddInt方法的实现
```java
public final int getAndAddInt(Object var1,long var2,int var4){
    int var5;
    do{
        var5 = this.getIntVolatile(var1,var2);
    }while(!this.compareAndSwapInt(var1,var2,var5,var5+var4))
    return var5;
}
```
我们看var5获取的是什么，通过调用unsafe的getIntVolatile(var1,var2)，这个是native方法，其实就是获取var1中，var2偏移量处的值。var1就是AtomicInteger，var2
就是我们前面提到的valueOffset，这样我们就从内存里获取到现在valueOffset处的值了。

现在重点来了，compareAndSwapInt(var1,var2,var5,var5+var4)其实换成compareAndSwapInt(obj,offset,expect,update)比较清楚，意思就是如果obj内的value和expect相等，就证明没有其他线程改变这个变量，那么就更新它为update，如果这一步的CAS没有成功，那就采用自旋的方式继续进行CAS操作。

Unsafe的getAndAddInt方法分析：自旋+CAS，在这个过程中，通过compareAndSwapInt比较并更新value值，如果更新失败，重新获取，然后再次更新，直到更新成功。

# CAS缺点
+ ABA问题
+ 自旋时间过长

















