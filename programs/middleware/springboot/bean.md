[TOC]

# IOC概览（Inversion of Control）
其中最常见的方式叫做依赖注入（Dependency Injection，简称DI），还有一种方式叫“依赖查找”（Dependency Lookup）。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。

# IOC优点
+ 松耦合
+ 灵活性
+ 可维护

# bean配置方式
+ xml
+ 注解

# xml
## 无参构造
```java
@Getter
@Setter
public class Student {

    private String name;

    private Integer age;

    private List<String> classList;

    public Student(){}

    public Student(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}
```
配置文件xml
```xml
<bean id="student" class="com.ioc.xml.Student">
    <property name="classList">
        <list>
            <value>math</value>
            <value>english</value>
        </list>
    </property>
</bean>
```
## 有参构造
```java
@Getter
@Setter
public class Student {

    private String name;

    private Integer age;

    private List<String> classList;

    public Student(){}

    public Student(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}
```
配置文件xml
```xml
<bean id="student" class="com.ioc.xml.Student">
    <constructor-arg index="0" value="zhangsan"/>
    <constructor-arg index="1" value="13"/>
    <property name="classList">
        <list>
            <value>math</value>
            <value>english</value>
        </list>
    </property>
</bean>
```
## 静态工厂方法
```java
public class AnimalFactory {

    public static Animal getAnimal(String type) {
        if ("dog".equals(type)) {
            return new Dog();
        } else {
            return new Cat();
        }
    }

}
```
配置文件xml
```xml
<bean id="dog" factory-bean="animalFactory" factory-method="getAnimal">
    <constructor-arg value="dog"/>
</bean>

<bean id="helloService" class="com.ioc.xml.HelloService">
    <property name="student" ref="student"/>
    <property name="animal" ref="dog"/>
</bean>
```

## 实例工厂方法
```java
public class AnimalFactory {

    public Animal getAnimal(String type) {
        if ("dog".equals(type)) {
            return new Dog();
        } else {
            return new Cat();
        }
    }

}
```
配置文件xml
```xml
<bean name="animalFactory" class="com.ioc.xml.AnimalFactory"/>
<bean id="dog" class="com.ioc.xml.AnimalFactory" factory-method="getAnimal">
    <constructor-arg value="dog"/>
</bean>

<bean id="helloService" class="com.ioc.xml.HelloService">
    <property name="student" ref="student"/>
    <property name="animal" ref="dog"/>
</bean>
```

## 优点
+ 低耦合
+ 对象关系清晰
+ 集中管理

## 缺点
+ 配置繁琐
+ 开发效率低
+ 文件解析耗时

# 注解方式配置bean
## @Conponent声明
```java
@Component
```
## 配置类中使用@Bean
```java
@Configuration
public class BeanConfiguration{

    @Bean("dog")
    Animal getDog(){
        return new Dog();
    }
}
```
## 继承FactoryBean
```java
@Component
public class MyCat implements FactoryBean<Animal> {
    @Override
    public Animal getObject() throws Exception {
        return new Cat();
    }

    @Override
    public Class<?> getObjectType() {
        return Animal.class;
    }
}
```
## 继承BeanDefinitionRegistryPostProcessor
```java
@Component
public class MyBeanRegister implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition();
        rootBeanDefinition.setBeanClass(Monkey.class);
        registry.registerBeanDefinition("monkey", rootBeanDefinition);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

    }
}
```
## 继承ImportBeanDefinitionRegistrar
```java
public class MyBeanImport implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition();
        rootBeanDefinition.setBeanClass(Bird.class);
        registry.registerBeanDefinition("bird", rootBeanDefinition);
    }
}

```

## 优点
+ 使用简单
+ 开发效率高
+ 高内聚

## 缺点
+ 配置分散
+ 对象关系不清晰
+ 配置修改需要重新编译工程

# refresh方法解析
+ bean配置读取加载入口
+ spring框架启动流程

## refresh方法调用了13个子方法
![](_v_images/20200222071240161_1132654311.png)

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {//为什么要设置为同步快
		// Prepare this context for refreshing.
		// 刷新上下文之前做个准备：1.容器状态设置，2.初始化属性设置，3.检查必备属性是否存在
		prepareRefresh();

		// Tell the subclass to refresh the internal bean factory.
		// 1.设置beanFactory序列化id，2.获取beanFactory
		// ConfigurableListableBeanFactory与它实现的三个接口比较重要
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

		// Prepare the bean factory for use in this context.
		// 1.设置beanFactory一些属性，2.添加后置处理器，3.设置忽略的自动装配接口，4.注册一些组件
		prepareBeanFactory(beanFactory);

		try {
			// Allows post-processing of the bean factory in context subclasses.
			// web请求处理器与bean的一些配置，子类重写，如果是web环境，就会加入一些web参数
			postProcessBeanFactory(beanFactory);

			// Invoke factory processors registered as beans in the context.
			// 1.调用BeanDefinitionRegistryPostProcessor实现向容器内添加bean的定义
			// 2.调用BeanFactoryPostProcessor实现向容器内bean的定义添加属性
			invokeBeanFactoryPostProcessors(beanFactory);

			// Register bean processors that intercept bean creation.
			// 1.找到BeanPostProcessor的实现，2.排序后注册进容器内
			registerBeanPostProcessors(beanFactory);

			// Initialize message source for this context.
			// 初始化国际化相关的属性
			initMessageSource();

			// Initialize event multicaster for this context.
			// 初始化事件广播器
			initApplicationEventMulticaster();

			// Initialize other special beans in specific context subclasses.
			// 如果是web项目，会构建一个web容器：TOMCAT，Jetty
			onRefresh();

			// Check for listener beans and register them.
			// 1.添加容器内事件监听器至事件广播器中，2.派发早期事件
			registerListeners();

			// Instantiate all remaining (non-lazy-init) singletons.
			// 初始化所有剩下的单实例bean
			finishBeanFactoryInitialization(beanFactory);

			// Last step: publish corresponding event.
			// 1.初始化生命周期处理器，2.调用生命周期处理器onRefresh方法，3.发布ContextRefreshedEvent事件
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
DefaultListableBeanFactory
```java
@SuppressWarnings("serial")
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
		implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {

	@Override
	public void preInstantiateSingletons() throws BeansException {
		if (logger.isTraceEnabled()) {
			logger.trace("Pre-instantiating singletons in " + this);
		}

		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

		// Trigger initialization of all non-lazy singleton beans...
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				if (isFactoryBean(beanName)) {
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
					if (bean instanceof FactoryBean) {
						final FactoryBean<?> factory = (FactoryBean<?>) bean;
						boolean isEagerInit;
						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
							isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
											((SmartFactoryBean<?>) factory)::isEagerInit,
									getAccessControlContext());
						}
						else {
							isEagerInit = (factory instanceof SmartFactoryBean &&
									((SmartFactoryBean<?>) factory).isEagerInit());
						}
						if (isEagerInit) {
							getBean(beanName);
						}
					}
				}
				else {
					getBean(beanName);
				}
			}
		}

		// Trigger post-initialization callback for all applicable beans...
		for (String beanName : beanNames) {
			Object singletonInstance = getSingleton(beanName);
			if (singletonInstance instanceof SmartInitializingSingleton) {
				final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
				if (System.getSecurityManager() != null) {
					AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
						smartSingleton.afterSingletonsInstantiated();
						return null;
					}, getAccessControlContext());
				}
				else {
					smartSingleton.afterSingletonsInstantiated();
				}
			}
		}
	}

}

```
