[TOC]

![](_v_images/20200106170347430_1504130535.png)
# 概览
## 什么是Threadlocal
ThreadLocal类顾名思义可以理解为线程本地变量。也就是说如果定义了一个ThreadLocal，每个线程往这个ThreadLocal中读写是线程隔离，互相之间不会影响的。它提供了一种将可变数据通过每个线程有自己的独立副本从而实现线程封闭的机制。
## 实现思路
Thread类有一个类型为ThreadLocal.ThreadLocalMap的实例变量threadLocals，也就是说每个线程有一个自己的ThreadLocalMap。
ThreadLocalMap有自己的独立实现，可以简单地将它的ThreadLocal视作key，value为代码中放入的值（实际上key并不是ThreadLocal本身，而是它的一个弱引用）。
每个线程在往某个ThreadLocal里塞值的时候，都会往自己的ThreadLocalMap里存，读也是以某个ThreadLocal作为引用，在自己的map里找对应的key，从而实现了线程隔离。
## 使用ThreadLocal带来的好处
1. 达到线程安全【单线程】
2. 不需要加锁，提高执行效率【不需要排队】
3. 更高效地利用内存、节省开销：相比于每个任务都新建一个SimpleDateFormat，显然用ThreadLocal可以节省内存和开销【单线程单例】
4. 免去传参的繁琐：无论是场景一的工具类，还是场景二的用户名，都可以在任何地方直接通过ThreadLocal拿到，再也不需要每次都传同样的参数。ThreadLocal使得代码耦合度低，更优雅【打破栈封闭，每个方法不要传值传来传去了】
## 备注（java四种引用类型）
1. 强引用：如果JVM垃圾回收器GC Roots可达性分析结果为可达，表示引用类型仍然被引用着，这类对象始终不会被垃圾回收器回收，即使JVM发生OOM也不会回收。而如果GC Roots的可达性分析结果为不可达，那么在GC时会被回收
2. 软引用：在JVM内存充足的情况下，软引用并不会被垃圾回收器回收，只有在JVM内存不足的情况下，才会被垃圾回收器回收
3. 弱引用：弱引用是一种比软引用生命周期更短的引用。它的生命周期很短，不论当前内存是否充足，都只能存活到下一次垃圾收集之前
4. 虚引用：随时可被回收
# 两大使用场景
+ 每个线程需要一个独享的对象（典型需要使用的类有SimpleDateFormat和Random）
+ 每个线程内需要保存全局变量（例如在拦截器中获取用户信息），可以让不同方法直接使用，避免参数传递的麻烦（例子如Spring当前线程参数）
## 场景1：每个线程需要一个独享的对象
每个Thread内有<font color="red">自己</font>的实例副本，<font color="red">不共享</font>
比喻：<font color="red">教材</font>只有一本，一起做笔记有线程安全问题。<font color="red">复印</font>后没问题

需求：打印1000个时间戳，最好不重复
### 方法一，每次都生成一个新的SimpleDateFormat对象，浪费了大量资源
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
### 方法二，使用线程池来输出日期，创建了10个线程，由于SimpleDateFormat是线程不安全的，无法得到正确的结果
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
### 方法三：使用Synchronized同步锁，每次只有一个线程可以访问SimpleDateFormat对象，性能不足
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
### 方法四：使用ThreadLocal，每个线程持有一个SimpleDateFomat
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
### SimpleDateFormat的进化之路
1. 2个线程分别用自己的SimpleDateFormat，这没问题【创建两个对象消耗不大】
2. 后来延伸出10个，那就有10个线程和10个SimpleDateFormat，这虽然写法不优雅（应该复用对象），但勉强可以接受【10个也就认了】
3. 但是当需求变成了1000个，那么必然要用线程池（否则消耗内存太多）【1000个疯了】
4. 所有的线程共有同一个SimpleDateFormat对象【线程安全】
5. 这是线程不安全的，出现了并发安全问题
6. 我们可以选择加锁，加锁结果正常，但是效率低【线性执行】
7. 在这里更好的解决方案是使用ThreadLocal【每个线程持有一个】
8. lambda表达式【初始化方式】
## 场景二：当前用户信息需要被线程内所有方法共享
+ 场景：一个比较繁琐的解决方案是把user作为参数层层传递，从service1()传递到service2()，再从service2()传递到service3()，以此类推，但是这样做会导致代码冗余且不移维护
+ 目标：每个线程内需要保存全局变量，可以让不同方法直接使用，避免参数传递的麻烦
+ 方法：用ThreadLocal保存一些业务内容（用户权限信息、从用户系统获取到的用户名、userId等）
+ 总结：这些信息在同一个线程内相同，但是不同的线程使用的业务内容是不相同的
+ 缺陷：持有时间过长，或者没有remove可能会造成内存溢出
### 方法一：可以使用UserMap
![未命名文件 (1)](_v_images/20200106162837107_1605118533.png)
当多线程同时工作时，我们需要保证线程安全，可以用synchronized，也可以用ConcurrentHashMap，但无论用什么，都会对性能有所影响
### 方法二：ThreadLocal
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

# ThreadLocal的重要方法
## T initialValue()：初始化
1. 该方法会返回当前线程对应的“初始值”，这是一个延迟加载的方法，只有在调用get的时候，才会触发
2. 当线程第一次使用get方法访问变量时，将调用次方法，除非线程当前调用了set方法，在这种情况下，不会为线程调用initialValue方法
3. 通常，每个线程最多调用一次此方法，但如果已经调用了remove()后，在调用get()，则可以再次调用此方法
4. <font color="red">如果不重写本方法，这个方法会返回null。一般使用匿名内部类的方法来重写initialize()方法，以便在后续使用者可以初始化副本对象</font>
## withInitial(Supplier<? extends S> supplier)
```java
static final class SuppliedThreadLocal<T> extends ThreadLocal<T> {

    private final Supplier<? extends T> supplier;

    SuppliedThreadLocal(Supplier<? extends T> supplier) {
        this.supplier = Objects.requireNonNull(supplier);
    }

    @Override
    protected T initialValue() {
        return supplier.get();
    }
}
```
## void set(T t)：为这个线程设置一个新值

```java
/**
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of
 *        this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

## T get()：得到这个线程对应的value。如果是首次调用get()，则会调用initialize来得到这个值
+ get方法是先取出当前线程的ThreadLocalMap，然后调用map.getEntry方法，把本ThreadLocal的引用作为参数传入，取出map中属于本ThreadLocal的value
+ 注意，这个map以及map中的key和value都是保存在线程中的，而不是保存在ThreadLocal中，Thread持有ThreadLocalMap，ThreadLocalMap持有ThreadLocal

```java
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

/**
 * Variant of set() to establish initialValue. Used instead
 * of set() in case user has overridden the set() method.
 *
 * @return the initial value
 */
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```
set()/initialValue()并不是直接将
## void remove()：删除对应这个线程的值
```java
/**
     * Removes the current thread's value for this thread-local
     * variable.  If this thread-local variable is subsequently
     * {@linkplain #get read} by the current thread, its value will be
     * reinitialized by invoking its {@link #initialValue} method,
     * unless its value is {@linkplain #set set} by the current thread
     * in the interim.  This may result in multiple invocations of the
     * {@code initialValue} method in the current thread.
     *
     * @since 1.5
     */
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```
## 根据共享对象的生成时机不同，选择initialValue或set来保存对象
+ initialValue：在ThreadLocal第一次get的时候把对象给初始化出来，对象的初始化时机可以由我们控制【懒加载】
+ set：如果需要保存到ThreadLocal里的对象的生成时机不由我们随意控制，例如拦截器生成的用户信息，用ThreadLocal.set直接放到我们的ThreadLocal中去，以便后续使用
+ 通过源码分析可以看出，setInitialValue和自己set最后都是利用map.set()方法来设置值
+ 也就是说，最后都会对应到ThreadLocalMap的一个Entity，只不过是起点和入口不一样

# Thread、ThreadLocal以及ThreadLocalMap三者之间的关系
## ThreadLocalMap类
```java
static class ThreadMap{
    Entry[] table

    class Entry{}//容器
    int getEntry(ThreadLocal<?>)
    set(ThreadLocal<?>)
    remove(ThreadLocal<?>)
    ......
}
```
+ ThreadLocalMap类，也就是Thread.threadLocals
+ ThreadLocalMap类是每个线程Thread类里面的变量，里面最重要的是一个键值对数组Entry[] table，可以认为是一个map键值对
+ ThreadLocalMap这里采用的是线性探测法，也就是如果发生冲突，就继续找下一个空位置，而不是链表拉链（切换红黑树）

![7432604-ad2ff581127ba8cc](_v_images/20200107101305050_152078636.jpg)

## 线程获取ThreadLocal的过程
1. 获取当前线程
2. 获取当前线程的ThreadLocalMap
3. map不为null，通过ThreadLocal的hash去获取对应的对象
4. 如果对象不为空，返回对象
5. 如果map为null或者ThreadLocal为null，返回ThreadLocal的初始值

ThreadLocalMap类
```java
private Entry[] table;
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();//获取当前线程
    ThreadLocalMap map = getMap(t);//获取当前线程的ThreadLocalMap对象
    if (map != null) {//如果为null，初始化（懒汉式）
        ThreadLocalMap.Entry e = map.getEntry(this);//在ThreadLocalMap中通过ThreadLocal的hash去获取对象
        if (e != null) {//存在就返回对象
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();//不存在就返回初始值
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    //private T referent;         /* Treated specially by GC Reference类持有*/

    Entry(ThreadLocal<?> k, Object v) {
        super(k);//WeakReference(T referent)
        value = v;
    }
}

/**
 * Get the entry associated with key.  This method
 * itself handles only the fast path: a direct hit of existing
 * key. It otherwise relays to getEntryAfterMiss.  This is
 * designed to maximize performance for direct hits, in part
 * by making this method readily inlinable.
 *
 * @param  key the thread local object
 * @return the entry associated with key, or null if no such
 */
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

/**
 * Set the value associated with key.
 *
 * @param key the thread local object
 * @param value the value to be set
 */
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```
# ThreadLocal常见问题
## 内存泄露：某个对象不再有用，但是占用的内存却不能被回收
![7432604-072ea1eed5e63601](_v_images/20200130140258339_358079252.jpg)

ThreadLocal的原理：每个Thread内部维护着一个ThreadLocalMap，它是一个Map。这个映射表的Key是一个弱引用，其实就是ThreadLocal本身，Value是真正存的线程变量Object。
也就是说ThreadLocal本身并不真正存储线程的变量值，它只是一个工具，用来维护Thread内部的Map，帮助存和取。注意上图的虚线，它代表一个弱引用类型，而弱引用的生命周期只能存活到下次GC前。

### ThreadLocal为什么会内存泄漏
ThreadLocal在ThreadLocalMap中是以一个弱引用身份被Entry中的Key引用的，因此如果ThreadLocal没有外部强引用来引用它，那么ThreadLocal会在下次JVM垃圾收集时被回收。
这个时候就会出现Entry中Key已经被回收，出现一个null Key的情况，外部读取ThreadLocalMap中的元素是无法通过null Key来找到Value的。
因此如果当前线程的生命周期很长，一直存在，那么其内部的ThreadLocalMap对象也一直生存下来，这些null key就存在一条强引用链的关系一直存在：Thread --> ThreadLocalMap-->Entry-->Value，这条强引用链会导致Entry不会回收，Value也不会回收，但Entry中的Key却已经被回收的情况，造成内存泄漏。

<font color="red">但是JVM团队已经考虑到这样的情况，并做了一些措施来保证ThreadLocal尽量不会内存泄漏：在ThreadLocal的get()、set()、remove()方法调用的时候会清除掉线程ThreadLocalMap中所有Entry中Key为null的Value，并将整个Entry设置为null，利于下次内存回收。</font>

将Entry的Key设置成弱引用，在配合线程池使用的情况下可能会有内存泄露的风险。之设计成弱引用的目的是为了更好地对ThreadLocal进行回收，当我们在代码中将ThreadLocal的强引用置为null后，这时候Entry中的ThreadLocal理应被回收了，但是如果Entry的key被设置成强引用则该ThreadLocal就不能被回收，这就是将其设置成弱引用的目的。

WeakReference对象的特性就是代码运行一段时间，如果碰到gc，WeakReference指向的对象会被gc回收。

WeakReference的一个特点是它何时被回收是不可确定的, 因为这是由GC运行的不确定性所确定的. 所以, 一般用weak reference引用的对象是有价值被cache, 而且很容易被重新被构建, 且很消耗内存的对象.

### ThreadLocalMap的key为啥要设置成WeakReference类型呢？
从表面上看内存泄漏的根源在于使用了弱引用。网上的文章大多着重分析ThreadLocal使用了弱引用会导致内存泄漏，但是另一个问题也同样值得思考：为什么使用弱引用而不是强引用？
我们先来看看官方文档的说法：
To help deal with very large and long-lived usages, the hash table entries use WeakReferences for keys.
为了帮助处理非常大且长期使用的用法，哈希表条目使用WeakReferences作为键。

下面我们分两种情况讨论：

key 使用强引用：引用的ThreadLocal的对象被回收了，但是ThreadLocalMap还持有ThreadLocal的强引用，如果没有手动删除，ThreadLocal不会被回收，导致Entry内存泄漏。
key 使用弱引用：引用的ThreadLocal的对象被回收了，由于ThreadLocalMap持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。value在下一次ThreadLocalMap调用set,get，remove的时候会被清除。
比较两种情况，我们可以发现：由于ThreadLocalMap的生命周期跟Thread一样长，如果都没有手动删除对应key，都会导致内存泄漏，但是使用弱引用可以多一层保障：弱引用ThreadLocal不会内存泄漏，对应的value在下一次ThreadLocalMap调用set,get,remove的时候会被清除。

因此，ThreadLocal内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。

```java
static class ThreadLoaclMap{
    public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
}
```

### 什么情况下内存会泄漏呢？
假设你用的是线程池，那池子里面的线程自始至终都是活的，线程不被销毁，并且你用完后就没怎么操作过这个ThreadLocal，key虽然会在gc时被回收，value一直被ThreadLocalMap引用着，可能会造成value的累积。

### 个人感觉
大家都说ThreadLocal用的时候会造成内存泄漏，原因呢大部分归结到弱引用，完了巴拉巴拉，说的人一头雾水。给我的感觉哈其实ThreadLocalMap用了WeakReferences做为键，并且在set,get,remove里面清除value，恰恰是未了防止内存泄漏。他写的时候想法很好，我把key设置WeakReferences，gc的时候就回收了，完了你再操作ThreadLocal的时候呢，我把value也帮着你删了。泄漏的原因其实就是你不会用。。。。。哈哈

### 解决方案
记得用完remove了 兄弟。
调用remove方法，就会删除对应的Entry对象，可以避免内存泄露，所以使用完ThreadLocal之后，应该调用remove方法，或者使用拦截器调用remove方法

## ThreadLocal的空指针异常（NPE）
装箱拆箱导致的，而不是ThreadLocal的问题
```java
ThreadLocal<Long> ThreadLocal = new ThreadLocal<Long>();
public long get(){
    return ThreadLocal.get();//拆箱转成long报错，如果value是空的话
}
public Long get(){
    return ThreadLocal.get();
}
```

## 共享对象
如果在每个线程中ThreadLocal.set()进去的东西本来就是多线程共享的同一个对象，比如static对象，那么多个线程的ThreadLocal.get()取得的还是这个共享对象本身，还是有并发访问问题

# ThreadLocal注意点
+ 如果可以不使用ThreadLocal就解决问题，那么不要强行使用
    例如在任务数很少的时候，在局部变量中可以新建对象就可以解决问题，那么就不需要使用到ThreadLocal
+ 优先使用框架的支持，而不是自己创造
    例如在Spring中，如果可以使用RequestContextHolder，那么就不需要自己维护ThreadLocal，因为自己可能会忘记调用remove()方法等，造成内存泄露

# 实际应用场景：在Spring中的实例分析
+ DateTimeContextHolder类，看到里面用了ThreadLocal
+ RequestContextHolder类，在每次HTTP请求中都对应一个线程，线程之间相互隔离，这就是ThreadLocal的典型应用场景
