vnote_backup_file_826537664 /home/cai/Documents/vnotebook/programs/language/java/framework/springboot/springboot启动.md
[TOC]

# 启动框架
1. 计时器开始计时
2. Headless模式赋值
3. 发送ApplicationStartingEvent
4. 配置环境模块
5. 发送ApplicationEnvironmentPreparedEvent
6. 发送ApplicationContextInitailizedEvent
7. 关联springboox组件与应用上下文对象
8. 初始化失败分析器
9. 创建上下文对象
10. 打印banner
11. 加载sources到context
12. 发送ApplicationPreparedEvent
13. 刷新上下文
14. 计时器停止计时
15. 发送ApplicationStartedEvent
16. 发送ApplicationReadyEvent
17. 调用框架启动扩展类

在构造方法中，将SpringApplication中的listeners全部注册到当前的注册表中。因为按照Spring的事件机制，事件的注册时机发生在refresh中，如果我们想监听SpringBoot的启动过程的话，则需要先将我们加载的监听器注册到注册表中。这样就和Spring的注册表在职能上没有关系了，这里面的注册表是EventPublishingRunListener的一个内部属性。

接下来就是实现SpringApplicationRunListener的七个方法了，它们会调用EventPublishingRunListener的内部属性initialMulticaster的广播方法，将相对应的事件对象广播出去，因为在构造方法中已经注册了监听器，那么这些事件对象就可以被相应的事件监听从而处理。


# 构造器内工作
```java
private List<ApplicationContextInitializer<?>> initializers;

private List<ApplicationListener<?>> listeners;

public SpringApplication(Class<?>... primarySources) {
    //ResourceLoader传null，传SpringBootApplication的class
	this(null, primarySources);
}
/**
 * Create a new {@link SpringApplication} instance. The application context will load
 * beans from the specified primary sources (see {@link SpringApplication class-level}
 * documentation for details. The instance can be customized before calling
 * {@link #run(String...)}.
 * @param resourceLoader the resource loader to use
 * @param primarySources the primary bean sources
 * @see #run(Class, String[])
 * @see #setSources(Set)
 */
@SuppressWarnings({ "unchecked", "rawtypes" })
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	this.resourceLoader = resourceLoader;
	Assert.notNull(primarySources, "PrimarySources must not be null");
	// LinkedHashSet
	this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
	// 两种类型：1.SERVLET,2.REACTIVE
	this.webApplicationType = WebApplicationType.deduceFromClasspath();
	// 将系统spring.factories中的初始化器实例化，并添加到initializers容器中
	setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
	// 将系统spring.factories中的监听器实例化，并添加到listeners容器中
	setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
	// 设置主类（main方法所在的类）
	this.mainApplicationClass = deduceMainApplicationClass();
}

private Class<?> deduceMainApplicationClass() {
	try {
		StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
		for (StackTraceElement stackTraceElement : stackTrace) {
			if ("main".equals(stackTraceElement.getMethodName())) {
				return Class.forName(stackTraceElement.getClassName());
			}
		}
	}
	catch (ClassNotFoundException ex) {
		// Swallow and continue
	}
	return null;
}
```

# run方法内工作
```java
/**
 * Run the Spring application, creating and refreshing a new
 * {@link ApplicationContext}.
 * @param args the application arguments (usually passed from a Java main method)
 * @return a running {@link ApplicationContext}
 */
public ConfigurableApplicationContext run(String... args) {
    // 计时器，记录项目启动时间
	StopWatch stopWatch = new StopWatch();
	stopWatch.start();
	// 要返回的springboot上下文封装类
	ConfigurableApplicationContext context = null;
	// 异常回调容器
	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
	// 配置java.awt.headless
	configureHeadlessProperty();
	// Spring启动时的Listeners封装
	SpringApplicationRunListeners listeners = getRunListeners(args);
	// 触发starting事件
	listeners.starting();
	try {
	    // 封装命令行参数
		ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
		// 运行环境的准备工作，在这步会触发Listener.environmentPrepared事件（当environment构建完成，ApplicationContext创建之前，该方法被调用）
		ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
		// 配置忽略BeanInfo
		configureIgnoreBeanInfo(environment);
		// Banner信息
		Banner printedBanner = printBanner(environment);
		// 创建应用上下文，并实例化了其三个属性：reader、scanner和beanFactory
		context = createApplicationContext();
		// 取异常报道器，即加载spring.factories中的SpringBootExceptionReporter实现类
		exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
				new Class[] { ConfigurableApplicationContext.class }, context);
	    /**
	    * 准备上下文
	    * 运行初始化器
	    * Listener.contextPrepared（当ApplicationContext构建完成时，该方法被调用）
	    * 是否设置单利模式
	    * 是否打印Banner
	    * lazyInitialization
	    * Listener.contextLoaded（在ApplicationContext完成加载，但没有被刷新前，该方法被调用）
	    **/
		prepareContext(context, environment, listeners, applicationArguments, printedBanner);
		refreshContext(context);
		afterRefresh(context, applicationArguments);
		stopWatch.stop();
		if (this.logStartupInfo) {
			new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
		}
		listeners.started(context);
		callRunners(context, applicationArguments);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, listeners);
		throw new IllegalStateException(ex);
	}

	try {
		listeners.running(context);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, null);
		throw new IllegalStateException(ex);
	}
	return context;
}
```

# prepareContext
```java
private void prepareContext(ConfigurableApplicationContext context,
        ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
        ApplicationArguments applicationArguments, Banner printedBanner) {
    // 设置上下文的environment
    context.setEnvironment(environment);
    // 应用上下文后处理
    postProcessApplicationContext(context);
    // 在context refresh之前，对其应用ApplicationContextInitializer
    applyInitializers(context);
    // 上下文准备（目前是空实现，可用于拓展）
    listeners.contextPrepared(context);
    // 打印启动日志和启动应用的Profile
    if (this.logStartupInfo) {
        logStartupInfo(context.getParent() == null);
        logStartupProfileInfo(context);
    }

    // Add boot specific singleton beans
    // 向beanFactory注册单例bean：命令行参数bean
    context.getBeanFactory().registerSingleton("springApplicationArguments",
            applicationArguments);
    if (printedBanner != null) {
        // 向beanFactory注册单例bean：banner bean
        context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
    }

    // Load the sources
    // 获取全部资源，其实就一个：SpringApplication的primarySources属性
    Set<Object> sources = getAllSources();
    // 断言资源是否为空
    Assert.notEmpty(sources, "Sources must not be empty");
    // 将bean加载到应用上下文中
    load(context, sources.toArray(new Object[0]));
    // 向上下文中添加ApplicationListener，并广播ApplicationPreparedEvent事件
    listeners.contextLoaded(context);
}
```





# 对比servlet与reactive
# main方法的独特性
# java.awt.headless

# 监听器事件
```java
public interface SpringApplicationRunListener {

    // 在run()方法开始执行时，该方法就立即被调用，可用于在初始化最早期时做一些工作
    void starting();
    // 当environment构建完成，ApplicationContext创建之前，该方法被调用
    void environmentPrepared(ConfigurableEnvironment environment);
    // 当ApplicationContext构建完成时，该方法被调用
    void contextPrepared(ConfigurableApplicationContext context);
    // 在ApplicationContext完成加载，但没有被刷新前，该方法被调用
    void contextLoaded(ConfigurableApplicationContext context);
    // 在ApplicationContext刷新并启动后，CommandLineRunners和ApplicationRunner未被调用前，该方法被调用
    void started(ConfigurableApplicationContext context);
    // 在run()方法执行完成前该方法被调用
    void running(ConfigurableApplicationContext context);
    // 当应用运行出错时该方法被调用
    void failed(ConfigurableApplicationContext context, Throwable exception);
}
```
