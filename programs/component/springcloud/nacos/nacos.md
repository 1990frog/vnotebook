[TOC]

# nacos能做什么
## 服务注册发现和服务健康检测
Nacos支持基于DNS和基于RPC的服务发现，服务端可以通过SDK或者Api进行服务注册，相应的服务消费者可以使用DNS或者Http查找的方式获取服务列表。Nacos同时提供对服务的实时健康检查，阻止想不健康的主机或服务发送请求，与Eureka类似Nacos也有友好的控制台界面。
## 动态配置服务
接触过SpringCloud应该对config有所了解，那么配置中心也就很好理解，Nacos支持动态的配置管理，将服务的配置信息分环境分类别外部管理，并且支持热更新。不过与Config不同Nacos的配置信息存储与数据库中，支持配置信息的监听和版本回滚。
## 动态DNS服务
支持权重路由，更容易地实现中间层负载均衡、更灵活的路由策略、流量控制以及数据中心内网的简单DNS解析服务。不过这个特性目前版本还不支持
## 服务及元数据管理
Nacos 能让您从微服务平台建设的视角管理数据中心的所有服务及元数据，包括管理服务的描述、生命周期、服务的静态依赖分析、服务的健康状态、服务的流量管理、路由及安全策略、服务的 SLA 以及最首要的 metrics 统计数据。
## 总结
Nacos从官方的介绍上看，就像是SpringCloud中Eureka+Config+Bus+Git+MQ的一个结合体，当然也不能完全这么理解。Nacos是脱胎于阿里内部的ConfigServer，而ConfigServer早在3.0版本就解决了Eureka在1.0版本留下的隐患，所以从技术的更新和迭代角度来看，稳定版本的Nacos将更适合做为微服务体系中的服务注册发现组件，当然了他也不单单只是注册和发现。更多的特性和功能，不如一起搭建试试吧。

# 文档
[nacos文档](https://nacos.io/zh-cn/docs/what-is-nacos.html)
# 什么是nacos
服务（Service）是 Nacos 世界的一等公民。Nacos 支持几乎所有主流类型的“服务”的发现、配置和管理

![16751762-9b9ead13708a21b3](_v_images/20200305001859644_1062184217.png)
+ Namespace:实现隔离，默认public
+ Group:不同服务可以分到一个组，默认DEFAULT_GROUP
+ Service:微服务
+ Cluster:对指定微服务的一个虚拟划分，默认DEFAULT
+ Instance:微服务实例

# 服务提供者与服务消费者
|   名词    |                   定义                    |
| -------- | ---------------------------------------- |
| 服务提供者 | 服务的被调用方（即：为其他微服务提供接口的微服务） |
| 服务消费者 | 服务的调用方（即：调用其他微服务接口的微服务）    |

# Nacos
+ 服务发现组件
+ 配置服务器

# nacos控制台
```
控制台：
http://localhost:8848/nacos
默认用户名
amdin:nacos
pwd:nacos
```

# 下载
https://github.com/alibaba/nacos/tags

# 文档
https://nacos.io/zh-cn/docs/quick-start.html

# 启动命令
```
sh startup.sh -m standalone
```

# SpringCloud注册
## Nacos依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

## 配置
早期版本需要添加注解`@EnableDiscoveryClient`
```yml
cloud:
    nacos:
        discovery:
            # 指定nacos server地址
            server-addr:localhost：8848
application：
    # 服务名称尽量用-，不要用_，不要用特殊字符
    name:user-center
```

## DiscoveryClient
```java
@Slf4j
@RestController
public class TestController {
    @Autowired
    private DiscoveryClient discoveryClient;

    public void discover(){
        List<ServiceInstance> instances = discoveryClient.getInstances("portname");
        String targetURL = instances.stream
        .map(instance -> instance.getUri().toString() + url)
        .findFirst()
        .orElseThrow(()->new IllegalArgumentException("当前没有实例"))；
    }
}
```

# Namespace
类似linux资源隔离，相同namespace下可以互相调用，不能跨namespace互调
# Group

# Service

# Cluster

# Instance

