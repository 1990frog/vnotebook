# Happens-Before

什么是happens-before（两种接受，一种意思）
happens-before规则是用来解决可见性问题的：在时间上，动作A发生在动作B之前，B保证能看到A，这就是happens-before。
两个操作可以用happens-before于另一个操作，那么我们说第一个操作对应第二个操作是可见的。

什么不是happens-before
两个线程没有相互配合的机制，所以代码x和y的执行结果并不能保证被对方看到的，这就不具备happens-before。

Happens-Before规则有哪些？
1.单线程规则
2.锁操作（synchronized和Lock）
3.volatile变量
4.线程启动
5.线程join
6.传递性
7.中断
8.构造方法
9.工具类的Happens-Before原则
1）线程安全的容器get一定能看到在此之前的put等存入动作
2）CountDownLatch
3）Semaphore
4）Future
5）线程池
6）CyclicBarrier



![20191020213736140_1772167076](_v_images/20191022144012490_853511396.png)

![20191020214138306_387138640](_v_images/20191022144026772_1555625727.png)

传递性：如果hb（A，B）而且bh（B，C），那么可以推断出hb（A，C）
中断：一个线程被其他线程interrupt，那么检查中断（isInterrupted）或者抛出InterruptedException一定能看到
构造方法：对象构造方法的最后一行指令happens-before于finalize()方法的第一行指令

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