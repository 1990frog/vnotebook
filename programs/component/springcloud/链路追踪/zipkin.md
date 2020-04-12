[TOC]

zipkin是twitter开源的分布式跟踪系统，主要用来收集系统的时序数据，从而追踪系统的调用问题

# 下载最新的zipkin
官方下载工具下载最新版
```
$ curl -sSL https://zipkin.io/quickstart.sh | bash -s
```
# 启动
```
$ java -jar zinpin.jar
```
# 配置
配置zipkin就不需要配置sleuth
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```
yml
```yml
spring:
    zipkin:
        base-url: http://localhost:9411
    sleuth:
        sampler:
          # 抽样率，默认是0.1(10%)
          probability: 1.0
```

# nacos无法识别zipkin
Spring Cloud把 http://localhost:9411/ 当作了服务发现组件里面的服务名称；于是，Nacos Client尝试从Nacos Server寻找一个名为 localhost:9411 的服务…这个服务根本不存在啊，于是就疯狂报异常（因为Nacos Client本地定时任务，刷新本地服务发现缓存）

解决方案：
```yml
spring:
    cloud:
        zipkin:
            base-url: http://localhost:9411
            # zipkin不支持注册发现，nacos无法识别zipkin疯狂报错
            discovery-client-enabled: false
        sleuth:
            sampler:
            # 抽样率，默认是0.1(10%)
            probability: 1.0
```

# zipkin持久化
+ MySQL
+ Elasticsearch
+ Cassandra

# zippin集成Elasticsearch
zipkin环境变量，配置之后会将数据存储到es里面了
如果使用es来存储，需要一个sparkjob来分析，才会有流程图


