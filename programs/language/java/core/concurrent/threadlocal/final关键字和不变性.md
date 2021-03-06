[TOC]

# 什么是不变性
+ 如果对象在被创建后，状态就不能被修改，那么它就是不可变的
+ 具有不可变性的对象一定是线程安全的，我们不需要对其采取任何额外的安全措施，也能保证线程安全
# final的作用
+ 天生是线程安全的，而不需要额外的同步开销
+ 类：防止被继承
+ 方法：防止被重写
+ 变量：防止被修改
# final修饰变量
含义：被final修饰的变量，意味着值不能被修改。如果变量是对象，那么对象的引用不能变，但是对象自身的内容依然可以变化
## 三种变量
+ final instance variable（对象属性：类中的final属性）
+ final static variable（类属性：类中的static final属性）
+ final local variable（栈变量：方法中的final变量）
## 赋值时机（全都是初始化赋值）
+ 属性被声明为final后，该变量则只能被赋值一次。且一旦赋值，final的变量就不能再被改变，无论如何也不会变
+ 如果初始化不赋值，后续赋值，就是从null变成你的赋值，这就违反了final不变的原则了
## 三种赋值方式
1. 直接赋值
2. 全部在构造方法中赋初值
3. 代码块赋值
## final对象属性
+ 第一种在声明变量的等号右边直接赋值
+ 第二种就是构造函数中赋值
+ 第三种就是在类的初始化代码块中赋值（不常用）
+ 如果不采用第一种赋值方法，那么久必须在2，3中挑一种来赋值，而不能不赋值，这是final语法所规定的
+ 只能选一种赋值方式
+ final声明的变量不能为null，所以必须保证被赋值，不然无法编译

```java
public class FinalVariableDemo{
    // Object的final三种赋值方式，只能使用一种
    private final int a = 1；//方法一直接赋值
    public FinalVariableDemo(int a){//方法二构造器赋值
        this.a=a;
    }
    {
        a=7;//方法三静态域赋值
    }
    // static的final赋值方式
    static final int b = 2;
    static {
        b = 2;
    }
    // 方法的final赋值方式
    public void dosomething(){
        final int c;
        c = 3;
        System.out.println(c);
    }
}
```
## final类属性（静态）
+ 在声明变量的等号右边直接赋值外
+ static final变量还可以用static初始化代码块赋值，但是不能用普通的初始化代码块赋值
## final方法变量
+ 和前面两种不同，由于这里的变量是在方法里的，所以没有构造函数，也不存在初始代码块
+ final local variable不规定赋值时机，只要求在使用前必须赋值，这和方法中的非final变量的要求也是一样的
## 三种属性赋值横比

|     种类      |                  方式                   |
| ------------ | -------------------------------------- |
| Object        | 1.直接赋值</br>2.构造器赋值</br>3.代码块赋值 |
| Class(static) | 1.直接赋值</br>2.静态代码块赋值             |
| Method        | 1.仅要求使用前赋值                        |
# final方法
+ 构造方法不允许final修饰
+ 不可被重写，也就是不能被override，即便是子类有同样名字的方法，那也不是override，这个和static方法是一个道理（static绑定类class对象，当前类与拓展类不是一个类）
+ 引申：static方法不能被重写（但允许存在重名的方法，static在创建的时候就会绑定方法，而不是后面动态绑定的）

```java
public class superCalss{
    public void drink(){}
    public final void eat(){}
    public static void sleep(){}
}
```

# final修饰类
不可被继承，例如典型的String类就是final的
拓展：String为什么不能被修改，每次都是新new，因为value是final的char数组

# final不变性
## 不变性并不意味着，简单地用final修饰就是不可变
+ 对于基本数据类型，确实final修饰后就具有不变性
+ 但是对于对象类型，需要改对象保证自身被创建后永远不会变才可以
## 如何利用final实现对象不可变
+ 对象创建后，其状态就不能修改
+ 所有属性都是final修饰的
+ 对象创建过程中没有发生逸出

