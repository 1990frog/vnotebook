[TOC]

# 主函数
```java
@Override
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
		// Prepare this context for refreshing.
		// 为了刷新准备上下文环境
		prepareRefresh();

		// Tell the subclass to refresh the internal bean factory.
		// 告诉子类刷新内部bean工厂
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

		// Prepare the bean factory for use in this context.
		// 准备在这种情况下使用的bean工厂
		prepareBeanFactory(beanFactory);

		try {
			// Allows post-processing of the bean factory in context subclasses.
			postProcessBeanFactory(beanFactory);

			// Invoke factory processors registered as beans in the context.
			invokeBeanFactoryPostProcessors(beanFactory);

			// Register bean processors that intercept bean creation.
			registerBeanPostProcessors(beanFactory);

			// Initialize message source for this context.
			initMessageSource();

			// Initialize event multicaster for this context.
			initApplicationEventMulticaster();

			// Initialize other special beans in specific context subclasses.
			onRefresh();

			// Check for listener beans and register them.
			registerListeners();

			// Instantiate all remaining (non-lazy-init) singletons.
			finishBeanFactoryInitialization(beanFactory);

			// Last step: publish corresponding event.
			finishRefresh();
		}

		catch (BeansException ex) {
			if (logger.isWarnEnabled()) {
				logger.warn("Exception encountered during context initialization - " +
						"cancelling refresh attempt: " + ex);
			}

			// Destroy already created singletons to avoid dangling resources.
			destroyBeans();

			// Reset 'active' flag.
			cancelRefresh(ex);

			// Propagate exception to caller.
			throw ex;
		}

		finally {
			// Reset common introspection caches in Spring's core, since we
			// might not ever need metadata for singleton beans anymore...
			resetCommonCaches();
		}
	}
}
```

## prepareRefresh()
```java
/**
 * Prepare this context for refreshing, setting its startup date and
 * active flag as well as performing any initialization of property sources.
 * 准备刷新容器，设置启动时间和激活标志，以及执行初始化property sources
 */
protected void prepareRefresh() {
	// Switch to active.
	// 设置时间和状态
	this.startupDate = System.currentTimeMillis();
	this.closed.set(false);
	this.active.set(true);
    
    // 如果日志等级是debug就输出trace日志
	if (logger.isDebugEnabled()) {
		if (logger.isTraceEnabled()) {
			logger.trace("Refreshing " + this);
		}
		else {
			logger.debug("Refreshing " + getDisplayName());
		}
	}

	// Initialize any placeholder property sources in the context environment.
	// 在上下文环境中初始化所有placeholder property sources
	initPropertySources();

	// Validate that all properties marked as required are resolvable:
	// see ConfigurablePropertyResolver#setRequiredProperties
	getEnvironment().validateRequiredProperties();

	// Store pre-refresh ApplicationListeners...
	if (this.earlyApplicationListeners == null) {
		this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
	}
	else {
		// Reset local application listeners to pre-refresh state.
		this.applicationListeners.clear();
		this.applicationListeners.addAll(this.earlyApplicationListeners);
	}

	// Allow for the collection of early ApplicationEvents,
	// to be published once the multicaster is available...
	this.earlyApplicationEvents = new LinkedHashSet<>();
}
```
### initPropertySources
```java
public class GenericWebApplicationContext extends GenericApplicationContext
    implements ConfigurableWebApplicationContext, ThemeSource {
    protected void initPropertySources() {
        // 获取环境
        	ConfigurableEnvironment env = getEnvironment();
        	// 如果是web环境就初始化web对应的property sources
        	if (env instanceof ConfigurableWebEnvironment) {
        		((ConfigurableWebEnvironment) env).initPropertySources(this.servletContext, null);
        	}
    }
}
```
#### StandardServletEnvironment.initPropertySources
```java
public class StandardServletEnvironment extends StandardEnvironment implements ConfigurableWebEnvironment {

    @Override
	public void initPropertySources(@Nullable ServletContext servletContext, @Nullable ServletConfig servletConfig) {

		WebApplicationContextUtils.initServletPropertySources(getPropertySources(), servletContext, servletConfig);
	}
}
```
##### WebApplicationContextUtils.initServletPropertySources
```java
public abstract class WebApplicationContextUtils {
    public static void initServletPropertySources(MutablePropertySources sources,
    			@Nullable ServletContext servletContext, @Nullable ServletConfig servletConfig) {

        	Assert.notNull(sources, "'propertySources' must not be null");
        	String name = StandardServletEnvironment.SERVLET_CONTEXT_PROPERTY_SOURCE_NAME;
        	if (servletContext != null && sources.contains(name) && sources.get(name) instanceof StubPropertySource) {
        		sources.replace(name, new ServletContextPropertySource(name, servletContext));
        	}
        	name = StandardServletEnvironment.SERVLET_CONFIG_PROPERTY_SOURCE_NAME;
        	if (servletConfig != null && sources.contains(name) && sources.get(name) instanceof StubPropertySource) {
        		sources.replace(name, new ServletConfigPropertySource(name, servletConfig));
        	}
    }
}
```
## obtainFreshBeanFactory
```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {

    protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		return getBeanFactory();
	}
}
```
### GenericApplicationContext.refreshBeanFactory
```java
public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {

    @Override
	protected final void refreshBeanFactory() throws IllegalStateException {
		if (!this.refreshed.compareAndSet(false, true)) {
			throw new IllegalStateException(
					"GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
		}
		this.beanFactory.setSerializationId(getId());
	}

    /**
	 * Specify an id for serialization purposes, allowing this BeanFactory to be
	 * deserialized from this id back into the BeanFactory object, if needed.
	 * 为序列化的目标准备个id，允许将此BeanFactory从此ID反序列化回BeanFactory对象（如果需要）
	 */
	public void setSerializationId(@Nullable String serializationId) {
		if (serializationId != null) {
			serializableFactories.put(serializationId, new WeakReference<>(this));
		}
		else if (this.serializationId != null) {
			serializableFactories.remove(this.serializationId);
		}
		this.serializationId = serializationId;
	}

}
```
## prepareBeanFactory
```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {

    /**
	 * Configure the factory's standard context characteristics,
	 * such as the context's ClassLoader and post-processors.
	 * @param beanFactory the BeanFactory to configure
	 */
	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// Tell the internal bean factory to use the context's class loader etc.
		// 设置类加载器
		beanFactory.setBeanClassLoader(getClassLoader());
		// 设置bean的表达式语言处理器，默认可以使用#{bean.xxx}的形式来调用相关的属性
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
		// 为beanFactory增加一个默认的propertyEditor，这个主要是对bean的属性等设置管理一个工具
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		// Configure the bean factory with context callbacks.
		// 添加BeanPostProcessor，bean前置处理器
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		// 忽略下面几种类型
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		/**
		* 看不懂
		*/
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// Register early post-processor for detecting inner beans as ApplicationListeners.
		// 注册早期的后处理器以将内部bean检测为ApplicationListeners
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

		// Detect a LoadTimeWeaver and prepare for weaving, if found.
		// 增加了对AspectJ的支持
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// Register default environment beans.
		// 添加默认的系统环境bean
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
	}
}
```
## postProcessBeanFactory
```java
public class AnnotationConfigServletWebServerApplicationContext extends ServletWebServerApplicationContext
		implements AnnotationConfigRegistry {
    @Override
	protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		super.postProcessBeanFactory(beanFactory);
		if (this.basePackages != null && this.basePackages.length > 0) {
			this.scanner.scan(this.basePackages);
		}
		if (!this.annotatedClasses.isEmpty()) {
			this.reader.register(ClassUtils.toClassArray(this.annotatedClasses));
		}
	}
}
```