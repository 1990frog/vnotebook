[TOC]

# 概览
在用Java编写并发程序时，往往会碰到某个线程因计算量大或因阻塞而一直处于无响应的情况，我们可能会等的不耐烦（也可能是不想让它占用太多资源）想及时终止掉它，那就需要用到任务超时结束的技巧了。在刚接触到多线程时，我本以为API会提供这样一个多线程类：Thread(Runnable r, long timeout) ,第二个参数用来设置超时时间，可事实并非如此。因为这样的类不具有通用性，面向对象设计语言的目标是达到更高级的抽象，所以系统只提供了更广泛的定时类，及其他一些类方法。这就需要我们借助这些工具来达到任务超时结束的目的。话不多说，直入正题。（PS：由于作者水平有限，这些方法只是给大家提供几个思路，可能并不是教科书式的标准案例）

# 方法一：使用Thread.join(long million)
（先讲一下本人对join方法的理解，已理解此方法的可以略过）join方法可以这样理解，在理解它之前，先解释另一个常识，即当前线程（后面称为目标线程，因为它是我们想使其超时结束的目标任务）的创建及start的调用，一定是在另一个线程中进行的（最起码是main线程，也可以是不同于main线程的其他线程），这里我们假设为main线程，并且称之为依赖线程，因为目标线程的创建是在他里面执行的。介绍完这些常识就可以进一步解释了，join的字面意思是，使目标线程加入到依赖线程中去，也可以理解为在依赖线程中等待目标线程一直执行直至结束（如果没有设置超时参数的话）。设置了超时参数（假设为5秒）就会这样执行，在依赖线程中调用了join之后，相当于告诉依赖线程，现在我要插入到你的线程中来，即两个线程合二为一，相当于一个线程（如果不执行插入的话，那目标线程和依赖线程就是并行执行），而且目标线程是插在主线程前面，所以目标线程先执行，但你主线程只需要等我5秒，5秒之后，不管我有没有执行完毕，我们两都分开，这时又会变成两个并行执行的线程，而不是目标线程直接结束执行，这点很重要。

其实这个方法比较牵强，因为它主要作用是用来多个线程之间进行同步的。但因为它提供了这个带参数的方法（所以这也给了我们一个更广泛的思路，就是一般带有超时参数的方法我们都可以尝试着用它来实现超时结束任务），所以我们可以用它来实现。注意这里的参数的单位是固定的毫秒，不同于接下来的带单位的函数。具体用法请看示例：

```java
public class JoinTest implements Runnable {

    public String name;
    private int time;

    public JoinTest(String s, int t) {
        name = s;
        time = t;
    }

public class JoinTest implements Runnable {

    public String name;
    private int time;

    public JoinTest(String s, int t) {
        name = s;
        time = t;
    }

    @Override
    public void run() {
        for (int i = 0; i < time; ++i) {
            System.out.println("task " + name + " " + (i + 1) + " round");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.println(name + "is interrupted when calculating, will stop...");
                return; // 注意这里如果不return的话，线程还会继续执行，所以任务超时后在这里处理结果然后返回
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        JoinTest task1 = new JoinTest("one", 4);
        JoinTest task2 = new JoinTest("two", 2);
        Thread t1 = new Thread(task1);
        Thread t2 = new Thread(task2);

        t1.start();
        t1.join(2000); // 在主线程中等待t1执行2秒
        t1.interrupt(); // 这里很重要，一定要打断t1,因为它已经执行了2秒。

        t2.start();
        t2.join(1000);
        t2.interrupt();

    }
}    @Override
    public void run() {
        for (int i = 0; i < time; ++i) {
            System.out.println("task " + name + " " + (i + 1) + " round");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.println(name + "is interrupted when calculating, will stop...");
                return; // e.printStackTrace(); 你好骚啊！
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        JoinTest task1 = new JoinTest("one", 4);
        JoinTest task2 = new JoinTest("two", 2);
        Thread t1 = new Thread(task1);
        Thread t2 = new Thread(task2);

        t1.start();
        t1.join(2000); // 在主线程中等待t1执行2秒
        t1.interrupt(); // 这里很重要，一定要打断t1,因为它已经执行了2秒。

        t2.start();
        t2.join(1000);
        t2.interrupt();

    }
}
```

在主线程中等待t1执行2秒之后，要interrupt（而不是直接调用stop，这个方法已经被弃用）掉它，然后在t1里面会产出一个中断异常，在异常里面处理完该处理的事，就要return，一定要return，如果不return的话，t1还会继续执行，只不过是与主线程并行执行。<font color="red">学到了，好骚的写法啊！</font>

# 方法二：Future.get(long million, TimeUnit unit) 配合Future.cancle(true)

Future系列（它的子类）的都可以实现，这里采用最简单的Future接口实现。

```java
public class FutureTest implements Callable<Boolean> {

    private int time;

    public FutureTest(int time) {
        this.time = time;
    }

    @Override
    public Boolean call() throws Exception {
        for (int i = 0; i < time; ++i) {
            System.out.println(Thread.currentThread().getName() + " round " + (i + 1));
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.println(Thread.currentThread().getName() + " is interrupted when calculating, will stop...");
                return false; // 注意这里如果不return的话，线程还会继续执行，所以任务超时后在这里处理结果然后返回
            }
        }
        return true;
    }

    public static void main(String[] args) {

        ExecutorService executor = Executors.newCachedThreadPool();
        FutureTest task1 = new FutureTest(10);
        Future<Boolean> f1 = executor.submit(task1);

        try {
            if (f1.get(2, TimeUnit.SECONDS)) { // future将在2秒之后取结果
                System.out.println("one complete successfully");
            }
        } catch (InterruptedException e) {
            System.out.println("future在睡着时被打断");
            executor.shutdownNow();
        } catch (ExecutionException e) {
            System.out.println("future在尝试取得任务结果时出错");
            executor.shutdownNow();
        } catch (TimeoutException e) {
            System.out.println("future时间超时");
            f1.cancel(true);
            // executor.shutdownNow();
            // executor.shutdown();
        } finally {
            executor.shutdownNow();
        }
    }
}
```

运行结果如下，task在2秒之后停止：

```
task one round 1
task one round 2
futrue时间超时
one is intertupted when callculationg,will stop...
```

如果把Task中捕获InterruptedException的catch块中的return注释掉，就是这样的结果：

```
task one round 1
task one round 2
futrue时间超时
one is intertupted when callculationg,will stop...
task one round 3
task one round 4
task one round 5
```

task继续执行，直至结束

# 方法三：ExecutorService.awaitTermination(long million, TimeUnit unit)
这个方法会一直等待所有的任务都结束，或者超时时间到立即返回，若所有任务都完成则返回true，否则返回false

```java
public class AwaitTermination implements Runnable {

    private int time;

    public AwaitTermination(int time) {
        this.time = time;
    }

    public void run() {
        for (int i = 0; i < time; ++i) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.println(Thread.currentThread().getName() + " is interrupted when calculating, will stop...");
                return; // 注意这里如果不return的话，线程还会继续执行，所以任务超时后在这里处理结果然后返回
            }
            System.out.println(Thread.currentThread().getName()  + " " + (i + 1) + " round");
        }
        System.out.println(Thread.currentThread().getName() + " finished successfully");
    }

    public static void main(String[] args) {

        ExecutorService executor = Executors.newCachedThreadPool();
        AwaitTermination task1 = new AwaitTermination(5);
        AwaitTermination task2 = new AwaitTermination(2);
        Future<?> future = executor.submit(task1);
        Future<?> future2 = executor.submit(task2);

        List<Future<?>> futures = new ArrayList<>();
        futures.add(future);
        futures.add(future2);

        try {
            if (executor.awaitTermination(3, TimeUnit.SECONDS)) {
                System.out.println("task finished");
            } else {
                System.out.println("task time out,will terminate");
                for (Future<?> f : futures) {
                    if (!f.isDone()) {
                        f.cancel(true);
                    }
                }
            }
        } catch (InterruptedException e) {
            System.out.println("executor is interrupted");
        } finally {
            executor.shutdown();
        }
    }
}
```

# 方法四：设置一个守护线程，守护线程先sleep一段定时时间，睡醒后打断它所监视的线程
一开始准备在守护任务里面用一个集合来实现监视多个任务，接着发现要实现这个功能还得在这个守护任务里面为每一个监视的任务开启一个监视任务，一时又想不到更好的方法来解决，索性只监视一个算了，留待以后改进吧。

