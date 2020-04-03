[TOC]

# 线程内表现为串行的语义
`Within Thread As-If-Serial Semantics`
# 定义
## 什么是happens-before
happens-before规则是用来解决可见性问题的：在时间上，动作A发生在动作B之前，B保证能看到A，这就是happens-before。
一个操作可以用happens-before于另一个操作，那么我们说第一个操作对应第二个操作是可见的。

中断：一个线程被其他线程interrupt，那么检查中断（isInterrupted）或者抛出InterruptedException一定能看到
构造方法：对象构造方法的最后一行指令happens-before于finalize()方法的第一行指令
## 什么不是happens-before
两个线程没有相互配合的机制，所以代码x和y的执行结果并不能保证被对方看到的，这就不具备happens-before。
## 结论
普通的变量仅仅会保证在该方法的执行过程中**所有依赖赋值结果的地方**都能获取到正确的结果，而不能保证变量**赋值操作的顺序**与程序代码中的执行顺序一致。
# 小栗子
```java
int a = 1;
int b = 2;
int c = a + b;
```
假如没有重排序这个东西，CPU肯定会按照从上往下的执行顺序执行：
1. a = 1
2. b = 2
3. c = a + b

但是，上文也提及了：CPU为了提高运行效率，在执行时序上不会按照刚刚所说的时序执行，很有可能是
1. b = 2
2. a = 1
3. c = a + b

因为只需要在变量c需要变量a+b的时候能够得到正确的值就行了，JVM允许这样的行为。 这种现象就是线程内表现为串行的语义。

# 重排序
指令重排序为了提高运行效率，CPU允许将多条指令不按照程序规定的顺序分开发送给各相应电路单元处理。 这里需要注意的是指令重排序并不是将指令任意的发送给电路单元，而是需要满足线程内表现为串行的语义（Within Thread As-If-Serial Semantics）
<font color="red">注意任何代码都有可能出现指令重排序的现象，与是否多线程条件下无关。在单线程内感受不到是因为单线程内会有线程内表现为串行的语义的限制。</font>

# Happens-Before（先行发生）
Happens-Before原则是判断数据是否存在竞争、线程是否安全的主要依据
为了叙述方便，如果操作X Happens-Before 操作Y，那么我们记为`hb(X,Y)`

如果存在`hb(a,b)`，那么操作a在内存上面所做的操作（如赋值操作等）都对操作b可见，即操作a影响了操作b。
+ Java内存模型中定义的两项操作之间的偏序关系，满足偏序关系的各项性质。我们都知道偏序关系中有一条很重要的性质：传递性，所以Happens-Before也满足传递性。这个性质非常重要，通过这个性质可以推导出两个没有直接联系的操作之间存在Happens-Before关系，如： 如果存在hb(a,b)和hb(b,c)，那么我们可以推导出hb(a,c)，即操作a Happens-Before 操作c
+ 判断数据是否存在竞争、线程是否安全的主要依据

这是《深入理解Java虚拟机》，375页的例子：
```java
i = 1;        //在线程A中执行
j = i;        //在线程B中执行
i = 2;        //在线程C中执行
```
假设线程A中的操作【i = 1】先行发生线程B的操作【j = i】，那么可以确定在线程B的操作执行后，变量【j】的值一定等于1，得出这个结论的依据有两个：
+ 一是根据先行发生原则，i = 1的结果可以被观察到
+ 二是线程C还没有“登场“，线程A操作结束之后没有其他的线程会修改变量i的值

现在再来考虑线程C，我们依然保持线程A和线程B之间的先行发生关系，而线程C出现在线程A和线程B的操作之间，但是线程C与线程B没有先行发生关系，那j的值会是多少呢？
答案是不确定：1和2都有可能

因为线程C对变量【i】的影响可能会被线程观察到，也可能不会，这时候线程B就存在读取到过期数据的风险，不具备多线程安全性。 通过这个例子我相信读者对Happens-Before已经有了一定的了解。

这里再重复一下Happens-Before的作用： **如果存在hb(a,b)，那么操作a在内存上面所做的操作（如赋值操作等）都对操作b可见，即操作a影响了操作b**

# Java原生存在的Happens-Before
这些是Java 内存模型下存在的原生Happens-Before关系，无需借助任何同步器协助就已经存在，可以在编码中直接使用。
## 程序次序规则（Program Order Rule）
在一个线程内，按照程序代码顺序，书写在前面的操作Happens-Before书写在后面的操作
## 管程锁定规则（Monitor Lock Rule）
An unlock on a monitor happens-before every subsequent lock on that monitor. 
一个unlock操作Happens-Before后面对同一个锁的lock操作。
## volatile变量规则（volatile Variable Rule）
A write to a volatile field happens-before every subsequent read of that volatile. 
对一个volatile变量的写入操作Happens-Before后面对这个变量的读操作。
## 线程启动规则（Thread Start Rule）
Thread对象的start()方法Happens-Before此线程的每一个动作。
## 线程终止规则（Thread Termination Rule）
线程中的所有操作都Happens-Before对此线程的终止检测。
## 线程中断规则（Thread Interruption Rule）
对线程interrupt()方法的调用Happens-Before被中断线程的代码检测到中断事件的发生，可以通过Thread.interrupt()方法检测到是否有中断发生。
## 对象终结规则（Finalizer Rule）
一个对象的初始化完成（构造函数执行结束）Happens-Before它的finalize()方法的开始。
## 传递性（Transitivity） 偏序关系的传递性
如果已知hb(a,b)和hb(b,c)，那么我们可以推导出hb(a,c)，即操作a Happens-Before 操作c。
```java
int a = 1;
volatile int b = 2;
private void change(){
    a = 3;
    b = a;
}
```
我们只对b一个变量加volatile就能达到效果
b加了volatile，后面想读取到b的时候，就能看到b写入之前的所有操作
近朱者赤：给b加了volatile，不仅b被影响，也可以实现轻量级同步
b之前的写入（对应代码b=a）对读取b后的diam（print  b）都可见，所以在writerThread里对a的赋值，一定会对readerThead里的读取可见，所以这里的a即使不加volatile，只要b读到是3，就可以由happens-before原则保证了读取到的都是3而不可能读取到1。
## 总结
这些规则都很好理解，在这里就不进行过多的解释了。 Java语言中无需任何同步手段保障就能成立的先行发生规则就只有上面这些了。

# 还存在其它的Happens-Before？
Java中原生满足Happens-Before关系的规则就只有上述8条，但是我们还可以通过它们推导出其它的满足Happens-Before的操作，如：

+ 将一个元素放入一个线程安全的队列的操作Happens-Before从队列中取出这个元素的操作
+ 将一个元素放入一个线程安全容器的操作Happens-Before从容器中取出这个元素的操作
+ 在CountDownLatch上的倒数操作Happens-Before CountDownLatch#await()操作
+ 释放Semaphore许可的操作Happens-Before获得许可操作
+ Future表示的任务的所有操作Happens-Before Future#get()操作
+ 向Executor提交一个Runnable或Callable的操作Happens-Before任务开始执行操作
如果两个操作之间不存在上述的Happens-Before规则中的任意一条，并且也不能通过已有的Happens-Before关系推到出来，那么这两个操作之间就没有顺序性的保障，虚拟机可以对这两个操作进行重排序！

重要的事情说三遍：如果存在hb(a,b)，那么操作a在内存上面所做的操作（如赋值操作等）都对操作b可见，即操作a影响了操作b。


# volatile如何保证可见性
## 可见性问题的由来
大家都知道CPU的处理速度非常快，快到内存都无法跟上CPU的速度而且差距非常大，而这个地方不加以处理通常会成为CPU效率的瓶颈，为了消除速度差带来的影响，CPU通常自带了缓存：一级、二级甚至三级缓存（我们可以在电脑描述信息上面看到）。JVM也是出于同样的道理给每个线程分配了工作内存（Woking Memory，注意：不是主内存）。我们要知道线程对变量的修改都会反映到工作内存中，然后JVM找一个合适的时刻将工作内存上的更改同步到主内存中。正是由于线程更改变量到工作内存同步到主内存中存在一个时间差，所以这里会造成数据一致性问题，这就是可见性问题的由来。
## volatile采取的措施
volatile采取的措施其实很好理解：只要被volatile修饰的变量被更改就立即同步到主内存，同时其它线程的工作内存中变量的值失效，使用时必须从主内存中读取。 换句话说，线程的工作内存“不缓存”被volatile修饰的变量。
