[TOC]

【写的不好，后面重写】

# ThreadLocalMap类
`ThreadLocal.ThreadLocalMap threadLocals = null;`

ThreadLocal.ThreadLocalMap静态类
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

# 线程获取ThreadLocal的过程
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