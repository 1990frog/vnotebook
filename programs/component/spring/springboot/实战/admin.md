[TOC]

# 创建项目
spring initial选择springboot-admin-server

如果是springcloud alibaba项目引入nacos
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

加注解
@EnableAdminServer

配置
```yml
server:
  port: 8020
spring:
  application:
    name: spring-boot-admin
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
```

被监控项目：
```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"  #暴露所有接口
  endpoint:
    shutdown.enabled: true  #允许shutdown
    health.show-details: always #展示详细的health信息
    jolokia:
      enabled: false #允许 jolokia
      config.debug: true
```

# jvm监控
SpringBootActuator：metrics、heapdump、threaddump
java自带的jvm监控工具：jconsole、jvisualvm

jconsole：
打开个终端，输入jconsole

jvisualvm：最强的了

# gc日志/线程dump日志/堆dump可视化分析

添加参数打印gc日志
`-Xmx5m -XX:PrintGCDetails -Xloggc:gc.log`

可视化分析工具：
GCEasy：https://gceasy.io/
FastThread:https//fastthread.io/
HeapHero:https://heaphero.io/

291751