[TOC]

Ribbon简介
Ribbon是Netflix发布的负载均衡器，它可以帮我们控制HTTP和TCP客户端的行为。只需为Ribbon配置服务提供者地址列表，Ribbon就可基于负载均衡算法计算出要请求的目标服务地址。

Ribbon默认为我们提供了很多的负载均衡算法，例如轮询、随机、响应时间加权等——当然，为Ribbon自定义负载均衡算法也非常容易，只需实现IRule 接口即可。



# 负载均衡的两种方式
+ 服务端负载均衡（nginx）
+ 客户端负载均衡（ribbon）

# ribbon接口
|           接口           |          作用           |                                   默认值                                   |
| ----------------------- | ---------------------- | ------------------------------------------------------------------------- |
| IClientConfig            | 读取配置                 | DefaultClientConfigImpl                                                    |
| IRule                   | 负载均衡规则，选择实例      | ZoneAvoidanceRule                                                          |
| IPing                   | 筛选掉ping不通的实例       | DummyPing                                                                 |
| Serverlist<Server>       | 交给Ribbon的实例列表      | Ribbon:ConfigurationBasedServerList</br>Spring Cloud Alibaba:NacosServerList |
| ServerListFilter<Server> | 过滤掉不符合条件的实例      | ZonePreferenceServerListFilter                                             |
| ILoadBalancer            | Ribbon的入口             | ZoneAwareLoadBalancer                                                      |
| ServerListUpdater        | 更新交给Ribbon的List的策略 | PollingServerListUpdater                                                   |

# ribbon内置负载均衡规则
| 规则名称    |  特点   |
| --- | --- |
|  AvailabilityFiteringRule   |   过滤掉一直连接失败的被标记为circuit tripped的后端server，并过滤掉那些高并发的后端server或者使用一个AvailablityPredicate来包含过滤server的逻辑，其实就是检查status里记录的各个server的运行状态  |
|  BestAvailableRule   | 选择一个最小的并发请求的server，逐个考察server，如果server被tripped了，则跳过    |
|   RandomRule  |   随机选择一个server  |
|  ResponseTimeWeightedRule   |  已废弃，作用同WeightedResponseTimeRule   |
|  RetryRule   |  对选定的负载均衡策略机上重试机制，在一个配置时间段内当选择Server不成功，则一直尝试使用subRule的方式选择一个可用的server  |
|   RoundRobinRule  |   轮询选择，轮询index，选择index对应位置的server  |
|   WeightedResponseTimeRule  |   根据响应时间加权，响应时间越长，权重越小，被选中的可能性越低  |
|  ZoneAvoidanceRule   |   复合判断server所zone的性能和server的可选性选择server，在没有zone的环境下，类似于轮询（RoundRobinRule）  |
默认是轮询

# 细粒度配置自定义
Configuration
```java
@Configuration
@RibbonClient(name="user-center",configuration = RibbonConfiguration.class)
public class MyRibbonConfiguration {
}
```
Configuration
```java
@Configuration
public class RibbonConfiguration {

    @Bean
    public IRule ribbonRule(){
        return new RandomRule();
    }
}
```

# springboot坑
启动类@SpringBootApplication注解
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```
内置@ComponentScan扫描注解，位置限定为Application类所在的目录下全部class

ribbon配置类不能被Appliaction扫描到的原因为，Application为父上下文，而Ribbon的为子上下文，父子上下文重叠会造成各种奇葩问题
类似问题：
Spring+SpringMVC配置事务管理无效
应用无法启动

spring的上下文其实是一个树状的上下文，而@SpringBootApplication上下文其实是一个主上下文，而

ribbon也是一个上下文，是一个子上下文，父子上下文扫描的包一旦重叠，会导致各种各样奇葩的问题：

ribbonconfiguration如果放在启动类下，会被共享成为全局的配置

The CustomConfiguration clas must be a @Configuration class, but take care that it is not in a @ComponentScan for the main application context. Otherwise, it is shared by all the @RibbonClients. If you use @ComponentScan (or @SpringBootApplication), you need to take steps to avoid it being included (for instance, you can put it in a separate, non-overlapping package or specify the packages to scan explicitly in the @ComponentScan).

# 配置文件配置方式
用属性配置，更加简单，并且没有坑（上下文重叠）
```yml
spring:
  application:
    name: user-center

user-center:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

# 两种方式对比
| 配置方式 |                            优点                            |                   缺点                    |
| ------ | ---------------------------------------------------------- | ---------------------------------------- |
| 代码配置 | 基于代码，更加灵活                                            | 有小坑（父子上下文）</br>线上修改得重新打包、发布 |
| 属性配置 | 易上手</br>配置更加直观</br>线上修改无需重写打包、发布</br>优先级更高 | 极端场景下没有代码配置方式灵活                 |

# 细粒度配置总结
+ 尽量使用属性配置，属性方式实现不了的情况下再考虑用代码配置
+ 在同一个微服务内尽量保持单一性，比如统一使用属性配置，不要两种方式混用，增加定位代码的复杂性

# 全局配置
```java
@Configuration
@RibbonClients(defaultConfiguration = RibbonConfiguration.class)
public class MyRibbonConfiguration {
}
```


1、java代码配置

单独建包，不然会产生spring父子上下文重叠，导致该配置为全局配置


# 配置属性方式
<clientName>.ribbon.
+ NFloadBalancerClassName：ILoadBalancer实现类
+ NFloadBalancerRuleClassName：IRule实现类
+ NFloadBalancerPingClassName：IPing实现类
+ NiWSServerListClassName：ServerList实现类
+ NiWSServerListFilterClassName：ServerListFilter实现类

# 饥饿加载
默认情况下ribbon是懒加载

开启饥饿加载
```yml
ribbon:
    eager-load:
        enabled:true
        # 指定ribbon请求哪些微服务使用饥饿加载，多个可以使用逗号分隔
        clients:user-center
```

# 在nacos控制台设置权重
值在0-1之间，值越大几率越大

# ribbon负载均衡权重
https://imooc.com/article/288660

# 拓展ribbon-同集群优先