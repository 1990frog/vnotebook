![9c1f724d6e444ed9bb8079928cbbf826](_v_images/20200329163648607_1307494099.png)


![2d607f6d670a440fa25a812f9373e6b5](_v_images/20200329163711035_2092389863.png)

以Reader这块来讲解装饰器模式在IO中的应用

+ 抽象类
Reader，是一个抽象类，里面定义了接口
+ 被装饰的类
InputStreamReader、CharArrayReader、PipedReader、StringReader、FileReader
+ 装饰器类
FilterReader、BufferedReader
+ 具体的装饰器类
BufferedReader、PushbackReader
FileReader尽管也在第三行，但是FileReader构不成一个具体的装饰器类，因为它不是BufferedReader的子类也不是FilterReader的子类，不持有Reader的引用
读写IO的时候每次打开文件会非常耗时，我们可以使用BufferedReader装饰FileReader这些被装饰器类，使用缓冲提高性能


![](_v_images/20191121165156097_391536508.png)

![](_v_images/20191121165216749_2002889243.png)

![](_v_images/20191121165233356_463152762.png)

![](_v_images/20191121165252172_462926785.png)

![IO系统](_v_images/20191121165537828_343432901.png)









+ <font color="red">Java BIO (blocking I/O)</font>： 同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。
+ Java NIO (non-blocking I/O)： 同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。
+ Java AIO(NIO.2) (Asynchronous I/O) ： 异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理，

BIO、NIO、AIO适用场景分析:

+ BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中
+ NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。
+ AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

# BIO中的阻塞
ServerSocket.accept()
InputStream.read()
OutputStream.write()
无法在同一个线程里处理多个Stream I/O