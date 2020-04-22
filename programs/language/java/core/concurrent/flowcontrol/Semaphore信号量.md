[TOC]

Semaphore是基于AQS的

+ Semaphore可以用来限制或管理数量有限的资源的使用情况
+ 信号量的作用是维护一个“许可证”的计数，线程可以“获取”许可证，那信号量剩余的许可证就减一，线程也可以“释放”一个许可证，那信号量剩余的许可证就加一，当信号量所拥有的许可证数量为0，那么下一个还想要获取许可证的线程，就需要等待，直到有另外的线程释放了许可证

# 信号量使用流程
+ 初始化Semaphore并指定许可证的数量
+ 在需要被现在的代码前加acquire()或者acquireUninterruptibly()方法
+ 在任务执行结束后，调用release()来释放许可证

# 信号量主要方法介绍
+ new Semaphore(int permits,boolean fair)：这里可以设置是否要使用公平策略
+ acquire()响应中断
+ acquireUninterruptibly()不响应中断
+ tryAcquire()：看看现在有没有空闲的许可证，如果有的话就获取，如果没有的话也没关系，我不必陷入阻塞，我可以去做别的事，过一会再来查看许可证的空闲情况
+ tryAcquire(timeout)：和tryAcquire()一样，但是多了一个超时时间，比如“在3秒内获取不到许可证，我就去做别的事”

# 常用方法
## new Semaphore(int permits,boolean fair) 构造方法
参数：
+ permits：信号量
+ fair：true公平策略，如果传入true，那么Semaphore会把之前等待的线程放到FIFO的队列里，以便于当有了新的许可证，可以分发给之前等了最长时间的线程
## acquire 请求信号量，响应中断（阻塞）
多态：
+ acquire()请求一个
+ acquire(int)请求多个
## acquireUninterruptibly 请求信号量，不响应中断（阻塞）
+ acquireUninterruptibly()
+ acquireUninterruptibly(int)
## tryAcquire 尝试获取
看看现在有没有空闲的许可证，如果有的话就获取，如果没有的话也没关系，我不必陷入阻塞，我可以去做别的事，过一会再来查看许可证的空闲情况
多态：
+ tryAcquire()
+ tryAcquire(long,TimeUnit)
+ tryAcquire(int)
+ tryAcquire(int,long,TimeUnit)
## release 释放信号量
多态：
+ release()
+ release(int)

# 一次性获取或释放多个许可证
+ 比如TaskA会调用很消耗资源的method1()，而TaskB调用的是不太消耗资源的method2()，假设我们一共有5个许可证。那么我们就可以要求TaskA获取5个许可证才能执行，而TaskB只需要获取到一个许可证就能执行，这样就避免了A和B同时运行的情况，我们可以根据自己的需求合理分配资源
+ 获取和释放的许可证数量必须一致，否则比如每次都获取2个但是只释放1个甚至不释放，随着时间的推移，到最后许可证数量不够用，会导致程序卡死。（虽然信号量类并不对释放和获取的数量做规定，但是这是我们的编程规范，否则容易出错）
+ 注意在初始化Semaphore的时候设置公平性，一般设置为true会更合理
+ 并不是必须由获取许可证的线程释放那个许可证，事实上，获取和释放许可证对线程并无要求，也许是A获取了，然后由B释放，只要逻辑合理即可（跨线程、跨线程池，如果将信号量设置为静态的，那就可以全局访问）
+ 信号量的作用，除了控制临界区最多同时有N个线程访问外，另一个作用是可以实现“条件等待”，例如线程1需要在线程2完成准备工作后才能开始工作，那么就线程1调用acquire()，而线程2完成任务后release()，这样的话，相当于是轻量级的CountDownLatch

# demo
```java
public class SemaphoreDemo implements Runnable {

    static Semaphore semaphore = new Semaphore(1,true);
//    static AtomicInteger counter = new AtomicInteger(0);
    int counter = 0;

    @SneakyThrows
    @Override
    public void run() {
        for(;;){
            semaphore.acquire();
            TimeUnit.SECONDS.sleep(1);
            System.out.println(counter++);
            semaphore.release();
        }
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(100);
        SemaphoreDemo runner = new SemaphoreDemo();
        for (int i = 0; i < 10; i++) {
            executorService.submit(runner);
        }
    }
}
```