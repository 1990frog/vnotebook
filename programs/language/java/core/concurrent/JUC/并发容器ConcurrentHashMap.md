[TOC]

# 概览
+ 取代同步的HashMap
+ ConcurrentHashMap任何情景之下都强于其他方法

# JDK7 HashMap结构（拉链法）
![5900919-95929ae090181652](_v_images/20200211190932963_527634586.png)

```java
static class Entry<K,V> implements Map.Entry<K,V>{
    final K key;
    V value;
    Entry<K,V> next;
    int hash;
}
```

# JDK8 HashMap结构（红黑树 or 拉链法）
![5900919-24f05cf9c03d1f9d](_v_images/20200211191204372_427791586.png)



# HashMap关于并发的特点
1. 非线程安全
2. 迭代时不允许修改内容
3. 只读的并发是安全的
4. 如果一定要把HashMap用在并发环境，用Collections.synchronizedMap(new HashMap())


![](_v_images/20200125165253070_1774560434.png)




# jdk7的ConcurrentHashMap实现和分析
+ java7中的ConcurrentHashMap最外层是多个segment，每个segment的底层数据结构与HashMap类似，仍然是数组和链表组成的拉链法
+ 每个segment独立上ReentrantLock锁，每个segment之间互不影响，提高了并发效率
+ ConcurrentHashMap默认有16个Segments，所以最多可以同时支持16个线程并发写（操作分别分布在不同的Segment上）。这个默认值可以在初始化的时候设置为其他值，但是一旦初始化以后，是不可以扩容的

# putVal流程
+ 判断key value不为空
+ 计算hash值
+ 根据对应位置节点的类型，来赋值，或者helpTransfer，或者增长链表，或者给红黑树增加节点
+ 检查满足阈值就“红黑树”化
+ 返回oldVal

# get流程
+ 计算hash值
+ 找到对应的位置，根据情况进行：
+ 直接取值
+ 红黑树里找值
+ 遍历链表取值
+ 返回找到的结果

# 为什么要把jdk7的结构改成jdk8的结构
+ 数据结构
+ hash碰撞
+ 保证并发安全
+ 查询复杂度
+ 为什么超过8要转化为红黑树？


```java
* first values are:
*
* 0:    0.60653066
* 1:    0.30326533
* 2:    0.07581633
* 3:    0.01263606
* 4:    0.00157952
* 5:    0.00015795
* 6:    0.00001316
* 7:    0.00000094
* 8:    0.00000006
* more: less than 1 in ten million
```

默认是链表的形式，因为它占用的内存更少

# 错误使用造成线程不安全
```java
public void run(){
    for(int i=0;i<1000;i++){
        Integer score = scores.get("小明");
        Integer newScore = score + 1;
        socres.put("小明",newScore);
    }
}
```
在hashmap中多个线程同时put会造成结果不可预知
上述的代码使用hashmap也是安全的


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

replace
putIfAbsent
