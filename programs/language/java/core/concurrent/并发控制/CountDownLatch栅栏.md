[TOC]

# 概览
倒数门闩
例子：购物拼团；大巴人满发车
流程：倒数结束之前，一直处于等待状态，直到倒计时结束了，此线程才继续工作

# 主要方法
## CountDownLatch(int count)
仅有一个构造函数，参数count为需要倒数的数值

## await()
调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行

## countDown()
将count值减1，直到为0时，等待的线程会被唤起

# 使用场景一：一个线程等待多个线程都执行完毕，再继续自己的工作
场景：体检

# 使用场景二：多个线程等待某一个线程的信号，同时开始执行（JMETER）
服务器压测：让所有的并发同一时刻发生，模拟高峰流量
```java
public class CountDownLatchDemo implements Runnable {

    private static CountDownLatch countDownLatch = new CountDownLatch(6);

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        CountDownLatchDemo runner = new CountDownLatchDemo();
        for (int i = 0; i < 5; i++) {
            executorService.submit(runner);
            countDownLatch.countDown();
        }
        executorService.shutdown();
        TimeUnit.SECONDS.sleep(3);
        countDownLatch.countDown();
        while (executorService.isTerminated()) {
            System.out.println("end!");
        }
    }

    @SneakyThrows
    @Override
    public void run() {
        countDownLatch.await();
        System.out.println("run");
    }

}
嵌套使用：
```java
public class CountDownLatchDemo1And2 {

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch begin = new CountDownLatch(1);

        CountDownLatch end = new CountDownLatch(5);
        ExecutorService service = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            final int no = i + 1;
            Runnable runnable = () -> {
                System.out.println("No." + no + "准备完毕，等待发令枪");
                try {
                    begin.await();
                    System.out.println("No." + no + "开始跑步了");
                    Thread.sleep((long) (Math.random() * 10000));
                    System.out.println("No." + no + "跑到终点了");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    end.countDown();
                }
            };
            service.submit(runnable);
        }
        //裁判员检查发令枪...
        Thread.sleep(5000);
        System.out.println("发令枪响，比赛开始！");
        begin.countDown();

        end.await();
        System.out.println("所有人到达终点，比赛结束");
    }
}
```
# 缺点
CountDownLatch是不能够重用的，如果需要重新计数，可以考虑使用CyclicBarrier或者创建新的CountDownLatch实例

# 总结
+ 两个典型用法：一等多和多等一
+ CountDownLatch类在创建实例的时候，需要传递倒数次数。倒数到0的时候，之前等待的线程会继续执行。
+ 创建时传递倒数次数（count构造函数），谁想等待谁就调用await()，谁去倒数谁就调用countDown()，这三个方法结合起来之后，直到countDown到0之后，之前的await的方法都会触发
+ CountDownLatch不能回滚重置（复用）