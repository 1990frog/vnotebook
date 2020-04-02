[TOC]

# 过滤器
是在java web中将你传入的request、response提前过滤掉一些信息，或者提前设置一些参数。
然后再传入Servlet或Struts2的 action进行业务逻辑处理。
比如过滤掉非法url（不是login.do的地址请求，如果用户没有登陆都过滤掉）
或者在传入Servlet或Struts2的action前统一设置字符集，或者去除掉一些非法字符。

# 拦截器
是面向切面编程（AOP，Aspect Oriented Program）的。
就是在你的Service或者一个方法前调用一个方法，或者在方法后调用一个方法。
比如动态代理就是拦截器的简单实现，在你调用方法前打印出字符串（或者做其它业务逻辑的操作），也可以在你调用方法后打印出字符串，甚至在你抛出异常的时候做业务逻辑的操作。

# 通俗理解
+ 过滤器（Filter）：当你有一堆东西的时候，你只希望选择符合你要求的某一些东西。定义这些要求的工具，就是过滤器。（理解：就是一堆字母中取一个B）
+ 拦截器（Interceptor）：在一个流程正在进行的时候，你希望干预它的进展，甚至终止它进行，这是拦截器做的事情。（理解：就是一堆字母中，干预它，通过验证的少点，顺便干点别的东西）

# 主要区别
+ 拦截器是基于Java的反射机制的，而过滤器是基于函数回调。
+ 拦截器不依赖于servlet容器，过滤器依赖于servlet容器。
+ 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。
+ 拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。
+ 在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次
+ 拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。

# 本质区别
从灵活性上说拦截器功能更强大些，Filter能做的事情它都能做，而且可以在请求前，请求后执行，比较灵活。
Filter主要是针对URL地址做一个编码的事情、过滤掉没用的参数、安全校验（比较泛的，比如登录不登录之类），太细的活儿还是建议用interceptor。
具体还得根据不同情况选择合适的。

|             |                filter                |                  interceptor                   |
| ----------- | ------------------------------------ | --------------------------------------------- |
| 多个的执行顺序 | 根据filter mapping配置的先后顺序         | 按照配置的顺序，但是可以通过order控制顺序             |
| 规范         | 在servlet规范中定义的，是servlet容器支持的 | spring容器内的，是spring框架支持的                 |
| 使用范围      | 只能用于web程序中                       | 既可以用于web程序，也可以用于Application、Swing程序中 |
| 深度         | Filter在只在servlet前后起作用            | 拦截器能够深入到方法前后、异常抛出前后等              |

# 执行顺序
过滤前-----拦截前-----Action处理-----拦截后-----过滤后。
个人认为过滤是一个横向的过程，首先把客户端提交的内容进行过滤（例如未登录用户不能访问内部页面的处理）；过滤通过后，拦截器将进行用户提交数据的验证，做一些前期的数据处理；接着把处理后的数据发给对应的Action，Action处理完成返回后，拦截器还可以做些其他事情，再向上返回到过滤器的后续操作。

![5c07e8c100012e8205350457](_v_images/20200328131513843_406150175.jpeg)

过滤器（Filter）是在请求进入容器后，但还未进入Servlet之前进行预处理的。
请求结束返回也是，是在Servlet处理完后，返回给前端之前。
所以过滤器（Filter）的`doFilter(ServletRequest request, ServletResponse response, FilterChain chain)`的入参是ServletRequest ，而不是httpservletrequest。
因为过滤器是在httpservlet之前。


Filter跟Servlet一样都是由服务器负责创建和销毁的。
在web应用程序启动时，服务器会根据应用程序的`web.xml`文件中的配置信息调用`public void init(FilterConfig filterConfig) throws ServletException`方法来初始化Filter；
在web应用程序被移除或者是服务器关闭时，会调用`public void destroy()`来销毁Filter。
在一个应用程序中一个Filter只会被创建和销毁一次。
在初始化之后，Filter中声明了`public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException`, `ServletException`方法，用来实现一些需要在拦截完成之后的业务逻辑。
注意到上面的`doFilter()`方法的参数中，有`FilterChain chain`这个参数，它是传递过来的拦截链对象，里面包含了用户定义的一系列的拦截器，这些拦截器根据其在web.xml中定义的顺序依次被执行。
当用户的信息验证通过或者当前拦截器不起作用时，我们可以执行`chain.doFilter(request, response)`; 方法来跳过当前拦截器来执行拦截器链中的下一个拦截器。
`chain.doFilter(request, response)`这个方法的调用作为分水岭。事实上调用`Servle`t的`doService()`方法是在`chain.doFilter(request, response)`这个方法中进行的。

# 拦截器与过滤器使用场景
SpringMVC的处理器拦截器类似于Servlet开发中的过滤器Filter，用于对处理器进行预处理和后处理：
1. 日志记录：记录请求信息的日志，以便进行信息监控、信息统计、计算PV（Page View）等。
2. 权限检查：如登录检测，进入处理器检测检测是否登录，如果没有直接返回到登录页面；
3. 性能监控：有时候系统在某段时间莫名其妙的慢，可以通过拦截器在进入处理器之前记录开始时间，在处理完后记录结束时间，从而得到该请求的处理时间（如果有反向代理，如apache可以自动记录）；
4. 通用行为：读取cookie得到用户信息并将用户对象放入请求，从而方便后续流程使用，还有如提取Locale、Theme信息等，只要是多个处理器都需要的即可使用拦截器实现。
5. OpenSessionInView：如hibernate，在进入处理器打开Session，在完成后关闭Session。

# 补充说明
Spring的拦截器与Servlet的Filter有相似之处，比如二者都是AOP编程思想的体现，都能实现权限检查、日志记录等。

不同的是：
+ 使用范围不同：Filter是Servlet规范规定的，只能用于Web程序中。而拦截器既可以用于Web程序，也可以用于Application、Swing程序中。
+ 规范不同：Filter是在Servlet规范中定义的，是Servlet容器支持的。而拦截器是在Spring容器内的，是Spring框架支持的。
+ 使用的资源不同：同其他的代码块一样，拦截器也是一个Spring的组件，归Spring管理，配置在Spring文件中，因此能使用Spring里的任何资源、对象，例如Service对象、数据源、事务管理等，通过IOC注入到拦截器即可；而Filter则不能。
+ 深度不同：Filter在只在Servlet前后起作用。而拦截器能够深入到方法前后、异常抛出前后等，因此拦截器的使用具有更大的弹性。所以在Spring构架的程序中，要优先使用拦截器。

实际上Filter和Servlet极其相似，区别只是Filter不能直接对用户生成响应。实际上Filter里doFilter()方法里的代码就是从多个Servlet的service()方法里抽取的通用代码，通过使用Filter可以实现更好的复用。
+ Filter是一个可以复用的代码片段，可以用来转换Http请求、响应和头信息。Filter不像Servlet，它不能产生一个请求或者响应，它只是修改对某一资源的请求，或者修改从某一资源的响应。
+ SR中说明的是，按照多个匹配的Filter，是按照其在web.xml中配置的顺序来执行的。所以这也就是，把自己的Filter或者其他的Filter（比如UrlRewrite的Filter）放在Struts2的 DispatcherFilter的前面的原因。因为它们需要在请求被Struts2框架处理之前，做一些前置的工作。
+ 当Filter被调用，并且进入了Struts2的DispatcherFilter中后，Struts2会按照在Action中配置的Interceptor Stack中的Interceptor的顺序，来调用Interceptor。


# 拦截器与aop区别