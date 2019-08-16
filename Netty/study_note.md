# study_note

Netty各组件：
1.BootStrap在netty的应用程序中负责引导服务器和客户端。netty包含了两种不同类型的引导： 
    1) 使用服务器的ServerBootStrap，用于接受客户端的连接以及为已接受的连接创建子通道。 
    2) 用于客户端的BootStrap，不接受新的连接，并且是在父通道类完成一些操作。
2.EventLoop：事件循环
3.EventLoopGroup：事件循环组
Netty的事件循环机制有两个基本接口：EventLoop和EventLoopGroup。前者是事件循环，后者是由多个事件循环组成的组。每个EventLoop被包装为一个Task放在在线程池中运行，但其本身也可以看做一个线程池，如Nio的事件循环会不断select后获取任务并执行。Nio的事件循环在实现时，使用死循环的方式不断select(),然后处理提交给EventLoop的系统任务。因此，我们可以将NioEventLoop当做线程池，EventLoopGroup作为线程池组，线程池组的意义是采用给的的策略选取一个EventLoop并提交任务。
EventLoop的定义如下，其继承了一个顺序执行的线程池接口和EventLoopGroup，也就是说EventLoop之间有父子关系，通过parent();返回任务循环组，通过next()选取一个事件循环。线程池组的register用于将Netty的Channel注册到线程池中。


Bootstrap 和 ServerBootstrap（引导类）
Bootstrap 和 ServerBootstrap 这两个引导类分别是用来处理客户端和服务端的信息，服务器端的引导一个父 Channel 用来接收客户端的连接，一个子 Channel 用来处理客户端和服务器端之间的通信，客户端则只需要一个单独的、没有父 Channel 的 Channel 来去处理所有的网络交互（或者是无连接的传输协议，如 UDP）


Bootstrap（引导）
ServerBootstrap：Builder模式，引导整个流程
ServerBootstrap和Bootstrap相比最大的区别，是拥有2个EventLoopGroup，一个用于ServerSocketChannel的accept（bossGroup），一个用于SocketChannel的读写事件（workerGroup）


EventLoopGroup：EventLoopGroup是eventLoop的集合，它管理着所有的eventloop的生命周期
bossGroup：创建处理连接的线程池
workerGroup：创建处理所有事件的线程池


EventLoop：每个channel都会被注册到EventLoop中, eventLoop会负责处理该channel的所有IO事件， 通常情况下，每个eventLoop会管理多个channel。这里可以类比下NIO的编程，这里的EventLoop类似于带有selector组件的ServerSocketChannel， 它负责多个channel的IO事件






Bytebuf: 对java nio中ByteBuffer的抽象


Channel：每个Channel代表了一种能够用于IO操作的实体，例如socket，文件等
pipeline：每个Channel都有一个pipeline，pipeline的作用是处理和这个channel相关的所有IO事件

channelHandler：每个pipeline中有一系列的channelHandler， 每个channelHandler是真实处理IO事件的地方， 并把这个事件沿着Pipeline传播. ChannelHandler可大体分为两类，一类是InboundHandler，也就是处理进来的消息的处理器，另一种则是OutboundHandler， 是用来处理出去的消息的





ChannelOption



Netty服务器启动流程：
1、创建线程池创建处理连接的线程池:bossGroup
创建处理所有事件的线程池:workerGroup
EventLoopGroup bossGroup = new NioEventLoopGroup();    
EventLoopGroup workerGroup = new NioEventLoopGroup();
2、设定辅助启动类。ServerBootStrap传入1中开辟的线程池指定连接该服务器的channel类型指定需要执行的childHandler设置部分参数，如AdaptiveRecvByteBufAllocator缓存大小.Option用于设置bossGroup相关参数.childOption用于设置workerGroup相关参数




ServerBootstrap的group干啥的


ChannelInitializer

Channel族：
Channel
ChannelHandler:
    InboundHandler
    OutboundHandler
ChannelPipelin
ChannelHandlerContext

Netty中的Channel接口继承了：AttributeMap, ChannelOutboundInvoker, Comparable<Channel>
其中ChannelOutboundInvoker继承了ChannelHandler

Netty的Channnel与NIO中Channel区别



Netty中设计模式：
1. 构造器模式
ServerBootstrap 和 Bootstrap 的构建
2. 责任链设计模式
pipeline 上事件的传播
3. 工厂模式
Channel 的实例化过程
4. 对象池
对线程池的应用，ByteBuf内存池
5. Reactor 模式的使用
Netty 底层事件的收发机制是多线程的 Reactor 模式的应用。
6. 模板模式
ServerBootstrap 和 Bootstrap 继承 AbstractBootstrap 父类抽象类，并实现init() 和clone()方法。
