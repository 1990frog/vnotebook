[TOC]

在创建子类对象时，会把父类里的成员变量和方法也加载进内存（因为要加载进内存，所以要看下这些数据是怎么初始化的，所以调用了父类的构造，仅此而已，并不是去创建了父类对象）

然后用this和super这两个引用来区分是父类的还是子类的，<font coloer="red">但是这个内存区域是子类的内存区域，绝不是父类的</font>
this指向了不仅父类中可继承的成员变量和可继承的方法外，它还指向了子类的成员变量和方法，而super仅仅只是指向了子类对象中从父类继承的成员变量和方法。

也就是说，在子类对象里（堆空间），不仅有自己的属性方法还有一块空间是指向父类的属性方法

# this与super
1. this指当前对象，super指继承的父类
this指向了不仅父类中可继承的成员变量和可继承的方法外，它还指向了子类的成员变量和方法
super仅仅只是指向了子类对象中从父类继承的成员变量和方法
2. this和super不能同时出现在一个构造函数里面
3. this或super调构造器时必须放在第一句，super()调用父类构造方法，this() 调用本类构造方法，及时调用了this,默认还会调用super。
不放在第一句编译报错：`Constructor call must be the first statement in a constructor`
4. this和super都指的是对象，所以，均不可以在static环境下使用
5. 如果父类只有一个带参构造方法，那么子类必须调用父类的构造方法super（… params）。注意：子类每个构造方法中默认有super()，前提是父类有无参构造。

```java
class Father{
    public void foo(){
        System.out.println("father");
    }
}

public class Main extends Father{

    private void printSuper(){
        System.out.println(super.hashCode());
    }

    private void printThis(){
        System.out.println(this.hashCode());
    }

    public static void main(String[] args) {
        Main main = new Main();
        main.printSuper();
        main.printThis();
    }
}
1650967483
1650967483
```