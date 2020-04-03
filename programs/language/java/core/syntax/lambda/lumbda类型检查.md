lambda的类型是从使用lambda的上下文推断出来的。上下文（比如，接受它传递的方法的参数，或接受它的值的局部变量）中lambda表达式需要的类型称为目标类型

# Lambda表达式类型检查
+ 表达式类型检查
+ 参数类型检查(参数是lambda表达式)


# 方法重载的问题
+ Java类型系统中的方法重载
+ 方法重载的实现
+ 当方法重载遇上Lambda表达式

```java
public class App2 {

    interface Param1{
        void outInfo(String info);
    }
    interface Param2{
        void outInfo(String info);
    }

    public void lambdaMethod(Param1 param){
        param.outInfo("hello param1");
    }

    public void lambdaMethod(Param2 param){
        param.outInfo("hello param2");
    }

    public static void main(String[] args) {
        App2 app2 = new App2();
        app2.lambdaMethod(new Param1() {
            @Override
            public void outInfo(String info) {
                System.out.println(info);
            }
        });
        app2.lambdaMethod(new Param2() {
            @Override
            public void outInfo(String info) {
                System.out.println(info);
            }
        });
    }
}
```
lambda表达式存在类型检查-->自动推断lambda表达式的目标类型
lambdaMethod()-->方法-->方法重载
-->Param1函数式接口
-->Param2函数式接口
调用方法-->传递lambda表达式-->自动推断-->Param1|Param2

