[TOC]

![](https://gitee.com/caijingquan/imagebed/raw/master/1602318247_20200329152311223_2053232899.png)

# BIO
传统的`java.io`包，它是基于`流模型`实现的，交互的方式是同步、阻塞方式，也就是说在读入输入流或者输出流时，在读写动作完成之前，线程会一直阻塞在那里，它们之间的调用时可靠的线性顺序。
它的有点就是代码比较简单、直观；缺点就是 IO 的效率和扩展性很低，容易成为应用性能瓶颈

# 分类
+ InputStream、OutputStream基于字节操作的IO
+ Writer、Reader基于字符操作的IO
+ File基于磁盘操作的IO
+ Socket基于网络操作的IO

# InputStream
抽象类非接口
```java
//读取单字节数据，返回的是"无符号的byte"类型值[0,255]，因为不存在无符号byte类型，所以返回值是int
public abstract int read() throws IOException
//读取字节数组到指定的byte数组中
public int read(byte b[]) throws IOException
//读取字节数组到指定的byte数组中，可以指定偏移量和总长度
public int read(byte b[], int off, int len) throws IOException
//跳过指定长度的字节序列
public long skip(long n) throws IOException 
//返回剩余可读取的字节序列的长度
public int available() throws IOException
//关闭输入流
public void close() throws IOException
//标记
public synchronized void mark(int readlimit)
//重置
public synchronized void reset() throws IOException
//是否支持标记和重置
public boolean markSupported()
```

