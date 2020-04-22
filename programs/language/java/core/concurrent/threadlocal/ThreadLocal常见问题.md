[TOC]

# 内存泄露
某个对象不再有用，但是占用的内存却不能被回收
![20200130140258339_358079252](_v_images/20200404004812584_2101087807.jpg)


ThreadLocal的原理：每个Thread内部维护着一个ThreadLocalMap，它是一个Map。这个映射表的Key是一个弱引用，其实就是ThreadLocal本身，Value是真正存的线程变量Object。
也就是说ThreadLocal本身并不真正存储线程的变量值，它只是一个工具，用来维护Thread内部的Map，帮助存和取。注意上图的虚线，它代表一个弱引用类型，而弱引用的生命周期只能存活到下次GC前。

## ThreadLocal为什么会内存泄漏
ThreadLocal在ThreadLocalMap中是以一个弱引用身份被Entry中的Key引用的，因此如果ThreadLocal没有外部强引用来引用它，那么ThreadLocal会在下次JVM垃圾收集时被回收。
这个时候就会出现Entry中Key已经被回收，出现一个null Key的情况，外部读取ThreadLocalMap中的元素是无法通过null Key来找到Value的。
因此如果当前线程的生命周期很长，一直存在，那么其内部的ThreadLocalMap对象也一直生存下来，这些null key就存在一条强引用链的关系一直存在：Thread --> ThreadLocalMap-->Entry-->Value，这条强引用链会导致Entry不会回收，Value也不会回收，但Entry中的Key却已经被回收的情况，造成内存泄漏。

<font color="red">但是JVM团队已经考虑到这样的情况，并做了一些措施来保证ThreadLocal尽量不会内存泄漏：在ThreadLocal的get()、set()、remove()方法调用的时候会清除掉线程ThreadLocalMap中所有Entry中Key为null的Value，并将整个Entry设置为null，利于下次内存回收。</font>

将Entry的Key设置成弱引用，在配合线程池使用的情况下可能会有内存泄露的风险。之设计成弱引用的目的是为了更好地对ThreadLocal进行回收，当我们在代码中将ThreadLocal的强引用置为null后，这时候Entry中的ThreadLocal理应被回收了，但是如果Entry的key被设置成强引用则该ThreadLocal就不能被回收，这就是将其设置成弱引用的目的。

WeakReference对象的特性就是代码运行一段时间，如果碰到gc，WeakReference指向的对象会被gc回收。

WeakReference的一个特点是它何时被回收是不可确定的, 因为这是由GC运行的不确定性所确定的. 所以, 一般用weak reference引用的对象是有价值被cache, 而且很容易被重新被构建, 且很消耗内存的对象.

## ThreadLocalMap的key为啥要设置成WeakReference类型呢？
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

## 什么情况下内存会泄漏呢？
假设你用的是线程池，那池子里面的线程自始至终都是活的，线程不被销毁，并且你用完后就没怎么操作过这个ThreadLocal，key虽然会在gc时被回收，value一直被ThreadLocalMap引用着，可能会造成value的累积。

## 个人感觉
大家都说ThreadLocal用的时候会造成内存泄漏，原因呢大部分归结到弱引用，完了巴拉巴拉，说的人一头雾水。给我的感觉哈其实ThreadLocalMap用了WeakReferences做为键，并且在set,get,remove里面清除value，恰恰是未了防止内存泄漏。他写的时候想法很好，我把key设置WeakReferences，gc的时候就回收了，完了你再操作ThreadLocal的时候呢，我把value也帮着你删了。泄漏的原因其实就是你不会用。。。。。哈哈

## 解决方案
记得用完remove了 兄弟。
调用remove方法，就会删除对应的Entry对象，可以避免内存泄露，所以使用完ThreadLocal之后，应该调用remove方法，或者使用拦截器调用remove方法

# 空指针异常（NPE）
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

# 共享对象
如果在每个线程中ThreadLocal.set()进去的东西本来就是多线程共享的同一个对象，比如static对象，那么多个线程的ThreadLocal.get()取得的还是这个共享对象本身，还是有并发访问问题

# ThreadLocal注意点
+ 如果可以不使用ThreadLocal就解决问题，那么不要强行使用
    例如在任务数很少的时候，在局部变量中可以新建对象就可以解决问题，那么就不需要使用到ThreadLocal
+ 优先使用框架的支持，而不是自己创造
    例如在Spring中，如果可以使用RequestContextHolder，那么就不需要自己维护ThreadLocal，因为自己可能会忘记调用remove()方法等，造成内存泄露

# 实际应用场景：在Spring中的实例分析
+ DateTimeContextHolder类，看到里面用了ThreadLocal
+ RequestContextHolder类，在每次HTTP请求中都对应一个线程，线程之间相互隔离，这就是ThreadLocal的典型应用场景
