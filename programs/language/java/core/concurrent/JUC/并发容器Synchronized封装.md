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

# 将线程不安全的容器类转化成线程安全：Collections工具类
## ArrayList
Collections.SynchronizedList
```java
static class SynchronizedList<E> extends SynchronizedCollection<E> implements List<E> {

    private static final long serialVersionUID = -7754090372962971524L;

    final List<E> list;

    SynchronizedList(List<E> list) {
        super(list);
        this.list = list;
    }
    SynchronizedList(List<E> list, Object mutex) {
        super(list, mutex);
        this.list = list;
    }

    public boolean equals(Object o) {
        if (this == o)
            return true;
        synchronized (mutex) {return list.equals(o);}
    }
    public int hashCode() {
        synchronized (mutex) {return list.hashCode();}
    }

    public E get(int index) {
        synchronized (mutex) {return list.get(index);}
    }
    public E set(int index, E element) {
        synchronized (mutex) {return list.set(index, element);}
    }
    public void add(int index, E element) {
        synchronized (mutex) {list.add(index, element);}
    }
    public E remove(int index) {
        synchronized (mutex) {return list.remove(index);}
    }

    public int indexOf(Object o) {
        synchronized (mutex) {return list.indexOf(o);}
    }
    public int lastIndexOf(Object o) {
        synchronized (mutex) {return list.lastIndexOf(o);}
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        synchronized (mutex) {return list.addAll(index, c);}
    }

    public ListIterator<E> listIterator() {
        return list.listIterator(); // Must be manually synched by user
    }

    public ListIterator<E> listIterator(int index) {
        return list.listIterator(index); // Must be manually synched by user
    }

    public List<E> subList(int fromIndex, int toIndex) {
        synchronized (mutex) {
            return new SynchronizedList<>(list.subList(fromIndex, toIndex),
                                        mutex);
        }
    }

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        synchronized (mutex) {list.replaceAll(operator);}
    }
    @Override
    public void sort(Comparator<? super E> c) {
        synchronized (mutex) {list.sort(c);}
    }

    /**
     * SynchronizedRandomAccessList instances are serialized as
     * SynchronizedList instances to allow them to be deserialized
     * in pre-1.4 JREs (which do not have SynchronizedRandomAccessList).
     * This method inverts the transformation.  As a beneficial
     * side-effect, it also grafts the RandomAccess marker onto
     * SynchronizedList instances that were serialized in pre-1.4 JREs.
     *
     * Note: Unfortunately, SynchronizedRandomAccessList instances
     * serialized in 1.4.1 and deserialized in 1.4 will become
     * SynchronizedList instances, as this method was missing in 1.4.
     */
    private Object readResolve() {
        return (list instanceof RandomAccess
                ? new SynchronizedRandomAccessList<>(list)
                : this);
    }
}
```
Collections.SynchronizedRandomAccessList
```java
static class SynchronizedRandomAccessList<E> extends SynchronizedList<E> implements RandomAccess {

    SynchronizedRandomAccessList(List<E> list) {
        super(list);
    }

    SynchronizedRandomAccessList(List<E> list, Object mutex) {
        super(list, mutex);
    }

    public List<E> subList(int fromIndex, int toIndex) {
        synchronized (mutex) {
            return new SynchronizedRandomAccessList<>(
                list.subList(fromIndex, toIndex), mutex);
        }
    }

    private static final long serialVersionUID = 1530674583602358482L;

    /**
     * Allows instances to be deserialized in pre-1.4 JREs (which do
     * not have SynchronizedRandomAccessList).  SynchronizedList has
     * a readResolve method that inverts this transformation upon
     * deserialization.
     */
    private Object writeReplace() {
        return new SynchronizedList<>(list);
    }
}
```
## Map
## Set