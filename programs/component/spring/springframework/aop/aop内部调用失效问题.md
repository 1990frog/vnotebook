[TOC]

![9564807-de1289d215c69da0](https://gitee.com/caijingquan/imagebed/raw/master/1602319174_20200305135752919_1004907721.png)

# 解决方案
+ 方法一：直接从BeanFactory中获取再次代理Bean
+ 方法二：从AopContext中获取代理Bean`@EnableAspectJAutoProxy(exposeProxy = true)`

# 代码
aop代码
```java
@Slf4j
@Aspect
@Component
public class ThisErrorAop {

    /**
     * 定义切点：如果有此注解的地方
     */
    @Pointcut("execution(public * com.action.aop.error.ThisErrorController.*())")
    public void serviceAspect() {
    }

    @Before(value = "serviceAspect()")
    public void before(){
        log.info("===before===");
    }

    @After(value = "serviceAspect()")
    public void after(){
        log.info("===after===");
    }
}
```
监控类代码
```java
@Slf4j
@Controller
public class ThisErrorController {

    @Autowired
    private ApplicationContext applicationContext;

    public void foo(){
        log.info("hello");
        // error
        bar();
        // 解决方案1
        ((ThisErrorController)AopContext.currentProxy()).bar();
        // 解决方案2
        applicationContext.getBean(this.getClass()).bar();
    }

    public void bar(){
        log.info("normal");
    }

}
```
运行结果
```
2020-03-29 12:50:28.105  INFO 6668 --- [           main] com.action.aop.error.ThisErrorAop        : ===before===
2020-03-29 12:50:28.108  INFO 6668 --- [           main] c.action.aop.error.ThisErrorController   : hello
2020-03-29 12:50:28.108  INFO 6668 --- [           main] c.action.aop.error.ThisErrorController   : normal
2020-03-29 12:50:28.108  INFO 6668 --- [           main] com.action.aop.error.ThisErrorAop        : ===before===
2020-03-29 12:50:28.109  INFO 6668 --- [           main] c.action.aop.error.ThisErrorController   : normal
2020-03-29 12:50:28.109  INFO 6668 --- [           main] com.action.aop.error.ThisErrorAop        : ===after===
2020-03-29 12:50:28.109  INFO 6668 --- [           main] com.action.aop.error.ThisErrorAop        : ===before===
2020-03-29 12:50:28.109  INFO 6668 --- [           main] c.action.aop.error.ThisErrorController   : normal
2020-03-29 12:50:28.109  INFO 6668 --- [           main] com.action.aop.error.ThisErrorAop        : ===after===
2020-03-29 12:50:28.109  INFO 6668 --- [           main] com.action.aop.error.ThisErrorAop        : ===after===
```