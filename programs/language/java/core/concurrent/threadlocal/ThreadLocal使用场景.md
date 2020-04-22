[TOC]

# 两大使用场景
+ 每个线程需要一个独享的对象（典型需要使用的类有SimpleDateFormat和Random）
+ 每个线程内需要保存全局变量（例如在拦截器中获取用户信息），可以让不同方法直接使用，避免参数传递的麻烦（例子如Spring当前线程参数）

# 场景1：每个线程需要一个独享的对象
每个Thread内有<font color="red">自己</font>的实例副本，<font color="red">不共享</font>
比喻：<font color="red">教材</font>只有一本，一起做笔记有线程安全问题。<font color="red">复印</font>后没问题

需求：打印1000个时间戳，最好不重复
## 方法一，每次都生成一个新的SimpleDateFormat对象，浪费了大量资源
```java
public class ThreadLocalNormalUsage02 {

    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            threadPool.submit(() -> {
                String date = new ThreadLocalNormalUsage02().date(finalI);
                System.out.println(date);
            });
        }
        threadPool.shutdown();
    }

    public String date(int seconds) {
        //参数的单位是毫秒，从1970.1.1 00:00:00 GMT计时
        Date date = new Date(1000 * seconds);
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        return dateFormat.format(date);
    }
}
```
## 方法二，使用线程池来输出日期，创建了10个线程，由于SimpleDateFormat是线程不安全的，无法得到正确的结果
```java
public class ThreadLocalNormalUsage03 {

    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);
    static SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            threadPool.submit(() -> {
                String date = new ThreadLocalNormalUsage03().date(finalI);
                System.out.println(date);
            });
        }
        threadPool.shutdown();
    }

    public String date(int seconds) {
        //参数的单位是毫秒，从1970.1.1 00:00:00 GMT计时
        Date date = new Date(1000 * seconds);
        return dateFormat.format(date);
    }
}
```
## 方法三：使用Synchronized同步锁，每次只有一个线程可以访问SimpleDateFormat对象，性能不足
```java
public class ThreadLocalNormalUsage04 {

    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);
    static SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            threadPool.submit(() -> {
                String date = new ThreadLocalNormalUsage04().date(finalI);
                System.out.println(date);
            });
        }
        threadPool.shutdown();
    }

    public String date(int seconds) {
        //参数的单位是毫秒，从1970.1.1 00:00:00 GMT计时
        Date date = new Date(1000 * seconds);
        String s = null;
        synchronized (ThreadLocalNormalUsage04.class) {
            s = dateFormat.format(date);
        }
        return s;
    }
}
```
## 方法四：使用ThreadLocal，每个线程持有一个SimpleDateFomat
```java
public class ThreadLocalNormalUsage05 {

    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            threadPool.submit(() -> {
                String date = new ThreadLocalNormalUsage05().date(finalI);
                System.out.println(date);
            });
        }
        threadPool.shutdown();
    }

    public String date(int seconds) {
        //参数的单位是毫秒，从1970.1.1 00:00:00 GMT计时
        Date date = new Date(1000 * seconds);
//        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        SimpleDateFormat dateFormat = ThreadSafeFormatter.dateFormatThreadLocal2.get();
        return dateFormat.format(date);
    }
}

class ThreadSafeFormatter {

    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };

    /**
     * java8之后的lumbda表达式，和上面等效
     */
    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal2 = ThreadLocal
            .withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
}
```
## SimpleDateFormat的进化之路
1. 2个线程分别用自己的SimpleDateFormat，这没问题【创建两个对象消耗不大】
2. 后来延伸出10个，那就有10个线程和10个SimpleDateFormat，这虽然写法不优雅（应该复用对象），但勉强可以接受【10个也就认了】
3. 但是当需求变成了1000个，那么必然要用线程池（否则消耗内存太多）【1000个疯了】
4. 所有的线程共有同一个SimpleDateFormat对象【线程安全】
5. 这是线程不安全的，出现了并发安全问题
6. 我们可以选择加锁，加锁结果正常，但是效率低【线性执行】
7. 在这里更好的解决方案是使用ThreadLocal【每个线程持有一个】
8. lambda表达式【初始化方式】

# 场景二：当前用户信息需要被线程内所有方法共享
+ 场景：一个比较繁琐的解决方案是把user作为参数层层传递，从service1()传递到service2()，再从service2()传递到service3()，以此类推，但是这样做会导致代码冗余且不移维护
+ 目标：每个线程内需要保存全局变量，可以让不同方法直接使用，避免参数传递的麻烦
+ 方法：用ThreadLocal保存一些业务内容（用户权限信息、从用户系统获取到的用户名、userId等）
+ 总结：这些信息在同一个线程内相同，但是不同的线程使用的业务内容是不相同的
+ 缺陷：持有时间过长，或者没有remove可能会造成内存溢出

## 方法一：可以使用UserMap
当多线程同时工作时，我们需要保证线程安全，可以用synchronized，也可以用ConcurrentHashMap，但无论用什么，都会对性能有所影响

## 方法二：ThreadLocal
更好的办法是使用ThreadLocal，这样无需synchronized，可以在不影响性能的情况下，也无需层层传递参数，就可达到保存当前线程对应的用户信息的目的：
1. 用ThreadLocal保存一些业务内容（用户权限信息、从用户系统获取到的用户名、userID等）
2. 这些信息在同一个线程内相同，但是不同的线程使用的业务内容是不相同的
3. 在线程生命周期内，都通过这个静态ThreadLocal实例的get()方法取得自己set过的那个对象，避免了将这个对象（例如user对象）作为参数传递的麻烦
4. 强调的是同一个请求内（同一个线程内）不同方法间的共享
5. 不需要重写initialValue()方法，但是必须手动调用set()方法

```java
public class ThreadLocalNormalUsage06 {

    public static void main(String[] args) {
        new Service1().process("");

    }
}

class Service1 {

    public void process(String name) {
        User user = new User("超哥");
        UserContextHolder.holder.set(user);
        new Service2().process();
    }
}

class Service2 {

    public void process() {
        User user = UserContextHolder.holder.get();
        ThreadSafeFormatter.dateFormatThreadLocal.get();
        System.out.println("Service2拿到用户名：" + user.name);
        new Service3().process();
    }
}

class Service3 {

    public void process() {
        User user = UserContextHolder.holder.get();
        System.out.println("Service3拿到用户名：" + user.name);
        UserContextHolder.holder.remove();
    }
}

class UserContextHolder {

    public static ThreadLocal<User> holder = new ThreadLocal<>();


}

class User {

    String name;

    public User(String name) {
        this.name = name;
    }
}
```
