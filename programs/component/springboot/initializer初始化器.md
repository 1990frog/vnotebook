[TOC]

# 用途
ApplicationContextInitializer是Spring在执行ConfigurableApplicationContext.refresh() 方法对应用上下文进行刷新之前调用的一个回调接口，用来完成对Spring应用上下文个性化的初始化工作

+ 类名：ApplicationContextInitializer
+ 介绍：Spring容器刷新之前执行的一个回调函数
+ 作用：向SpringBoot容器注册属性
+ 使用方式：继承接口自定义实现

# 三种实现
## 实现方式一:factories
+ 实现ApplicationContextInitializer接口
+ spring.factories内填写接口实现
+ key值为org.springframework.context.ApplicationContextInitializer

```java
@Order(1)
public class FirstInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        Map<String, Object> map = new HashMap<>();
        map.put("key1", "value1");
        MapPropertySource mapPropertySource = new MapPropertySource("firstInitializer", map);
        environment.getPropertySources().addLast(mapPropertySource);
        System.out.println("run firstInitializer");
    }
}
```
resources/META-INF/spring.factories
```
org.springframework.context.ApplicationContextInitializer=com.initializer.FirstInitializer
```

## 实现方式二:SpringApplication方法注入
+ 实现ApplicationContextInitializer接口
+ SpringApplication类初始化后设置进去

```java
@Order(2)
public class SecondInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        Map<String, Object> map = new HashMap<>();
        map.put("key2", "value2");
        MapPropertySource mapPropertySource = new MapPropertySource("secondInitializer", map);
        environment.getPropertySources().addLast(mapPropertySource);
        System.out.println("run secondInitializer");
    }
}
```
SpringbootApplication
```java
@SpringBootApplication
public class SpringbootApplication {

    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(SpringbootApplication.class);
        springApplication.addInitializers(new SecondInitializer());
        springApplication.run(args);
    }

}

```
## 实现方式三：properties
+ 实现ApplicationContextInitializer接口
+ application.properties内填写接口实现
+ key值为context.initializer.classes

```java
@Order(3)
public class ThirdInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        Map<String, Object> map = new HashMap<>();
        map.put("key3", "value3");
        MapPropertySource mapPropertySource = new MapPropertySource("thirdInitializer", map);
        environment.getPropertySources().addLast(mapPropertySource);
        System.out.println("run thirdInitializer");
    }
}
```
application.properties
```
context.initializer.classes=com.initializer.ThirdInitializer
```

## 三种方法总结
+ 都要实现ApplicationContextInitializer接口
+ Order值越小越先执行
+ application.properties中定义的优先于其它方式


# ApplicationContextInitializer接口
+ 用于在spring容器刷新之前初始化Spring ConfigurableApplicationContext的回调接口。（简短说就是在容器刷新之前调用该类的 initialize方法。并将 ConfigurableApplicationContext类的实例传递给该方法）
+ 通常用于需要对应用程序上下文进行编程初始化的web应用程序中。例如，根据上下文环境注册属性源或激活配置文件等。
+ 可排序的（实现Ordered接口，或者添加@Order注解）

# springboot.jar内置初始化器(spring.factories)
```
@author Phillip Webb

# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.rsocket.context.RSocketPortInfoApplicationContextInitializer,\
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer
```
## ConfigurationWarningsApplicationContextInitializer
在日志中警告一般配置错误
## ContextIdApplicationContextInitializer
设置Spring应用上下文的id
## DelegatingApplicationContextInitializer
用来初始化核心配置文件*.properties内设置的初始化器（这也是为啥properties的order优先级最高的原因）
部分源码：
```java
public class DelegatingApplicationContextInitializer
		implements ApplicationContextInitializer<ConfigurableApplicationContext>, Ordered {

	// NOTE: Similar to org.springframework.web.context.ContextLoader

	private static final String PROPERTY_NAME = "context.initializer.classes";

	private int order = 0;

	@Override
	public void initialize(ConfigurableApplicationContext context) {
	    // 上下文
		ConfigurableEnvironment environment = context.getEnvironment();
		// 获取properties配置的初始化器
		List<Class<?>> initializerClasses = getInitializerClasses(environment);
		if (!initializerClasses.isEmpty()) {
		    // 运行
			applyInitializerClasses(context, initializerClasses);
		}
	}

    // 获取properties配置的初始化器
	private List<Class<?>> getInitializerClasses(ConfigurableEnvironment env) {
		String classNames = env.getProperty(PROPERTY_NAME);
		List<Class<?>> classes = new ArrayList<>();
		if (StringUtils.hasLength(classNames)) {
			for (String className : StringUtils.tokenizeToStringArray(classNames, ",")) {
				classes.add(getInitializerClass(className));
			}
		}
		return classes;
	}

    // 断言
	private Class<?> getInitializerClass(String className) throws LinkageError {
		try {
			Class<?> initializerClass = ClassUtils.forName(className, ClassUtils.getDefaultClassLoader());
			Assert.isAssignable(ApplicationContextInitializer.class, initializerClass);
			return initializerClass;
		}
		catch (ClassNotFoundException ex) {
			throw new ApplicationContextException("Failed to load context initializer class [" + className + "]", ex);
		}
	}

	private void applyInitializerClasses(ConfigurableApplicationContext context, List<Class<?>> initializerClasses) {
		Class<?> contextClass = context.getClass();
		List<ApplicationContextInitializer<?>> initializers = new ArrayList<>();
		for (Class<?> initializerClass : initializerClasses) {
			initializers.add(instantiateInitializer(contextClass, initializerClass));
		}
		applyInitializers(context, initializers);
	}

	private ApplicationContextInitializer<?> instantiateInitializer(Class<?> contextClass, Class<?> initializerClass) {
		Class<?> requireContextClass = GenericTypeResolver.resolveTypeArgument(initializerClass,
				ApplicationContextInitializer.class);
		Assert.isAssignable(requireContextClass, contextClass,
				String.format(
						"Could not add context initializer [%s] as its generic parameter [%s] is not assignable "
								+ "from the type of application context used by this context loader [%s]: ",
						initializerClass.getName(), requireContextClass.getName(), contextClass.getName()));
		return (ApplicationContextInitializer<?>) BeanUtils.instantiateClass(initializerClass);
	}

	@SuppressWarnings({ "unchecked", "rawtypes" })
	private void applyInitializers(ConfigurableApplicationContext context,
			List<ApplicationContextInitializer<?>> initializers) {
		initializers.sort(new AnnotationAwareOrderComparator());
		for (ApplicationContextInitializer initializer : initializers) {
			initializer.initialize(context);
		}
	}

	public void setOrder(int order) {
		this.order = order;
	}

	@Override
	public int getOrder() {
		return this.order;
	}

}
```
## RSocketPortInfoApplicationContextInitializer
## ServerPortInfoApplicationContextInitializer
设置端口

# SpringFactoriesLoader:获取要创建的类的清单（spring.factories）工厂
+ 框架内部使用的通用工厂加载机制
+ 从classpath下多个jar包特定的位置读取文件并初始化类
+ 文件内容必须是key_value形式，即properties类型
`<pre class="code">example.MyService=example.MyServiceImpl1,example.MyServiceImpl2</pre>`
+ key是全限定名（抽象类|接口）、value是实现，多个实现用逗号分隔

# 总结
+ 定义在spring.factories文件中被SpringFactoriesLoader发现注册
+ SpringApplication初始化完毕后手动添加
+ 定义成环境变量被DelegatingApplicationContextInitializer发现注册
