[TOC]

Hashtable：
基于数组（hash槽）的单链表

Hashtable 和 HashMap 十分类似, 它是继承 Dictionary, 这个类已经不推荐使用了. 下面列举几点 Hashtable 与 HashMap 的不同之处.

Hashtable 是线程安全的;
Hashtable 的数组长度可以自定义, 不一定是 2 的幂次方, 默认初始长度是 11;
Hashtable 同样用链表保存, 但不会转换成树结构;
Hashtable 的 key, value 都不允许为空.


# 实现Dictionary抽象类
![Dictionary](_v_images/20200405142705305_1832715607.png)

```java
public abstract
class Dictionary<K,V> {
    public Dictionary() {
    }

    abstract public int size();

    abstract public boolean isEmpty();

    abstract public Enumeration<K> keys();

    abstract public Enumeration<V> elements();

    abstract public V get(Object key);

    abstract public V put(K key, V value);

    abstract public V remove(Object key);
}

```

hashtable底层存储也是通过静态数组实现的

位置
int index = (hash & 0x7FFFFFFF) % tab.length;

线程安全是因为包了synchronized

扩容要rehash
```java
protected void rehash() {
    int oldCapacity = table.length;
    Entry<?,?>[] oldMap = table;

    // overflow-conscious code
    int newCapacity = (oldCapacity << 1) + 1;
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        if (oldCapacity == MAX_ARRAY_SIZE)
            // Keep running with MAX_ARRAY_SIZE buckets
            return;
        newCapacity = MAX_ARRAY_SIZE;
    }
    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

    modCount++;
    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    table = newMap;

    for (int i = oldCapacity ; i-- > 0 ;) {
        for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            Entry<K,V> e = old;
            old = old.next;

            int index = (e.hash & 0x7FFFFFFF) % newCapacity;
            e.next = (Entry<K,V>)newMap[index];
            newMap[index] = e;
        }
    }
}
```

Hashtable同样是基于哈希表实现的，同样每个元素是一个key-value对，其内部也是通过单链表解决冲突问题，容量不足（超过了阀值）时，同样会自动增长。

e.next = (Entry<K,V>)newMap[index];// 新元素设置为头结点
                newMap[index] = e;
每次从表头压入元素