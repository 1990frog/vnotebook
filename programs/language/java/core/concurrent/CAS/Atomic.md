[TOC]

# 什么是元子类，有什么作用
+ 原子类的作用和锁类似，是为了保证并发情况下线程安全。不过原子类相比于锁，有一定的优势
+ 粒度更细：原子变量可以把竞争范围缩小到变量级别，这是我们可以获得的最细粒度的情况了，通常锁的粒度都要大于原子变量的粒度
+ 效率更高：通常，使用原子类的效率会比使用锁的效率更高，除了高度竞争的情况
# 6种原子类
|             类型              |                                        详情                                        |
| ---------------------------- | ---------------------------------------------------------------------------------- |
| Atomic基本类型                 | AtomicInterger整型</br>AtomicLong长整型</br>AtomicBoolean布尔型                         |
| AtomicArray数组类型             | AtomicIntegerArray</br>ArrayAtomicLongArray</br>AtomicReferenceArray                 |
| AtomicReference引用类型        | AtomicReference</br>AtomicStampedReference</br>AtomicMarkableReference               |
| AtomicFieldUpdater升级类型原子类 | AtomicIntegerFieldUupdater</br>AtomicLongFieldUpdater</br>AtomicReferenceFieldUpdater |
| Adder累加器                    | LongAdder</br>DoubleAdder                                                           |
| Accumulator累加器              | LongAccumulator</br>DoubleAccumulator                                               |
# 基本类型原子类
## AtomicInterger
```java
public final void set(int newValue)    //赋值
public final int get()    //获取当前值
public final int getAndSet(int newValue)    //获取当前值并设置新的值
public final int getAndINcrement()    //获取当前值并自增
public final int getAndDecrement()    //获取当前值并自减
public final int getAndAdd(int delta)    //获取当前的值，并加上预期的值
boolean compareAndSet(int expect,int update)    //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
```
# 引用类型原子类
## AtomicReference
AtomicReference类的作用，和AtomicInteger并没有本质区别，AtomicInteger可以让一个整数保证原子性，而AtomicReference可以让一个对象保证原子性，当然，AtomicReference的功能明细比AtomicInteger强，因为一个对象里可以包含很多属性，用法和AtomicInteger类似
## AtomicStampedReference
AtomicStampedReference它内部不仅维护了对象值，还维护了一个时间戳（我这里把它称为时间戳，实际上它可以使任何一个整数，它使用整数来表示状态值）。当AtomicStampedReference对应的数值被修改时，除了更新数据本身外，还必须要更新时间戳。当AtomicStampedReference设置对象值时，对象值以及时间戳都必须满足期望值，写入才会成功。因此，即使对象值被反复读写，写回原值，只要时间戳发生变化，就能防止不恰当的写入。
## AtomicReferenceFieldUpdater

# 升级类型原子类
使用场景：
+ 偶尔需要一个原子get-set操作
+ 一个类不是我们编写的，但是我们想并发使用
## AtomicIntegerFieldUpdater注意点
+ 可见范围
+ 不支持static（基于反射）

# Adder累加器
+ 是java8引入的，相对是比较新的一个类
+ 高并发下LongAdder比AtomicLong效率高，不过本质是空间换时间
+ 竞争激烈的时候，LongAdder把不同线程对应到不同的Cell上进行修改，降低冲突的概率，是多段锁的理念，提高了并发性
+ AtomicLong由于竞争很激烈，每一次加法，都要flush和refresh，导致很耗费资源

# LongAdder带来的改进和原理
+ 在内部，这个LongAdder的实现原理和刚才的AtomicLong是有不同的，刚才的AtomicLong的实现原理是，每一次加法都需要做同步，所以在高并发的时候会导致冲突比较多，也就降低了效率
+ 而此时的LongAdder，每个线程会有自己的一个计数器，仅用来在自己线程内计数，这样一来就不会和其他线程的计数器干扰
+ 第一个线程的计数器数值也就是ctr'，为1的时候，可能线程2的计数器ctr''的数值已经是3了，他们之间并不存在竞争关系，所以在加和的过程中，根本不需要同步机制，也不需要刚才的flush和refresh。这里也没有一个公共的counter来给所有线程统一计数

# LongAdder带来的改进和原理
+ LongAdder引入了分段累加的概念，内部有一个base变量和一个Cell[]数组共同参与计数
+ base变量：竞争不激烈，直接累加到该变量上
+ Cell[]：竞争激烈，各个线程分散累加到自己的槽Cell[i]中

# 对比AtomicLong和LongAdder
+ 在低争用下，AtomicLong和LongAdder这两个类具有相似的特征。但是在竞争激烈的情况下，LongAdder的预期吞吐量要高得多，但要消耗更多的空间
+ LongAdder适合的场景是统计求和计数的场景，而且LongAdder基本只提供了add方法，而AtomicLong还具有cas方法

# Accumulator累加器
Accumulator和Adder非常相似，Accumulator就是一个更通用版本的Adder

# Unsafe类