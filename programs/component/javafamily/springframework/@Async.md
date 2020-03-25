[TOC]

# 开启async注解
```java
@EnableAsync
public class SpringbootApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootApplication.class, args);
    }

}
```
# 使用
```java
@Slf4j
@Controller
public class AsyscController {

    @Autowired
    private ApplicationContext applicationContext;

    @GetMapping("asysc")
    @ResponseBody
    public String foo(){
        log.info("入口方法线程id = {}",Thread.currentThread().getId());
        /**
         * error
         * 因为foo没有动态代理增强对象
         */
//        ((AsyscController)AopContext.currentProxy()).bar();
        /**
         * right
         * 通过上下文获取增强后的对象
         * 创建了一个新线程去执行bar
         */
        applicationContext.getBean(AsyscController.class).bar();

        /**
         * this调用，用的是一般对象
         * threadid相同
         */
        bar();
        return "";
    }

    @Async
    public void bar(){
        log.info("async方法线程id = {}",Thread.currentThread().getId());
    }

}
```