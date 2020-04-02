[TOC]

<font color="red">wait(),notify(),notifyAll()必须要在Synchronized关键中使用</font>

# wait()
当前调用synchronized修饰代码的线程进入阻塞状态（wait）,立刻让出锁
```java
public class WaitDemo implements Runnable {


    @Override
    public void run() {
        play1();
    }

    public synchronized void play1() {
        System.out.println(Thread.currentThread().getName());
        try {
            if("thread1".equals(Thread.currentThread().getName())){
//                wait();
            }else {
                notify();
                System.out.println("notify ....");
            }
//            TimeUnit.SECONDS.sleep(10);
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName() + " is stop");
    }


    public static void main(String[] args) throws InterruptedException {
        WaitDemo demo = new WaitDemo();
        Thread thread1 = new Thread(demo);
        thread1.setName("thread1");
        Thread thread2 = new Thread(demo);
        thread2.setName("thread2");
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("all stop");
    }
}

```
# notify()/notifyAll()
会在执行完毕后续可执行的代码之后让出锁唤醒
```java
public class WaitDemo implements Runnable {


    @Override
    public void run() {
        play1();
    }

    public synchronized void play1() {
        System.out.println(Thread.currentThread().getName());
        try {
            if("thread1".equals(Thread.currentThread().getName())){
                wait();
            }else {
                notify();
                System.out.println("notify ....");
            }
            TimeUnit.SECONDS.sleep(10);
//            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName() + " is stop");
    }


    public static void main(String[] args) throws InterruptedException {
        WaitDemo demo = new WaitDemo();
        Thread thread1 = new Thread(demo);
        thread1.setName("thread1");
        Thread thread2 = new Thread(demo);
        thread2.setName("thread2");
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("all stop");
    }
}
```

# 重点
每个对象都可以被认为是一个"监视器monitor"，这个监视器由三部分组成（一个独占锁，一个入口队列，一个等待队列）。

注意是一个对象只能有一个独占锁synchronized，但是任意线程线程都可以拥有这个独占锁。

对于对象的非同步方法而言，任意时刻可以有任意个线程调用该方法。（即普通方法同一时刻可以有多个线程调用）

对于对象的同步方法而言，只有拥有这个对象的独占锁才能调用这个同步方法。如果这个独占锁被其他线程占用，那么另外一个调用该同步方法的线程就会处于阻塞状态，此线程进入入口队列。

若一个拥有该独占锁的线程调用该对象同步方法的wait()方法，则该线程会释放独占锁，并加入对象的等待队列；（为什么使用wait()？希望某个变量被设置之后再执行，notify()通知变量已经被设置。）

某个线程调用notify(),notifyAll()方法是将等待队列的线程转移到入口队列，然后让他们竞争锁，所以这个调用线程本身必须拥有锁。

调用`wait()`就是释放锁，释放锁的前提是必须要先获得锁，先获得锁才能释放锁
调用`notify()`是将锁交给含有`wait()`方法的线程，让其继续执行下去，如果自身没有锁，怎么叫把锁交给其他线程呢；（本质是让处于入口队列的线程竞争锁）

# Block阻塞阶段
直到以下4种情况之一发生时，才会被唤醒
1.另一个线程调用这个对象的notify()方法且刚好被唤醒的是本线程；
2.另一个线程调用这个对象的notifyAll()方法；
3.过了wait(long timeout)规定的超时时间，如果传入0就是永久等待；
4.线程自身调用了interrupt()