[TOC]

# IO介绍
我们通常所说的BIO是相对于NIO来说的，BIO也就是Java开始之初推出的IO操作模块，BIO是Blocking IO的缩写，顾名思义就是阻塞IO的意思。

# BIO、NIO、AIO的区别
+ BIO 就是传统的 java.io 包，它是基于<font color="red">**流模型**</font>实现的，交互的方式是同步、阻塞方式，也就是说在读入输入流或者输出流时，在读写动作完成之前，线程会一直阻塞在那里，它们之间的调用时可靠的线性顺序。它的有点就是代码比较简单、直观；缺点就是 IO 的效率和扩展性很低，容易成为应用性能瓶颈。
+ NIO 是 Java 1.4 引入的java.nio包，提供了Channel、Selector、Buffer等新的抽象，可以<font color="red">**构建多路复用**</font>的、同步非阻塞IO程序，同时提供了更接近操作系统底层高性能的数据操作方式。
+ AIO 是Java 1.7之后引入的包，是NIO的升级版本，提供了异步非堵塞的IO操作方式，所以人们叫它AIO（Asynchronous IO），异步IO是<font color="red">**基于事件和回调机制**</font>实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

# 全面认识 IO
传统的 IO 大致可以分为4种类型：

+ 字节IO：InputStream、OutputStream
+ 字符IO：Writer、Reader
+ 磁盘操作IO：File
+ 网络IO：Socke（java.net下提供的Scoket很多时候人们也把它归为同步阻塞IO ,因为网络通讯同样是IO行为）

java.io 下的类和接口很多，但大体都是 InputStream、OutputStream、Writer、Reader 的子集，所有掌握这4个类和File的使用，是用好 IO 的关键。