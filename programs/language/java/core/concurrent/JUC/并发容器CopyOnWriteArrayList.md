[TOC]

# CopyOnWriteArrayList
+ 代替Vector和SynchronizedList，就和ConcurrentHashMap代替SynchronizedMap的原因一样
+ Vector和SynchronizedList的锁的粒度太大，并发效率相对比较低，并且迭代时无法编辑
+ Copy-On-Write并发容器还包括CopyOnWriteArraySet，用来替代同步Set

# CopyOnWriteArrayList适用场景
+ 读操作可以尽可能地快，而写操作即使慢一些也没有太大关系
+ 读多写少：黑名单，每日更新；监听器：迭代操作远多余修改操作

# CopyOnWriteArrayList读写规则
+ 回顾读写锁：读读共享、其他都互斥（写写互斥、读写互斥、写读互斥）
+ 读写锁规则的升级：读取是完全不用加锁的，并且更厉害的是写入也不会阻塞读取操作。只有写入和写入之间需要进行同步等待

![](_v_images/20200213113858608_880998620.png)



hashmap不能在迭代的时候修改

![](_v_images/20200213113918600_1178618175.png)





CopyOnWriteArrayList可以在迭代中修改


# CopyOnWriteArrayList实现原理
+ CopyOnWrite的含义
+ 创建新副本、读写分离
+ “不可变”原理
+ 迭代的时候（依然使用旧数组不报错）


![](_v_images/20200213113944720_2065032430.png)
![](_v_images/20200213113955136_412785180.png)



modCount修改的次数

![](_v_images/20200213114015872_1305610672.png)



迭代出现数据过期的问题

# CopyOnWriteArrayList的缺点
+ 数据一致性问题：CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的数据，马上能读到，请不要使用CopyOnWrite容器
+ 内存占用问题：因为CopyOnWrite的写是复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存


![](_v_images/20200213114033680_1731141074.png)

