# 深入理解Lambda表达式
+ Lambda表达式在JVM底层解析成私有静态方法和匿名内部类型
+ 通过实现接口的匿名内部类型中接口方法调用静态实现方法，完成Lambda表达式的执行

```java
public class App{

    @FunctionalInterface
    interface Inter{
        void test();
    }

    public static void main(String[] arrgs){
        Inter inter = ()->System.out.println("test");
        inter.test();
    }
}
```

简单查看Lambda字节码
`javap -p App`
```java
Compiled from "App.java"
public class App {
  public App();
  public static void main(java.lang.String[]);
  private static void lambda$main$0();
}
```


导出内部类：
`java -Djdk.internal.lambda.dumpProxyClasses App`
test

结合以上我们可以总结出：
+ lambda表达式会被编译为invokedynamic指令
+ 每一个lambda表达式的实现逻辑均会被封装为一个静态私有方法
+ 只要存在lambda表达式调用，便会生成一个内部类
+ 内部类中每一个方法（启动方法 BoostrapMethod）对应一个lambda表达式所生成的静态私有方法，内部类中的方法用以生成对应的调用点绑定到相应的invokedynamic指令上
