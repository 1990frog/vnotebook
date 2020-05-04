[TOC]

下面以Redis为例，讲解Spring Integration里面如何使用分布式锁。

# 加依赖
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-integration</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.integration</groupId>
  <artifactId>spring-integration-redis</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
# 写配置
```yml
spring:
  redis:
    port: 6379
    host: localhost
```
# 初始化
```java
@Configuration
public class RedisLockConfiguration {
  @Bean
  public RedisLockRegistry redisLockRegistry(RedisConnectionFactory redisConnectionFactory) {
    return new RedisLockRegistry(redisConnectionFactory, "spring-cloud");
  }
}
```
# 测试
```java
@GetMapping("redislock")
    @ResponseBody
    public String redisLock() {

        String port = applicationContext.getEnvironment().getProperty("server.port");
        Lock lock = redisLockRegistry.obtain("lock");

        try{

            while (lock.tryLock()){
                System.out.println(port+"获取到锁");
                for (int i = 10; i > 0; i--) {
                    Thread.sleep(1000);
                    System.out.println(port+"倒计时-"+i+"s");
                }
                break;
            }
            System.out.println(port+"释放锁");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
        return "redis lock";
    }
```