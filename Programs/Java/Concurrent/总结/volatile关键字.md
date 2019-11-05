# volatile关键字

volatile是什么？
volatile是一种同步机制，比synchronized或者Lock相关类更轻量，因为使用volatile并不会发生上下文切换等开销很大的行为。
如果一个变量被修饰成volatile，那么JVM就知道了这个变量可能会被并发修改（避免重排序）。
但是开销小，相应的能力也小，虽然说volatile是用来同步的保证线程安全的，但是volatile做不到synchronized那样的原子保护，volatile仅在很有限的场景下才能发挥作用。（i++异常）

volatile的适用场合？
不适用：a++
适用场合1：boolean flag，如果一个共享变量至始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全。
适用场合2：作为刷新之前变量的触发器
volatile之前的操作具体范围？
跟它有关，跟它无关？
volatile之前的所有操作赋值都会被看到

![20191020223915327_1626747371](_v_images/20191022144102912_700453581.png)

volatile的作用：可见性、禁止重排序
可见性：读一个volatile变量之前，需要先使相应的本地缓存失效，这样就必须到主内存读取最新值，写一个volatile属性会立即刷入到主内存。
禁止指令重排序优化：解决单例双重锁乱序问题

volatile和synchronized的关系？
volatile在这方面可以看做是轻量版的synchronized：如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全。
a++，为什么不行呢：因为a++要根据a原来的值做操作，而非完全赋值（有读取操作）

学以致用：用volatile修正重排序问题

volatile小结
1.volatile修饰符适用于以下场景：某个属性被多个线程共享，其中有一个线程修改了此属性，其他线程可以立即得到修改后的值，比如boolean flag；或者作为触发器，实现轻量级同步。
2.volatile属性的读写都是无锁的，它不能替代synchronized，因为它没有提供原子性和互斥性。因为无锁，不需要花费时间在获取锁和释放锁上，所以说它是低成本的。
3.volatile只能作用于属性，我们用volatile修饰属性，这样compilers就不会对这个属性做命令重排序。
4.volatile提供了可见性，任何一个线程对其的修改将立马对其他线程可见。volatile属性不会被线程缓存，始终从主存中读取。
5.volatile提供了happens-before保证，对volatile变量v的写入happens-before所有其他线程后续对v的读操作。
6.volatile可以使得long和double的赋值是原子的。

# 能保证可见性的措施

除了volatile可以让变量保证可见性外，synchronized、Lock、并发集合、Thread.join()和Thread.start()等都可以保证可见性
具体看happens-before原则的规定

# 升华：对synchronized可见性的正确理解

synchronized不仅保证了原子性，还保证了可见性
synchronized不仅让被保护的代码安全，还近朱者赤

# 可见性脉络总结

1.什么是可见性
2.为什么会有可见性问题
3.jmm的抽象：主内存和本地内存
4.Happens-Before原则
5.volatile关键字
6.能保证可见性的措施
7.升华：对synchronized可见性的正确理解

# volatile局限性
volatile最大的问题就是无法应用于拥塞方法。假设在循环中调用了拥塞方法，任务可能因拥塞而永远不会去检查取消标志位，甚至会造成永远不能停止。

```java
public class PrimeGenerator implements Runnable {
    private static ExecutorService exec = Executors.newCachedThreadPool();

    private final List<BigInteger> primes
            = new ArrayList<BigInteger>();
    //取消标志位
    private volatile boolean cancelled;

    public void run() {
        BigInteger p = BigInteger.ONE;
        //每次在生成下一个素数时坚持是否取消
        //如果取消，则退出
        while (!cancelled) {
            p = p.nextProbablePrime();
            synchronized (this) {
                primes.add(p);
            }
        }
    }

    public void cancel() {
        cancelled = true;
    }

    public synchronized List<BigInteger> get() {
        return new ArrayList<BigInteger>(primes);
    }

    static List<BigInteger> aSecondOfPrimes() throws InterruptedException {
        PrimeGenerator generator = new PrimeGenerator();
        exec.execute(generator);
        try {
            SECONDS.sleep(1);
        } finally {
            generator.cancel();
        }
        return generator.get();
    }
}

```