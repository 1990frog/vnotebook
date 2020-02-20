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