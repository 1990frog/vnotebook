[TOC]

![1251492-20180507132038095-966635725](https://gitee.com/caijingquan/imagebed/raw/master/1602319189_20200328002328434_913232122.jpg)

# 生命周期
从创建到毁灭：
1. 调用 init() 方法初始化
2. 调用 service() 方法来处理客户端的请求
3. 调用 destroy() 方法释放资源，标记自身为可回收

被垃圾回收器回收

# 接口
servlet是一个接口，其实特别简单，在编程中接口等于契约，抽象出各种行为
```java
public interface Servlet {

    public void init(ServletConfig config) throws ServletException;

    public ServletConfig getServletConfig();

    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;

    public String getServletInfo();

    public void destroy();
}
```

# 访问
![1251492-20180507141819258-2036948492](https://gitee.com/caijingquan/imagebed/raw/master/1602319190_20200328003137040_1876145381.png)

Servlet容器默认是采用单实例多线程的方式处理多个请求的
1. 当web服务器启动的时候（或客户端发送请求到服务器时），Servlet就被加载并实例化(只存在一个Servlet实例)
2. 容器初始化化Servlet主要就是读取配置文件（例如tomcat,可以通过servlet.xml的设置线程池中线程数目，初始化线程池通过web.xml,初始化每个参数值等等
3. 当请求到达时，Servlet容器通过调度线程(Dispatchaer Thread) 调度它管理下线程池中等待执行的线程（Worker Thread）给请求者
4. 线程执行Servlet的service方法
5. 请求结束，放回线程池，等待被调用

（注意：避免使用实例变量（成员变量），因为如果存在成员变量，可能发生多线程同时访问该资源时，都来操作它，照成数据的不一致，因此产生线程安全问题）

从上面可以看出：
第一：Servlet单实例，减少了产生servlet的开销
第二：通过线程池来响应多个请求，提高了请求的响应时间
第三：Servlet容器并不关心到达的Servlet请求访问的是否是同一个Servlet还是另一个Servlet，直接分配给它一个新的线程；如果是同一个Servlet的多个请求，那么Servlet的service方法将在多线程中并发的执行
第四：每一个请求由ServletRequest对象来接受请求，由ServletResponse对象来响应该请求

# spring中servlet
任何Spring Web的entry point，都是servlet。

大名顶顶的spring框架已经风靡多时，一个事物的出现和流行都是会有原因的，那么为什么spring 框架会出现呢？原因就是为了简化java开发。

spring的核心就是通过`依赖注入`、`面向切面编程aop`、和`模版技术`，解耦业务与系统服务，消除重复代码。借助aop，可以将遍布应用的关注点（如事物和安全）从它们的应用对象中解耦出来。

# DispatcherServletAutoConfiguration

# Spring只实例了一个servlet：DispatcherServlet