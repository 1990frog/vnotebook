[TOC]

@EventListener注解方式实现


![](_v_images/20200215092541645_2115349287.png)

# 监听器模式要素
+ Event事件
+ Listener监听器
+ Multicaster广播器（内含触发机制）

# 代码实现
Event
```java
public interface Event {}

public class PlayEvent implements Event{}

public class WorkEvent implements Event {}
```
Listener
```java
public interface Listener {
    public void action(Event event);
}
public class PlayListener implements Listener {
    @Override
    public void action(Event event) {
        if(event instanceof PlayEvent)
            System.out.println("this is play listener");
    }
}
public class WorkListener implements Listener {
    @Override
    public void action(Event event) {
        if(event instanceof WorkEvent)
            System.out.println("this is work listener");
    }
}
```
Multicaster
```java
public class Multicaster {

    private List<Listener> listenerList;

    public Multicaster(){
        listenerList = new ArrayList<>();
    }

    public void addLisener(Listener listener){
        this.listenerList.add(listener);
    }

    public void happen(Event event){
        listenerList.forEach(listener -> listener.action(event));
    }

}
```
App
```java
public class App {

    public static void main(String[] args) {
        Multicaster multicaster = new Multicaster();
        multicaster.addLisener(new PlayListener());
        multicaster.addLisener(new WorkListener());
        multicaster.happen(new PlayEvent());
    }
}

```

# SpringBoot监听器事件
1. ApplicationContextInitializedEvent
2. ApplicationEnvironmentPreparedEvent：springboot框架环境准备完毕，context还没有创建完成，bean也没有完成创建
3. ApplicationFailedEvent：springboot框架启动失败
4. ApplicationPreparedEvent：springboot框架启动，context创建完，bean没有创建
5. ApplicationReadyEvent
6. ApplicationStartedEvent：springboot框架启动完成，context和bean都已创建完毕
7. ApplicationStartingEvent：springboot框架开始启动的事件

# 自定义springboot事件
event
```java
public class DiySpringBootEvent extends ApplicationEvent {
    /**
     * Create a new {@code ApplicationEvent}.
     *
     * @param source the object on which the event initially occurred or with
     *               which the event is associated (never {@code null})
     */
    public DiySpringBootEvent(Object source) {
        super(source);
    }
}
```
listener
```java
public class DiySpringBootListener implements ApplicationListener<DiySpringBootEvent> {
    @Override
    public void onApplicationEvent(DiySpringBootEvent event) {
        System.out.println("this is my event");
    }
}
```
发布事件
```java
@SpringBootApplication
@PropertySource({"demo.properties"})
public class SpringbootApplication {

    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(SpringbootApplication.class);
        // 自定义监听器
        springApplication.addListeners(new DiySpringBootListener());
        ConfigurableApplicationContext context = springApplication.run(args);
        context.publishEvent(new DiySpringBootEvent(new Object()));
    }

}
```
# ApplicationListener
SpringBoot监听器实现接口`ApplicationListener<E extends ApplicationEvent> extends EventListener`

# SpringBoot监听器实现
## 实现方式一
+ 实现ApplicationListener接口
+ spring.factories内填写接口实现
+ key值为org.springframework.context.ApplicationListener
```java
@Order(1)
public class FirstListener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("hello first");
    }
}
```
配置文件
```
org.springframework.context.ApplicationListener=com.listener.FirstListener
```
## 实现方式一
+ 实现ApplicationListener接口
+ SpringApplication类初始后设置进去
```java
@Order(2)
public class SecondListener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("hello second");
    }
}
```
spring启动类
```java
@SpringBootApplication
// @PropertySource({"demo.properties"})
public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(SpringbootApplication.class);
        // 设置监听器
        springApplication.addListeners(new SecondListener());
        // 运行
        springApplication.run(args);
    }
```
## 实现方式三
+ 实现ApplicationListener接口
+ application.properties内填写接口实现，key值为context.listener.classes
```java
@Order(3)
public class ThirdListener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("hello third");
    }
}
```
配置文件
```
context.listener.classes=com.listener.ThirdListener
```
## 实现方式四：优势（监听多个事件）
+ 实现SmartApplicationListener接口
+ 重写supportsEventType方法
+ 同前三种注入方式注入框架
```java
@Order(4)
public class FourthListener implements SmartApplicationListener {
    @Override
    public boolean supportsEventType(Class<? extends ApplicationEvent> eventType) {
        return ApplicationStartedEvent.class.isAssignableFrom(eventType) || ApplicationPreparedEvent.class.isAssignableFrom(eventType);
    }

    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("hello fourth");
    }
}
```
配置文件，或Application注入
## 总结
+ 实现ApplicationListener接口针对单一事件监听
+ 实现SmartApplicationListener接口针对多种事件监听
+ Order值越小越先执行
+ application.properties中定义的优先于其他方式


