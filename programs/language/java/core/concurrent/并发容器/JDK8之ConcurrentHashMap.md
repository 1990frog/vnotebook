[TOC]

# 概览
+ 取代同步的HashMap
+ ConcurrentHashMap任何情景之下都强于其他方法

# HashMap关于并发的特点
1. 非线程安全
2. 迭代时不允许修改内容
3. 只读的并发是安全的
4. 如果一定要把HashMap用在并发环境，用Collections.synchronizedMap(new HashMap())

# 组合操作
```java
public void run(){
    for(int i=0;i<1000;i++){
        while(true){
            Integer score = scores.get("小明");
            Integer newScore = socre + 1;
            boolean b = scores.replace("小明",score,newScore);
            if(b){
                break;
            }
        }
    }
}
```

# 变量
```java
transient volatile Node<K,V>[] table;
```

# 构造方法
```java
public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (initialCapacity < concurrencyLevel)   // Use at least as many bins
        initialCapacity = concurrencyLevel;   // as estimated threads
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?
        MAXIMUM_CAPACITY : tableSizeFor((int)size);
    this.sizeCtl = cap;
}
```
# 初始化容器
```java
/**
 * Initializes table, using the size recorded in sizeCtl.
 * sizeCtl是用于多线程之间同步的一个互斥变量。当sizeCtl < 0时，表示已经有线程正在初始化哈希表或哈希表正在扩容，此时，不能再进行操作。
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        // 正在扩容，线程等待
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        // CAS进入扩容设置值-1
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    // 以2的倍数扩容
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```
这里，可以提出一个问题，自选锁方式和死循环方式来判断sizeCtl的值，有什么不同？

当然是效率的不同。死循环时，程序会频繁读取sizeCtl的值，在满足条件之前，会浪费很多CPU周期。而自选锁的效率更高，因为当它判断sizeCtl不满足条件时，会主动让出CPU，此时，当前线程会处于ready状态，等待下一次被处理器选中并执行的机会。在这段时间里，其他的线程得以利用CPU周期。所以，自旋锁的效率更高。
# 添加元素
```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // null抛出
    if (key == null || value == null) throw new NullPointerException();
    // 获取当前key的hash值
    int hash = spread(key.hashCode());
    // 链表层级数，红黑树=0
    int binCount = 0;
    // 自旋
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        // 如果table为null那就初始化容器
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        // table对应key所在的位置为null，初始化链表（node）
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // CAS的方式添加node
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        // MOVED节点，转移节点，代表槽点正在扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        // 正常添加元素
        else {
            // 返回旧值
            V oldVal = null;
            // 使用node做锁
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    // 链表
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // BRTree
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            // 尝试转成红黑树
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```
# 转成红黑树
```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
```