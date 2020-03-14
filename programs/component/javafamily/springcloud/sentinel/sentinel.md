[TOC]

# Sentinel对比Hystrix
|      #      |               Sentinel                |       Hystrix       |
| ----------- | ------------------------------------- | ------------------- |
| 隔离策略      | 信号量隔离（并发线程数限流）               | 线程池隔离/信号量隔离   |
| 熔断降级策略   | 基于响应时间、异常比率、异常数	          | 基于失败比率          |
| 实时指标实现   | 滑动窗口                               | 滑动窗口（基于RxJava） |
| 规则配置      | 支持多种数据源                           | 支持多种数据源        |
| 扩展性       | 多个扩展点                              | 插件的形式            |
| 基于注解的支持 | 支持                                   | 支持                |
| 限流         | 基于QPS，支持基于调用关系的限流             | 有限的支持	           |
| 流量整形      | 支持预热模式、匀速器模式、预热排队模式	      | 不支持               |
| 系统负载保护   | 支持                                   | 不支持               |
| 控制台       | 开箱即用，可配置规则，查看秒级监控，机器发现等 | 简单的监控查看	               |

# 文档
[github wiki](https://github.com/alibaba/Sentinel/wiki)
# 下载控制台
[下载地址](https://github.com/alibaba/Sentinel/releases)
# starter
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```
# 配置actuator
```yml
spring:
  application:
    # 服务名称尽量用-，不要用_，不要用特殊字符
    name: customer
  cloud:
    sentinel:
      transport:
        # port端口配置会在应用对应的机器上启动一个 Http Server，该 Server 会与 Sentinel 控制台做交互。比如 Sentinel 控制台添加了一个限流规则，会把规则数据 push 给这个 Http Server 接收，Http Server 再将规则注册到 Sentinel 中。
        port: 8719
        dashboard: localhost:8080
      # 取消懒加载
      eager: true
management:
  endpoints:
    web:
      exposure:
        include: '*'
```
# dashboard流控规则
/actuator/sentinel达到阈值1，就限流/shares/1

使用情景：
两个线程访问同一块数据，一个查询，一个修改，如果某一个触发的太多就会影响另一个，这时候可以根据业务需求去衡量希望优先读还是优先修改
保护的观念

# 快速失败
直接失败，抛异常
com.alibaba.csp.sentinel.slots.block.flow.controller.DefaultController

# Warm Up
根据codeFactor（默认3）的值，从阈值/codeFactor，经过预热时长，才到达设置的QPS阈值
com.alibaba.csp.sentinel.slots.block.flow.controller.WarmUpController

# 排队等待
匀速排队，让请求以均匀的速度通过，阈值类型必须设成QPS，否则无效
com.alibaba.csp.sentinel.slots.block.flow.controller.RateLimiterController




# 什么是流量控制
流量控制在网络传输中是一个常用的概念，它用于调整网络包的发送数据。然而，从系统稳定性角度考虑，在处理请求的速度上，也有非常多的讲究。任意时间到来的请求往往是随机不可控的，而系统的处理能力是有限的。我们需要根据系统的处理能力对流量进行控制。Sentinel 作为一个调配器，可以根据需要把随机的请求调整成合适的形状，如下图所示：


## 流量控制设计理念
流量控制有以下几个角度:
+ 资源的调用关系，例如资源的调用链路，资源和资源之间的关系；
+ 运行指标，例如 QPS、线程池、系统负载等；
+ 控制的效果，例如直接限流、冷启动、排队等。

Sentinel 的设计理念是让您自由选择控制的角度，并进行灵活组合，从而达到想要的效果。

# 熔断降级
## 什么是熔断降级
除了流量控制以外，降低调用链路中的不稳定资源也是 Sentinel 的使命之一。由于调用关系的复杂性，如果调用链路中的某个资源出现了不稳定，最终会导致请求发生堆积。

Sentinel 和 Hystrix 的原则是一致的: 当检测到调用链路中某个资源出现不稳定的表现，例如请求响应时间长或异常比例升高的时候，则对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联故障。

##熔断降级设计理念
在限制的手段上，Sentinel 和 Hystrix 采取了完全不一样的方法。

Hystrix 通过【线程池隔离】的方式，来对依赖（在 Sentinel 的概念中对应 资源）进行了隔离。这样做的好处是资源和资源之间做到了最彻底的隔离。缺点是除了增加了线程切换的成本（过多的线程池导致线程数目过多），还需要预先给各个资源做线程池大小的分配。

Sentinel 对这个问题采取了两种手段:
1. 通过并发线程数进行限制
和资源池隔离的方法不同，Sentinel 通过限制资源并发线程的数量，来减少不稳定资源对其它资源的影响。这样不但没有线程切换的损耗，也不需要您预先分配线程池的大小。当某个资源出现不稳定的情况下，例如响应时间变长，对资源的直接影响就是会造成线程数的逐步堆积。当线程数在特定资源上堆积到一定的数量之后，对该资源的新请求就会被拒绝。堆积的线程完成任务后才开始继续接收请求。
2. 通过响应时间对资源进行降级
除了对并发线程数进行控制以外，Sentinel 还可以通过响应时间来快速降级不稳定的资源。当依赖的资源出现响应时间过长后，所有对该资源的访问都会被直接拒绝，直到过了指定的时间窗口之后才重新恢复。

# 系统负载保护
Sentinel 同时提供系统维度的自适应保护能力。防止雪崩，是系统防护中重要的一环。当系统负载较高的时候，如果还持续让请求进入，可能会导致系统崩溃，无法响应。在集群环境下，网络负载均衡会把本应这台机器承载的流量转发到其它的机器上去。如果这个时候其它的机器也处在一个边缘状态的时候，这个增加的流量就会导致这台机器也崩溃，最后导致整个集群不可用。

针对这个情况，Sentinel 提供了对应的保护机制，让系统的入口流量和系统的负载达到一个平衡，保证系统在能力范围之内处理最多的请求。




