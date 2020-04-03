[TOC]

# 请求的过程
浏览器发给服务端的是一个HTTP格式的请求，HTTP服务器收到这个请求后，需要调用服务端程序来处理，所谓的服务端程序就是你写的Java类，一般来说不同的请求需要由不同的java类来处理。

HTTP服务器怎么知道要调用哪个Java类的哪个方法呢。最直接的做法是在HTTP服务器代码里写一大堆`if else`逻辑判断：
如果是A请求就调X类的M1方法
如果是B请求就调Y类的M2方法

但这样做明显有问题，因为HTTP服务器的代码跟业务逻辑耦合在一起了，如果新加一个业务方法还要改HTTP服务器的代码。

面向接口编程是解决耦合问题的法宝，于是就定义了一个接口，各种业务类都必须实现这个接口，这个接口就叫Servlet接口，<font color="red">有时我们也把实现了Servlet接口的业务类叫作Servlet。</font>

但是这里还有一个问题，对于特定的请求，HTTP服务器如何知道由哪个Servlet来处理呢？Servlet又是由谁来实例化呢？

显然HTTP服务器不适合做这个工作，否则又和业务类耦合了。
于是就有了Servlet容器，Servlet容器用来加载和管理业务类。
HTTP服务器不直接跟业务类打交道，而是把请求交给Servlet容器去处理，Servlet容器会将请求转发到具体的Servlet，如果这个Servlet还没创建，就加载并实例化这个Servlet，然后调用这个Servlet的接口方法。
因此Servlet接口其实是Servlet容器跟具体业务类之间的接口。

![dfe304d3336f29d833b97f2cfe8d7801](_v_images/20200402202719303_754562429.jpg)

作为 Java 程序员，如果我们要实现新的业务功能，只需要实现一个 Servlet，并把它注册到 Tomcat（Servlet 容器）中，剩下的事情就由 Tomcat 帮我们处理了。

# Servlet接口
```java
public interface Servlet {
    void init(ServletConfig config) throws ServletException;
    
    ServletConfig getServletConfig();
    
    void service(ServletRequest req, ServletResponse res）throws ServletException, IOException;
    
    String getServletInfo();
    
    void destroy();
}
```

## service方法
其中最重要是的 service 方法，具体业务类在这个方法里实现处理逻辑。

这个方法有两个参数，本质上这两个类是对通信协议的封装：
+ ServletRequest用来封装请求信息
+ ServletResponse用来封装响应信息
## init方法
Servlet容器在加载Servlet类的时候会调用init方法

我们可能会在init方法里初始化一些资源，并在destroy方法里释放这些资源，比如Spring MVC中的DispatcherServlet，就是在init方法里创建了自己的Spring容器。
## destory方法
在卸载的时候会调用destroy方法
## ServletConfig
ServletConfig 这个类，ServletConfig 的作用就是封装 Servlet 的初始化参数。
你可以在web.xml给 Servlet 配置参数，并在程序里通过 getServletConfig 方法拿到这些参数。

# Servlet工作流程
+ 客户请求某个资源
+ HTTP服务器会用一个ServletRequest对象把客户的请求信息封装起来
+ 调用Servlet容器的service方法（懒加载，如果没被创建那就反射创建，并init初始化）
+ 调用Servlet的service方法来处理请求，把ServletResponse对象返回给HTTP服务器，HTTP服务器会把响应发送给客户端
![b70723c89b4ed0bccaf073c84e08e115](_v_images/20200402204056187_1024729779.jpg)

# servlet注册到web容器
我们是以Web应用程序的方式来部署Servlet的，而根据Servlet规范，Web应用程序有一定的目录结构，在这个目录下分别放置了Servlet的类文件、配置文件以及静态资源，Servlet 容器通过读取配置文件，就能找到并加载Servlet

Web应用的目录结构大概是下面这样的：
```
| -  MyWebApp
      | -  WEB-INF/web.xml        -- 配置文件，用来配置Servlet等
      | -  WEB-INF/lib/           -- 存放Web应用所需各种JAR包
      | -  WEB-INF/classes/       -- 存放你的应用类，比如Servlet类
      | -  META-INF/              -- 目录存放工程的一些信息
```

# ServletContext 上下文
Servlet规范里定义了ServletContext这个接口来对应一个Web应用。

Web应用部署好后，Servlet容器在启动时会加载Web应用，并为每个Web应用创建唯一的ServletContext对象。
你可以把ServletContext看成是一个全局对象，一个Web应用可能有多个Servlet，这些Servlet可以通过全局的ServletContext来共享数据，这些数据包括Web应用的初始化参数、Web应用目录下的文件资源等

由于ServletContext持有所有Servlet实例，你还可以通过它来实现Servlet请求的转发。

# 扩展机制
不知道你有没有发现，引入了 Servlet 规范后，你不需要关心 Socket 网络通信、不需要关心 HTTP 协议，也不需要关心你的业务类是如何被实例化和调用的，因为这些都被 Servlet 规范标准化了，你只要关心怎么实现的你的业务逻辑。

这对于程序员来说是件好事，但也有不方便的一面。所谓规范就是说大家都要遵守，就会千篇一律，但是如果这个规范不能满足你的业务的个性化需求，就有问题了，因此设计一个规范或者一个中间件，要充分考虑到可扩展性。

Servlet 规范提供了两种扩展机制：
+ Filter
+ Listener

# Filter
Filter是过滤器，这个接口允许你对请求和响应做一些统一的定制化处理，比如你可以根据请求的频率来限制访问，或者根据国家地区的不同来修改响应内容。

过滤器的工作原理是这样的：
Web应用部署完成后，Servlet容器需要实例化Filter并把Filter链接成一个FilterChain（责任链模式）。当请求进来时，获取第一个Filter并调用 doFilter方法，doFilter方法负责调用这个FilterChain中的下一个Filter。

# Listener
Listener是监听器，这是另一种扩展机制。

当Web应用在Servlet容器中运行时，Servlet容器内部会不断的发生各种事件，如Web应用的启动和停止、用户请求到达等。

Servlet容器提供了一些默认的监听器来监听这些事件，当事件发生时，Servlet容器会负责调用监听器的方法（事件驱动机制）。

当然，你可以定义自己的监听器去监听你感兴趣的事件，将监听器配置在web.xml中。

比如Spring就实现了自己的监听器（Listener模块，event触发广播器），来监听ServletContext的启动事件，目的是当Servlet容器启动时，创建并初始化全局的Spring容器。

最后我给你总结一下Filter和Listener的本质区别：
Filter是干预过程的，它是过程的一部分，是基于过程行为的。
Listener是基于状态的，任何行为改变同一个状态，触发的事件是一致的。

# Spring
Tomcat&Jetty在启动过程中触发容器初始化事件，Spring的ContextLoaderListener会监听到这个事件，它的contextInitialized方法会被调用，在这个方法中，Spring会初始化全局的Spring根容器，这个就是Spring的IoC容器，IoC容器初始化完毕后，Spring将其存储到ServletContext中，便于以后来获取。

Tomcat&Jetty在启动过程中还会扫描Servlet，一个Web应用中的Servlet可以有多个，以SpringMVC中的DispatcherServlet为例，这个Servlet实际上是一个标准的前端控制器，用以转发、匹配、处理每个Servlet请求。

Servlet一般会延迟加载，当第一个请求达到时，Tomcat&Jetty发现DispatcherServlet还没有被实例化，就调用DispatcherServlet的init方法，DispatcherServlet在初始化的时候会建立自己的容器，叫做SpringMVC容器，用来持有Spring MVC相关的Bean。同时，Spring MVC还会通过ServletContext拿到Spring根容器，并将Spring根容器设为SpringMVC容器的父容器，请注意，Spring MVC容器可以访问父容器中的Bean，但是父容器不能访问子容器的Bean， 也就是说Spring根容器不能访问SpringMVC容器里的Bean。说的通俗点就是，在Controller里可以访问Service对象，但是在Service里不可以访问Controller对象。

Spring和SpringMVC分别有自己的IOC容器或者说上下文。

为什么要分成两个容器呢？为了职责划分清晰。

SpringMVC的容器直接管理跟DispatcherServlet相关的Bean，也就是Controller，ViewResolver等，并且SpringMVC容器是在DispacherServlet的init方法里创建的。

而Spring容器管理其他的Bean比如Service和DAO。

并且SpringMVC容器是Spring容器的子容器，所谓的父子关系意味着什么呢，就是你通过子容器去拿某个Bean时，子容器先在自己管理的Bean中去找这个Bean，如果找不到再到父容器中找。但是父容器不能到子容器中去找某个Bean。

其实这个套路跟JVM的类加载器设计有点像，不同的类加载器也为了隔离，不过加载顺序是反的，子加载器总是先委托父加载器去加载某个类，加载不到再自己来加载。

