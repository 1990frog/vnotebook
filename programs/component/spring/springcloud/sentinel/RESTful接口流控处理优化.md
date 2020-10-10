[TOC]

在Spring Cloud架构下，如果Http请求格式是按照RESTful风格设计的，当大规模的Http请求访问系统集群，Sentinel Dashboard 的实时监控和簇点链路的记录数非常多；看到的资源名已经飙到了几千条之多！另外虽然资源名数量庞大，但是监控的TPS和并发数却非常低，甚至很多资源名仅仅被访问过一次。想从这么多资源名中找到动态调控TPS或者并发数的资源名，是非常困难的。另外由于统计量级的问题，也会导致 sentinel控制机器往Sentinel Dashboard传输的统计数据也非常大，整体请求下来，会导致整体服务的质量变得非常差。

# 问题重现
在服务端定义一个RESTful风格的URL接口
```java
@Controller
public class TestController {
    @GetMapping("/api/v1/{tenantId}/order/{orderId}/basic")
    public String pay(@PathVariable("tenantId")String tenantId,@PathVariable("orderId")String orderId){
        return tenantId+":"+orderId;
    }
}
```
引入依赖
```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    <version>0.2.2.RELEASE</version>
</dependency>
```
模拟访问
```java
RestTemplate restTemplate = new RestTemplate();
Random random = new Random(100000000);

int poolSize = 100;
ThreadPoolExecutor
    pool = new ThreadPoolExecutor(poolSize,poolSize,60, TimeUnit.MINUTES,new ArrayBlockingQueue<>(1000000));
for (int i = 0; i < poolSize; i++) {
    for (int j = 0; j < 9000; j++) {
        pool.submit(()->{
            String tenantId = ""+random.nextInt(100000000);
            String orderId =""+ random.nextInt(100000000);
            //随机生成请求连接
            String url = "http://<server-host>:<server-port>/api/v1/"+tenantId+"/order/"+orderId+"/basic";
            String result = restTemplate.getForEntity(url,String.class).getBody();
            System.out.println(result);
        });
    }
}
```

Sentinel 将每一个Http 请求的URL当成了一个唯一的资源名,用来做流控限制，这显然是非常不合适的。当大并发过来时，已经影响到了流控组件的性能消耗。

![14126519-06a10d11fc39557d](https://gitee.com/caijingquan/imagebed/raw/master/1602319298_20200330141157804_1724931872.png)

https://www.jianshu.com/p/96f5980d9798