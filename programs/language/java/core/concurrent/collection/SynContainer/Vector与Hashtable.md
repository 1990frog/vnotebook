[TOC]

# 概览
大量使用Synchronized，性能差

# Vector
```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public synchronized boolean isEmpty();
    public synchronized void setElementAt(E obj, int index);
}
```

# Hashtable
```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable
{
    public synchronized V get(Object key);
}
```

