[TOC]

# 概览
Feign是Netflix开源的声明式http客户端
[github](https://github.com/OpenFeign/feign)
![overview](_v_images/20200308094416196_354967477.png)

# Feign组成
|        接口        |               作用               |                                    默认值                                    |
| ----------------- | ------------------------------- | --------------------------------------------------------------------------- |
| Feign.Builder      | Feign的入口                       | Feign.Builder                                                                |
| Client             | Feign底层用什么去请求               | 和Ribbon配合时：LoadBalancerFeignClient</br>不和Ribbon配合时：feign.Client.Default |
| Contract           | 契约，注解支持                     | SpringMvcContract                                                            |
| Encoder            | 编码器，用于将对象转换成HTTP请求消息体 | SpringEncoder                                                                |
| Decoder            | 解码器，将响应消息体转换成对象        | ResponseEntityDecoder                                                        |
| Logger             | 日志管理器                        | Slf4jLogger                                                                  |
| RequestInterceptor | 用于为每个请求添加通用逻辑           | 无                                                                           |

# 引入feign
+ @EnableFeignClients
+ @FeignClient

```java
@SpringBootApplication
@EnableFeignClients
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

# 多种配置方式
+ java代码方式
+ 配置属性方式

# 自定义Feign日志级别
|     级别      |                打印内容                 |
| ------------ | -------------------------------------- |
| NONE（默认值） | 不记录任何日志                            |
| BASIC        | 仅记录请求方法、URL、响应状态代码及执行时间    |
| HEADERS      | 记录BASIC级别的基础上，记录请求和响应的header |
| FULL         | 记录请求和响应的header、body和元数据        |

# 局部配置
## 代码配置日志
```java
@FeignClient(name = "productor",configuration = ProductorFeignConfiguration.class)
public interface ProductorFeignClient {

    @GetMapping("/productor/{id}")
    HashMap queryProduct(@PathVariable("id")String id);
}

/**
 * 如果加了@Configuration注解，
 * 就要将ProductorFeignConfiguration移动到Application的@Scan能扫描的路径以外，避免上下文重叠
 * 如果被扫描到那就是全局的
 */
//@Configuration
public class ProductorFeignConfiguration {

    @Bean
    public Logger.Level level(){
        return Logger.Level.FULL;
    }

}
```
## 配置文件配置日志
```yml
feign:
  client:
    config:
      # 想要调用的微服务的名称
      productor:
        loggerLevel:full
```

# 全局配置
## 代码配置
方式一：让上下文重叠（bug）
方式二：`@EnableFeignClients(defaultConfiguration=xxx.class)`
```java
@SpringBootApplication
@EnableFeignClients(defaultConfiguration = ProductorFeignConfiguration.class)
public class CustomerApplication {

    public static void main(String[] args) {
        SpringApplication.run(CustomerApplication.class, args);
    }

}
```
## 配置方式
```yml
feign:
  client:
    config:
      default:
        loggerLevel:full
```

# 代码方式支持的配置项
|             配置项             |                       作用                       |
| ----------------------------- | ----------------------------------------------- |
| Logger.Level                  | 指定日志级别                                      |
| Retryer                       | 指定重试策略                                      |
| ErrorDecoder                  | 指定错误解码器                                     |
| Request.Options               | 超时时间                                          |
| Collection<RequestInterceptor> | 拦截器                                           |
| SetterFactory                 | 用于设置Hystrix的配置属性，</br>Feign整合Hystrix才会用 |
# 配置属性支持的配置项
```yml
feign:
  client:
    config:
      feignName:
        #连接超时时间
        connectTimeout: 5000
        #读取超时时间
        readTimeout: 5000
        #日志级别
        loggerLevel: full
        #错误解码器
        errorDecoder: com.example.SimpleErrorDecoder
        #重试策略
        retryer: com.example.SimpleRetryer
        #拦截器
        requestInterceptors:
          - com.example.FooRequestInterceptor
          - com.example.BarRequestInterceptor
        # 是否对404错误解码
        decode404: false
        #编码器
        encoder: com.example.SimpleEncoder
        #解码器
        decoder: com.example.SimpleDecoder
        #契约
        contract: com.example.SimpleContract
```
全局
```yml
feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
```

# Feign脱离Ribbon使用
设置指定的url，访问对应的服务
```java
@FeignClient(name="baidu",url="http://")
public interface xxx{}
```

# RestTemplate VS Feign
|      角度      | RestTemplate | Feign |
| ------------- | ------------ | ----- |
| 可读性、可维护性 | 一般         | 极佳   |
| 开发体验       | 欠佳         | 极佳   |
| 性能           | 很好         | 中等   |
| 灵活性         | 极佳         | 中等   |

# Feign性能优化
开启连接池连接池（提升15%左右）
关闭日志：建议设置成basic，绝对不建议设置为full，打印的日志太多，性能消耗大（Feign的性能中等，可能官方对自己的性能也是知道的，索性全部关闭日志了）

两种实现方式：
+ httpclient
+ okhttp
## Apache httpclient方式
加依赖
```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```
配置
```yml
feign:
    httpclient:
        # 让feign使用apache httpclient做请求，而不是默认的urlconnection
        enabled:true
        # feign的最大连接数
        max-connections:200
        # feign单个路径的最大连接数
        max-connections-per-route:50
```
最优值压测取得
## okhttp方式
加依赖
```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
    <version></version>
</dependency>
```
配置
```yml
feign:
    okhttp:
        enabled:true
    httpclient:
        # feign的最大连接数
        max-connections:200
        # feign单个路径的最大连接数
        max-connections-per-route:50
```

# Feign构造多参数的请求
GET请求多参数的URL

方法一[推荐]
```java
@FeignClient("microservice-provider-user")
public interface UserFeignClient {
  @GetMapping("/get")
  public User get0(@SpringQueryMap User user);
}
```
方法二[推荐]
```java
@FeignClient(name = "microservice-provider-user")
public interface UserFeignClient {
  @RequestMapping(value = "/get", method = RequestMethod.GET)
  public User get1(@RequestParam("id") Long id, @RequestParam("username") String username);
}
```
方法三
```java
@FeignClient(name = "microservice-provider-user")
public interface UserFeignClient {
  @RequestMapping(value = "/get", method = RequestMethod.GET)
  public User get2(@RequestParam Map<String, Object> map);
}
```
POST请求包含多个参数
```java
@FeignClient(name = "microservice-provider-user")
public interface UserFeignClient {
  @RequestMapping(value = "/post", method = RequestMethod.POST)
  public User post(@RequestBody User user);
}
```

# 使用Spring Cloud Feign上传文件
加依赖
```xml
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
	<version>lasted</version>
</dependency>
<dependency>
	<groupId>io.github.openfeign.form</groupId>
	<artifactId>feign-form-spring</artifactId>
	<version>lasted</version>
</dependency>
```
feign client
```java
@FeignClient(name = "ms-content-sample", configuration = UploadFeignClient.MultipartSupportConfig.class)
public interface UploadFeignClient {
    @RequestMapping(value = "/upload", method = RequestMethod.POST,
            produces = {MediaType.APPLICATION_JSON_UTF8_VALUE},
            consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    @ResponseBody
    String handleFileUpload(@RequestPart(value = "file") MultipartFile file);

    class MultipartSupportConfig {
        @Bean
        public Encoder feignFormEncoder() {
            return new SpringFormEncoder();
        }
    }
}
```
注意点
+ @RequestMapping中的produeces 、consumes 不能少；
+ 接口定义中的注解@RequestPart(value = "file") 不能写成@RequestParam(value = "file" 。
+ 最好将Hystrix的超时时间设长一点，例如5秒，否则可能文件还没上传完，Hystrix就超时了，从而导致客户端侧的报错。

# Java代码自定义Feign Client的注意点与坑
```java
@FeignClient(name = "microservice-provider-user", configuration = UserFeignConfig.class)
public interface UserFeignClient {
  @GetMapping("/users/{id}")
  User findById(@PathVariable("id") Long id);
}

/**
 * 该Feign Client的配置类，注意：
 * 1. 该类可以独立出去；
 * 2. 该类上也可添加@Configuration声明是一个配置类；
 * 配置类上也可添加@Configuration注解，声明这是一个配置类；
 * 但此时千万别将该放置在主应用程序上下文@ComponentScan所扫描的包中，
 * 否则，该配置将会被所有Feign Client共享，无法实现细粒度配置！
 * 个人建议：像我一样，不加@Configuration注解
 *
 * @author zhouli
 */
class UserFeignConfig {
  @Bean
  public Logger.Level logger() {
    return Logger.Level.FULL;
  }
}
```
+ 配置类上也可添加@Configuraiton 注解，声明这是一个配置类；但此时千万别将该放置在主应用程序上下文@ComponentScan 所扫描的包中，否则，该配置将会被所有Feign Client共享（相当于变成了通用配置，其实本质还是Spring父子上下文扫描包重叠导致的问题），无法实现细粒度配置！
+ 个人建议：不加@Configuration注解，省得进坑。
+ 最佳实践：尽量用配置属性自定义Feign的配置

# @FeignClient注解属性
`@FeignClient(name = "microservice-provider-user")`
在早期的Spring Cloud版本中，无需提供name属性，从Brixton版开始，@FeignClient必须提供name属性，否则应用将无法正常启动！

另外，name、url等属性支持占位符。例如：
```
@FeignClient(name = "${feign.name}", url = "${feign.url}")
```

# 类级别的@RequestMapping会被SpringMVC加载
```java
@RequestMapping("/users")
@FeignClient(name = "microservice-user")
public class TestFeignClient {
    // ...
}
```
类上的@RequestMapping 注解也会被Spring MVC加载。该问题现已经被解决，早期的版本有两种解决方案：

方案1：不在类上加@RequestMapping 注解；

方案2：添加如下代码：
```java
@Configuration
@ConditionalOnClass({ Feign.class })
public class FeignMappingDefaultConfiguration {
    @Bean
    public WebMvcRegistrations feignWebRegistrations() {
        return new WebMvcRegistrationsAdapter() {
            @Override
            public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
                return new FeignFilterRequestMappingHandlerMapping();
            }
        };
    }

    private static class FeignFilterRequestMappingHandlerMapping extends RequestMappingHandlerMapping {
        @Override
        protected boolean isHandler(Class<?> beanType) {
            return super.isHandler(beanType) && !beanType.isInterface();
        }
    }
}
```

# 首次请求失败

# 如需产生Hystrix Stream监控信息，需要做一些额外操作
Feign本身已经整合了Hystrix，可直接使用@FeignClient(value = "microservice-provider-user", fallback = XXX.class) 来指定fallback类，fallback类继承@FeignClient所标注的接口即可。

但是假设如需使用Hystrix Stream进行监控，默认情况下，访问http://IP:PORT/actuator/hystrix.stream 是会返回404，这是因为Feign虽然整合了Hystrix，但并没有整合Hystrix的监控。如何添加监控支持呢？需要以下几步：

第一步：添加依赖，示例：

<!-- 整合hystrix，其实feign中自带了hystrix，引入该依赖主要是为了使用其中的hystrix-metrics-event-stream，用于dashboard -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
第二步：在启动类上添加@EnableCircuitBreaker 注解，示例：

@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
@EnableCircuitBreaker
public class MovieFeignHystrixApplication {
  public static void main(String[] args) {
    SpringApplication.run(MovieFeignHystrixApplication.class, args);
  }
}
第三步：在application.yml中添加如下内容，暴露hystrix.stream端点：

management:
  endpoints:
    web:
      exposure:
        include: 'hystrix.stream'
这样，访问任意Feign Client接口的API后，再访问http://IP:PORT/actuator/hystrix.stream ，就会展示一大堆Hystrix监控数据了。
