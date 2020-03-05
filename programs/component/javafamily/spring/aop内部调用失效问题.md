[TOC]

Spring AOP基本原理
Spring AOP是基于动态代理机制实现的，通过动态代理机制生成目标对象的代理对象，当外部调用目标对象的相关方法时，Spring注入的其实是代理对象Proxy，通过调用代理对象的方法执行AOP增强处理，然后回调目标对象的方法。


![9564807-de1289d215c69da0](_v_images/20200305135752919_1004907721.png)

我们来看下面一个需要进行AOP增强的类，外部调用methodA()且该方法中调用methodB()，调用methodB()不会执行AOP的增强逻辑。

```java
import org.springframework.stereotype.Service;
/**
 * 目标对象类
 * @author Gufung
 */
@Service
public class TestAopService {

    public void methodA() {
        System.out.println("method A run");
        //内部调用方法B时AOP的增强处理方法不会执行
        methodB();
    }
    
    public void methodB() {
        System.out.println("method B run");
    }
}
```

通过上图的AOP基本原理，我们知道真正执行methodA()的是目标对象，那么methodA()中调用methodB()就是目标对象的方法而不是代理对象的，也就自然不会执行AOP的增强逻辑。

解决方案
最基本的思路就是在内部调用时要调用代理对象的方法这样就可以执行AOP的增强逻辑了。

A.开启暴露代理对象
1.配置spring-context.xml通过注解实现AOP

```
<!-- 通过@Aspect注解实现AOP -->
<!-- proxy-target-class="true"表示使用CGlib动态代理 -->
<!-- expose-proxy="true"暴露代理对象 -->
<aop:aspectj-autoproxy proxy-target-class="true" expose-proxy="true"/>
```

2.目标对象内部调用时获取代理对象，再调用代理对象的方法
```
import org.springframework.aop.framework.AopContext;
import org.springframework.stereotype.Service;
/**
 * 目标对象类
 * @author Gufung
 *
 */
@Service
public class TestAopService {

    public void methodA() {
        System.out.println("method A run");
        //methodB();
        //从spring上下文获取代理对象执行方法
        ((TestAopService) AopContext.currentProxy()).methodB();
    }

    public void methodB() {
        System.out.println("method B run");
    }
}
```

B.利用初始化方法在目标对象中注入代理对象
在目标对象类中注入spring上下文，通过context获取代理对象，并调用代理对象的方法。

注意：该方案对于scope为prototype的bean无法适用，因为每次获取bean时都返回一个新的对象。

```
import javax.annotation.PostConstruct;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Service;
import com.blog.common.aop.service.TestService;

@Service
public class TestServiceImpl implements TestService {

    @Autowired
    private ApplicationContext context;

    private TestService proxyObject;

    @PostConstruct
    // 初始化方法，在IOC注入完成后会执行该方法
    private void setSelf() {
        // 从spring上下文获取代理对象（直接通过proxyObject=this是不对的，this是目标对象）
        // 此种方法不适合于prototype Bean，因为每次getBean返回一个新的Bean
        proxyObject = context.getBean(TestService.class);
    }

    public void methodA() throws Exception {
        System.out.println("method A run");
        System.out.println("method A 中调用method B，通过注入的代理对象，调用代理对象的方法，解决内部调用实现的问题。");
        proxyObject.methodB(); //调用代理对象的方法，解决内部调用失效的问题
    }

    public void methodB() {
        System.out.println("method B run");
    }

}
```