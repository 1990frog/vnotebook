[TOC]

# 具体的日志功能实现
+ JUL
+ Log4j
+ Logback
+ Log4j2

# 日志实现的抽象层（门面与实现的区别）
+ JCL
+ SLF4J（最优）

# 日志发展历程
+ JDK1.3及以前，通过System.(out|err).println打印，存在巨大的缺陷
+ log4j作者推出slf4j，功能完善兼容性好，成为业界主流
+ log4j作者在推出log4j后进行新的改进思考推出logback
+ log4j2是apache对log4j的重大升级，修复已知缺陷，极大提升性能
+ 最佳组合：slf4j+logback（springboot使用）

# demo
```
private Logger logger = LoggerFactory.getLogger(xxx.class);
等价于
@Slf4j
```
# 日志实现寻址
+ LoggerFactory.getLogger
+ findPossibleStaticLoggerBinderPathSet
+ 获取StaticLoggerBinder所在jar包途径
+ 若存在多个日志实现框架打印提示及选择
+ 使用StaticLoggerBinder获得日志工厂再嘚实现

# springboot配置日志
spring-boot-starter-logging
+ logback-classic
+ log4j-to-slf4j
+ jul-to-slf4j

![](_v_images/20200301143232820_1268825950.png)

# 日志使用
Logger logger = LoggerFactory.getLogger(xx.clas)
logger.level()
官方推荐使用占位符的方式：
logger.debgug("xyzP {} is {}",i,j)

# 日志级别
+ ERROR
+ WARN
+ INFO
+ DEBUG
+ TRACE

# 日志配置
```
<configuration scan="true" scanPeriod="60 seconds" debug="false"/>
```
+ scan：当设置为true时，配置文件若发生改变，将会重新加载
+ scanPeriod：扫描时间间隔，若没给出时间单位默认为毫秒
+ debug：若设置为true，将打印出logback内部日志信息

# configuration子节点
+ contextName：上下文名称
+ property：属性配置
+ appender：格式化日志输出
+ root：全局日志输出设置
+ logger：具体包或子类输出设置

# 常用pattern介绍
+ logger{length}:输出日志的logger名，可有一个整形参数，功能是缩短logger名
+ contextName|cn：上下文名称
+ date{pattern}：输出日志的打印时间，模式语法与java.text.SimpleDateFormat兼容
+ p/le/level：日志级别
+ M/method：输出日志的方法名
+ r/relative：从程序启动到创建日志记录的时间
+ m/msg/message：程序提供的信息
+ n：换行符

# root&logger
+ root全局
+ logger是指定

# 日志实战
结合切面

aopstarter
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

@AfterThrowing(pointcut = "@within()")
@AfterThrowing(pointcut = "within()")



首先，官方推荐使用的xml名字的格式为：logback-spring.xml而不是logback.xml，至于为什么，因为带spring后缀的可以使用<springProfile>这个标签。

在resource下创建logback-spring.xml文件

# SiftingAppender

![](_v_images/20200301205259787_1316867442.png)