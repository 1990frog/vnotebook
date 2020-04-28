[TOC]

串行

# 特性
1. 互斥同步锁
2. 有序性（happen-before）
3. 原子性
4. 可见性
5. 可重入
6. 执行完毕自动释放锁（Lock不会）
7. 不可被中断
8. 无法尝试获取锁
9. 适合于密集型并发
# 锁的分类（synchronized）
+ 阻塞锁
+ 排它锁
+ 非公平锁
+ 可重入锁（递归锁）
+ 不可中断锁
# 不能被继承
synchronized关键字不能被多态继承。如果父类中的某个方法是synchronized的，而子类中覆盖了这个方法，那么默认情况下子类中的这个方法并不是synchronized的，必须显式的在子类的这个方法中加上synchronized关键字才行。当然，不覆盖的话没事。不过子类的那个方法也可以通过调用父类中该相应的方法来实现synchronized效果。
# 作用
JVM会自动通过使用[monitor](monitor.md)来加锁和解锁，保证了同时只有一个线程可以执行指定代码，从而保证了线程安全，同时具有可重入和不可中断的性质。
能保证在同一时刻最多只有一个线程执行该段代码，以达到保证并发安全的效果。
# 地位
+ `synchronizded`是Java的关键字，被Java语言原生支持
+ 是最基本的互斥同步手段
+ 是并发编程中的元老级角色，是并发编程的必学内容
# 缺陷
+ 效率低：锁的释放情况少、试图获得锁时不能设定超时、不能中断一个正在试图获得锁的线程（容易死锁）
+ 不够灵活（读写锁更灵活）：加锁和释放的时机单一，每个所仅有单一的条件（某个对象），可能是不够的
+ 无法知道是否成功获取到锁
# Synchronized的两种用法
+ 对象锁
+ 类锁（class对象是不是也可以看成对象锁呢？）
# 对象锁
## 概念
使用Object对象及其子类作为锁
## 本质
非Class对象作为锁
## 形式1：非静态方法（该类的实例为锁）
```java
public Test{
    public void sychronized met1(){}
}
```
## 形式2：synchronized代码块中，指定其锁（非class对象），或使用this
```java
public Test{
    public void met1(){
        sychronized(this){}
    }
}
```
## 形式1与形式2的区别
只要是Object对象的子类都能做锁，锁是自己，还是别人
# 类锁
## 概念（重要）
Java类可能有很多个对象，但只有一个Class对象（一个类只有一个class对象）
## 本质
所以所谓的类锁，不过是Class对象的锁而已。
## 形式1：synchronized加在static方法上
```java
public Test{
    public void static sychronized met1(){}
}
```
## 形式2：synchronized（*.class）代码块
```java
public Test{
    public void static met1(){
        sychronized(Test.class){}
    }
}
```
# 使用注意点
+ 锁对象不能为空
+ 作用域不宜过大（性能压力太大）
+ 避免死锁
# 思考
1. 多个线程等待同一个synchronized锁的时候，JVM如何选择下一个获取锁的是哪个线程？
2. Synchronized使得同时只有一个线程可以执行，性能较差，有什么办法可以提升性能？
3. 我想更灵活地控制锁的获取和释放（现在释放锁的时机都被规定死了），怎么办？
4. 什么是锁的升级、降级？什么是JVM里的偏斜所、轻量级锁、重量级锁？