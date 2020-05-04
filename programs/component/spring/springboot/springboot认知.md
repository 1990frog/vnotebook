[TOC]

# Springboot四大核心
+ 自动装配-Auto Configuration
+ 起步依赖-Starter Dependency
+ 命令行界面-Spring Boot CLI
+ Actuator

# 了解自动装配
自动装配
+ 基于添加jar依赖自动对SpringBoot应用程序进行配置
+ spring-boot-autoconfiguration

开启自动装配配置
+ @EnableAutoConfiguration
    + exclude=Class<?>[]
+ @SpringBootApplication

# 自动配置的实现原理
@EnableAutoConfiguration
+ AutoConfigurationImportSelector
+ META-INF/spring.factories
    + org.springframwork.boot.autoconfiguration.EnableAutoConfiguration

# 自动装配条件
Conditional


# 自动配置的执行顺序
+ @AutoConfigurationBefore
+ @AutoConfigurationAfter
+ @AutoConfigureOrder

