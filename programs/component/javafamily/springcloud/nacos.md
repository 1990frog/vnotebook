[TOC]

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



