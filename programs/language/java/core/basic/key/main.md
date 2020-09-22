[TOC]

```java
public class Main {

    // 只有这个才是主函数，下面的弟弟不行
    public static void main(String[] args) {
        main();
    }

    // 普通方法，main弟弟
    public static void main() {
        System.out.println("hello main");
    }
}
```
# 概览
在Java中，main方法是Java应用程序的入口方法。程序运行时，要执行的第一个方法是main（）方法。此方法与其他方法有很大不同。
例如，方法的名称必须为main，方法的类型必须为public static void，方法必须接收字符串数组的参数。

# 函数签名解析
+ public修饰符：java类由java虚拟机（JVM）调用，为了没有限制可以自由的调用，所以采用public修饰符。
+ static修饰符：JVM调用这个主方法时肯定不是先创建这个主类的对象，再通过对象来调用方法，而是直接通过该类来调用这个方法，因此需要使用static修饰符修饰这个类。
+ void返回值：主方法被JVM调用，将返回值返回给JVM没有任何意义，因此该方法没有返回值，所以使用void。

扩展：
+ public：该修饰符表明该数据成员、成员函数是对所有用户开放的，所有用户都可以直接进行调用。
+ static：该修饰符表示静态的意思，简单理解被static修饰符修饰的成员都属于类本身，而不属于类的某个实例，静态成员不能能直接访问非静态成员。
+ void：使用void说明没有返回值。

# 参数
我们首先知道，谁调用方法，谁去传递形参，所以args形参由JVM负责赋值，JVM给args赋了什么值？

通过java命令传递参数：
```
>java [-options] class [args...]
>java [-options] -jar jarfile [args...]
>java HelloWord args1 args2
```