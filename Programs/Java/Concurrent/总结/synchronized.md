# Synchronized的作用
同步方法支持一种简单的策略来防止线程干扰和内存一致性错误：如果一个对象对多个线程可见，则对该对象变量的所有读取或写入都是通过同步方法完成的。

一句话说出Synchronized的作用：
能保证在同一时刻最多只有一个线程执行该段代码，以达到保证并发安全的效果

# Synchronized的地位
+ Synchronizde是Java的关键字，被Java语言原生支持
+ 是最基本的互斥同步手段
+ 是并发编程中的元老级角色，是并发编程的必学内容

# 不使用并发手段会有什么后果？
例子：i++
原因：
count++，它看上去只是一个操作，实际上包含了三个动作：
1.读count
2.将count加1
3.将count的值写入到内存中

# Synchronized的两个用法
对象锁：
包括方发锁（默认锁对象为this当前实例对象）和同步代码块锁（自己指定锁对象）
类锁：
指synchronized修饰静态的方法或指定锁为Class对象

![](_v_images/20191026154123446_824375716.png)

# 类锁
概念（重要） ：Java类可能有很多个对象，但只有一个Class对象
形式1：synchronized加在static方法上
形式2：synchronized（*.class）代码块

只有一个Class对象：Java类可能会有很多个对象，但是只有1个Class对象。
本质：所以所谓的类锁，不过是Class对象的锁而已。

# 多线程访问同步方法的7种情况
1. 两个线程同时访问一个对象的同步方法（同一把锁）
2. 两个线程访问的是两个对象的同步方法（两把不同的锁）
3. 两个线程访问的是synchronized的静态方法（同一把锁：类锁）
4. 同时访问同步方法与非同步方法（非同步方法不受到影响）
5. 访问同一个对象的不同的普通同步方法（同一把锁，串行）
6. 同时访问静态synchronized和非静态synchronized方法（两把锁，一把对象锁、一把类锁）
7. 方法抛出异常后，会释放锁（Lock不会释放锁，synchronized会）

# 7种情况总结：3点核心思想
1. 一把锁只能同时被一个线程获取，没有拿到锁的线程必须等待（对应1、5种情况）；
2. 每个实例都对应有自己的一把锁，不同实例之间互不影响；例外：锁对象是*.class以及synchronized修饰的是static方法的时候，所有对象共用同一把类锁（对象第2、3、4、6种情况）；
3. 无论方法是正常执行完毕或者方法抛出异常，都会释放锁（对应第7种）

在synchronized方法中调用非synchronized方法，是线程安全的吗？
不是，非synchronized方法，可以被多个线程同时访问，所以不是线程安全的

# synchronized性质
1. 可重入
2. 不可中断

什么是可重入：指的是同一线程的外层函数获得锁之后，内层函数可以直接再次获取该锁
类比：
买车上牌需要摇号，如果摇到了号可以给车上牌照，如果家里有多台车，摇不可能摇到一次就给三台车都上牌。这就是不可重入性；如果摇到一个号就可以一直的获取拍照，就是可重入。
JAVA下有两个可重入锁：
synchronized
ReentrantLock
好处：避免死锁、提升封装性
粒度：线程范围而非调用层面（用3种情况来说明和pthread的区别）

证明：
情况1：证明同一个方法是可重入的
情况2：证明可重入不要求是同一方法
情况3：证明可重入不要求是同一个类中的

# 不可中断
一旦这个锁已经被别人获得了，如果我还想获得，我只能选择等待或者阻塞，知道别的线程释放这个锁。如果别人永远不释放锁，那么我只能永远地等下去。
相比之下，未来会介绍Lock类，可以拥有中断的能力，第一点，如果我觉得我等的时间太长了，有权中断现在已经获取到锁的线程的执行；第二点，如果我觉得我等待的时间太长了不想再等了，也可以退出。

# 原理
获取和释放锁的时机：内置锁

# Monitor
加锁和解锁的原理：深入JVM看字节码

```java
/**
 * 反编译字节码
 */
public class Decompilation14 {
    private Object object = new Object();

    public void insert(Thread thread){
        synchronized (object){

        }
    }
}
```
javac o.java//编译
javap -verbose o.class//反汇编

```java
Classfile /home/cai/Code/Java/study/java8/src/main/java/src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14.class
  Last modified 2019-10-28; size 533 bytes
  MD5 checksum 7fa3db82ee15d6b10466d4e604784e9b
  Compiled from "Decompilation14.java"
public class src.main.java.concurrency.moocwukong.synchronizedlock.Decompilation14
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #2.#20         // java/lang/Object."<init>":()V
   #2 = Class              #21            // java/lang/Object
   #3 = Fieldref           #4.#22         // src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14.object:Ljava/lang/Object;
   #4 = Class              #23            // src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14
   #5 = Utf8               object
   #6 = Utf8               Ljava/lang/Object;
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               insert
  #12 = Utf8               (Ljava/lang/Thread;)V
  #13 = Utf8               StackMapTable
  #14 = Class              #23            // src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14
  #15 = Class              #24            // java/lang/Thread
  #16 = Class              #21            // java/lang/Object
  #17 = Class              #25            // java/lang/Throwable
  #18 = Utf8               SourceFile
  #19 = Utf8               Decompilation14.java
  #20 = NameAndType        #7:#8          // "<init>":()V
  #21 = Utf8               java/lang/Object
  #22 = NameAndType        #5:#6          // object:Ljava/lang/Object;
  #23 = Utf8               src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14
  #24 = Utf8               java/lang/Thread
  #25 = Utf8               java/lang/Throwable
{
  public src.main.java.concurrency.moocwukong.synchronizedlock.Decompilation14();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: new           #2                  // class java/lang/Object
         8: dup
         9: invokespecial #1                  // Method java/lang/Object."<init>":()V
        12: putfield      #3                  // Field object:Ljava/lang/Object;
        15: return
      LineNumberTable:
        line 6: 0
        line 7: 4

  public void insert(java.lang.Thread);
    descriptor: (Ljava/lang/Thread;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=2
         0: aload_0
         1: getfield      #3                  // Field object:Ljava/lang/Object;
         4: dup
         5: astore_2
         6: monitorenter
         7: aload_2
         8: monitorexit
         9: goto          17
        12: astore_3
        13: aload_2
        14: monitorexit
        15: aload_3
        16: athrow
        17: return
      Exception table:
         from    to  target type
             7     9    12   any
            12    15    12   any
      LineNumberTable:
        line 10: 0
        line 12: 7
        line 13: 17
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 12
          locals = [ class src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14, class java/lang/Thread, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4
}
SourceFile: "Decompilation14.java"
```

monitorexit数量多于monitorenter？
因为可能正常退出、也可能抛异常退出

# Monditorenter和Monditorexit指令
计数器为0
可重入计数器累计
计数器非0，其他阻塞

# 可重入原理：加锁次数计时器
JVM负责跟踪对象被加锁的次数
线程第一次给对象加锁的时候，计数变为1.每当这个相同的线程在此对象上再次获得锁时，计数会递增
每当任务离开时，计数递减，当计数为0的时候，锁被完全释放

# 可见性原理：java内存模型

# synchronized缺陷
1. 效率低：锁的释放情况少、试图获得锁时不能设定超时、不能中断一个正在试图获得锁的线程
释放锁情况：执行完毕、触发异常
Lock可以设置超时时间、可以中断
2. 不够灵活（读写锁更灵活）：加锁和释放的时机单一，每个所仅有单一的条件（某个对象），可能是不够的
3. 无法知道是否成功获取到锁

# Lock类