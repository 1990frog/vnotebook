[TOC]

# Tomcat支持的I/O模型
+ NIO：非阻塞I/O，采用Java NIO类库实现
+ NIO.2：异步I/O，采用JDK7最新NIO.2类库实现
+ ARP：采用Apache可移植运行库实现，是C/C++编写的本地库

# Tomcat 支持的应用层协议
+ HTTP/1.1：这是大部分 Web 应用采用的访问协议
+ AJP：用于和 Web 服务器集成（如 Apache）
+ HTTP/2：HTTP 2.0 大幅度的提升了 Web 性能

Tomcat 为了实现支持多种 I/O 模型和应用层协议，一个容器可能对接多个连接器，就好比一个房间有多个门。但是单独的连接器或者容器都不能对外提供服务，需要把它们组装起来才能工作，组装后这个整体叫作 Service 组件。这里请你注意，Service 本身没有做什么重要的事情，只是在连接器和容器外面多包了一层，把它们组装在一起。Tomcat 内可能有多个 Service，这样的设计也是出于灵活性的考虑。通过在 Tomcat 中配置多个 Service，可以实现通过不同的端口号来访问同一台机器上部署的不同应用。

![ee880033c5ae38125fa91fb3c4f8cad6](https://gitee.com/caijingquan/imagebed/raw/master/1602319027_20200404161937580_1810772684.jpg)

从图上你可以看到，最顶层是 Server，这里的 Server 指的就是一个 Tomcat 实例。一个 Server 中有一个或者多个 Service，一个 Service 中有多个连接器和一个容器。连接器与容器之间通过标准的 ServletRequest 和 ServletResponse 通信。

# 连接器
连接器对 Servlet 容器屏蔽了协议及 I/O 模型等的区别，无论是 HTTP 还是 AJP，在容器中获取到的都是一个标准的 ServletRequest 对象。

我们可以把连接器的功能需求进一步细化，比如：
1. 监听网络端口。
2. 接受网络连接请求。
3. 读取网络请求字节流。
4. 根据具体应用层协议（HTTP/AJP）解析字节流，生成统一的 Tomcat Request 对象。
5. 将 Tomcat Request 对象转成标准的 ServletRequest。
6. 调用 Servlet 容器，得到 ServletResponse。
7. 将 ServletResponse 转成 Tomcat Response 对象。
8. 将 Tomcat Response 转成网络字节流。
9. 将响应字节流写回给浏览器。

需求列清楚后，我们要考虑的下一个问题是，连接器应该有哪些子模块？优秀的模块化设计应该考虑高内聚、低耦合。
+ 高内聚是指相关度比较高的功能要尽可能集中，不要分散。
+ 低耦合是指两个相关的模块要尽可能减少依赖的部分和降低依赖的程度，不要让两个模块产生强依赖。

通过分析连接器的详细功能列表，我们发现连接器需要完成 3 个高内聚的功能：
+ 网络通信。
+ 应用层协议解析。
+ Tomcat Request/Response 与 ServletRequest/ServletResponse 的转化。

因此 Tomcat 的设计者设计了 3 个组件来实现这 3 个功能，分别是:
+ Endpoint
+ Processor
+ Adapter

组件之间通过抽象接口交互。这样做还有一个好处是封装变化。这是面向对象设计的精髓，将系统中经常变化的部分和稳定的部分隔离，有助于增加复用性，并降低系统耦合度。

网络通信的 I/O 模型是变化的，可能是非阻塞 I/O、异步 I/O 或者 APR。应用层协议也是变化的，可能是 HTTP、HTTPS、AJP。浏览器端发送的请求信息也是变化的。

但是整体的处理逻辑是不变的：
+ Endpoint 负责提供字节流给 Processor
+ Processor 负责提供 Tomcat Request 对象给 Adapter
+ Adapter 负责提供 ServletRequest 对象给容器

由于 I/O 模型和应用层协议可以自由组合，比如 NIO + HTTP 或者 NIO.2 + AJP。Tomcat 的设计者将网络通信和应用层协议解析放在一起考虑，设计了一个叫 ProtocolHandler 的接口来封装这两种变化点。各种协议和通信模型的组合有相应的具体实现类。比如：Http11NioProtocol 和 AjpNioProtocol。

除了这些变化点，系统也存在一些相对稳定的部分，因此 Tomcat 设计了一系列抽象基类来封装这些稳定的部分，抽象基类 AbstractProtocol 实现了 ProtocolHandler 接口。每一种应用层协议有自己的抽象基类，比如 AbstractAjpProtocol 和 AbstractHttp11Protocol，具体协议的实现类扩展了协议层抽象基类。下面我整理一下它们的继承关系。

![13850ee56c3f09cbabe9892e84502155](https://gitee.com/caijingquan/imagebed/raw/master/1602319029_20200404164007094_1257911139.jpg)

小结一下，连接器模块用三个核心组件：Endpoint、Processor 和 Adapter 来分别做三件事情，其中 Endpoint 和 Processor 放在一起抽象成了 ProtocolHandler 组件，它们的关系如下图所示。

![6eeaeb93839adcb4e76c15ee93f545ce](https://gitee.com/caijingquan/imagebed/raw/master/1602319029_20200404164117758_593797161.jpg)

下面我来详细介绍这两个顶层组件 ProtocolHandler 和 Adapter。

# ProtocolHandler 组件
连接器用 ProtocolHandler 来处理网络连接和应用层协议，包含了 2 个重要部件：Endpoint 和 Processor，下面我来详细介绍它们的工作原理。
## Endpoint
Endpoint 是通信端点，即通信监听的接口，是具体的 Socket 接收和发送处理器，是对传输层的抽象，因此 Endpoint 是用来实现 TCP/IP 协议的。

Endpoint 是一个接口，对应的抽象实现类是 AbstractEndpoint，而 AbstractEndpoint 的具体子类，比如在 NioEndpoint 和 Nio2Endpoint 中，有两个重要的子组件：Acceptor 和 SocketProcessor。

其中 Acceptor 用于监听 Socket 连接请求。SocketProcessor 用于处理接收到的 Socket 请求，它实现 Runnable 接口，在 run 方法里调用协议处理组件 Processor 进行处理。为了提高处理能力，SocketProcessor 被提交到线程池来执行。而这个线程池叫作执行器（Executor)。

## Processor
如果说 Endpoint 是用来实现 TCP/IP 协议的，那么 Processor 用来实现 HTTP 协议，Processor 接收来自 Endpoint 的 Socket，读取字节流解析成 Tomcat Request 和 Response 对象，并通过 Adapter 将其提交到容器处理，Processor 是对应用层协议的抽象。

Processor 是一个接口，定义了请求的处理等方法。它的抽象实现类 AbstractProcessor 对一些协议共有的属性进行封装，没有对方法进行实现。具体的实现有 AjpProcessor、Http11Processor 等，这些具体实现类实现了特定协议的解析方法和请求处理方式。

![309cae2e132210489d327cf55b284dcf](https://gitee.com/caijingquan/imagebed/raw/master/1602319030_20200404164510314_29345540.jpg)

从图中我们看到，Endpoint 接收到 Socket 连接后，生成一个 SocketProcessor 任务提交到线程池去处理，SocketProcessor 的 run 方法会调用 Processor 组件去解析应用层协议，Processor 通过解析生成 Request 对象后，会调用 Adapter 的 Service 方法。

# Adapter 组件
我在前面说过，由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat 定义了自己的 Request 类来“存放”这些请求信息。ProtocolHandler 接口负责解析请求并生成 Tomcat Request 类。但是这个 Request 对象不是标准的 ServletRequest，也就意味着，不能用 Tomcat Request 作为参数来调用容器。Tomcat 设计者的解决方案是引入 CoyoteAdapter，这是适配器模式的经典运用，连接器调用 CoyoteAdapter 的 sevice 方法，传入的是 Tomcat Request 对象，CoyoteAdapter 负责将 Tomcat Request 转成 ServletRequest，再调用容器的 service 方法。

# 总结
Tomcat 的整体架构包含了两个核心组件连接器和容器。连接器负责对外交流，容器负责内部处理。连接器用 ProtocolHandler 接口来封装通信协议和 I/O 模型的差异，ProtocolHandler 内部又分为 Endpoint 和 Processor 模块，Endpoint 负责底层 Socket 通信，Processor 负责应用层协议解析。连接器通过适配器 Adapter 调用容器。通过对 Tomcat 整体架构的学习，我们可以得到一些设计复杂系统的基本思路。

首先要分析需求，根据高内聚低耦合的原则确定子模块，然后找出子模块中的变化点和不变点，用接口和抽象基类去封装不变点，在抽象基类中定义模板方法，让子类自行实现抽象方法，也就是具体子类去实现变化点。

# tomcat和netty有什么区别
可以把Netty理解成Tomcat中的连接器，它们都负责网络通信，都利用了Java NIO非阻塞特性。

但Netty素以高性能高并发著称，为什么Tomcat不把连接器替换成Netty呢？
第一个原因是Tomcat的连接器性能已经足够好了，同样是Java NIO编程，套路都差不多。
第二个原因是Tomcat做为Web容器，需要考虑到Servlet规范，Servlet规范规定了对HTTP Body的读写是阻塞的，因此即使用到了Netty，也不能充分发挥它的优势。

所以Netty一般用在非HTTP协议和Servlet的场景下