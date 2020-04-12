[TOC]

# 配置
注解
```java
@SpringBootApplication
@EnableScheduling
public class SpringbootApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootApplication.class, args);
    }
}
```
代码
```java
@Slf4j
@Component
public class Schedule {

    @Scheduled(cron = "0 10 * * * *")
    public void task(){
        log.info("schedule");
    }
}
```
# cron
触发时间

# fixedRate
该属性的含义是上一个调用开始后再次调用的延时（不用等待上一次调用完成），这样就会存在重复执行的问题，所以不是建议使用，但数据量如果不大时在配置的间隔时间内可以执行完也是可以使用的