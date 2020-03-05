[TOC]

# conditional注解解析
## 含义
基于条件的注解
## 作用
根据是否满足某一个特定条件来决定是否创建某个特定的Bean
## 意义
springboot实现自动配置的关键基础能力

## 常见的conditional注解
+ #ConditionalOnBean：存在某个bean才会生效
+ #ConditionalOnMissingBean：不存在某个bean才会生效
+ #ConditionalOnClass：某个class是否存在才会生效
+ #ConditionalOnMissingClass：不存在某个class才会生效
+ #ConditionalOnWebApplication：判断当前是web环境
+ #ConditionalOnNotWebApplication：判断当前非web环境
+ #ConditionalOnProperty：判断当前是否包含特定属性
+ #ConditionalOnJava：判断当前是java的某个版本

# starter介绍
## 简介
可插拔插件
## 与jar区别
starter能实现自动配置
## 作用
大幅提升开发效率

# 常用starter

|                 名称                  |                            描述                             |
| ------------------------------------ | ---------------------------------------------------------- |
| spring-boot-starter-thymeleaf          | 使MVC web application支持Thymeleaf                            |
| spring-boot-starter-mail              | 使用JavaMail，Spring email发送支持                             |
| spring-boot-starter-data-redis         | 通过spring data redis、jedis client使用Redis键值存储数据库       |
| spring-boot-starter-web               | 构建Web，包含RESTful风格框架SpringMVC和默认的嵌入式容器Tomcat       |
| spring-boot-starter-activemq           | 为JMS使用ApacheActiveMQ                                      |
| spring-boot-starter-data-elasticsearch | 使用Elasticsearch、analytics engine、Spring Data Elasticsearch |
| spring-boot-starter-aop               | 通过Spring AOP、AspectJ面向切面编程                             |
| spring-boot-starter-security           | 使用Spring Security                                          |
| spring-boot-starter-data-jpa           | 通过Hibernate使用SpringData JPA                               |
| spring-boot-starter                   | Core starter，包括自动配置支持，logging and YAML                |
| spring-boot-starter-freemarker         | 使MVC Web Application支持Freemarker                           |
| spring-boot-starter-batch              | 使用Spring Batch                                             |
| spring-boot-starter-solr              | 通过Spring Data Solr使用Apache Solr                           |
| spring-boot-starter-mongodb            | 使用MongoDB文件存储数据库、SpringData MongoDB                   |


# 动手搭建starter


Condition接口

# 新建starter步骤
1. 新建Springboot项目
2. 引入spring-boot-autoconfigure
3. 编写属性源及自动配置类
4. 在spring.factories中添加自动配置类实现
5. maven打包

# starter自动配置类导入
1. 启动类上@SpringBootApplication
2. 引入AutoConfigurationImportSelector
3. ConfigurationClassParser中处理
4. 获取spring.factories中EnableAutoConfiguration实现

# starter自动配置类过滤
1. @ConditionalOnProperty
2. OnPropertyCondition
3. getMatchOutcome
4. 遍历注解属性集判断environment中是否含有并值一致
5. 返回对比结果

# starter原理