[TOC]

# 概览
+ static只能修饰`field`和`method`
+ static可以修饰内部类，但是内部类不能有`static`修饰的元素

# 用途
我们知道，当我们通过new关键字去创建对象的时候，那么数据的存储空间才会被分配，类中的成员方法才能被对象所使用。但是有两种特殊的情况：
+ 我们通过new关键字创建的对象共享同一个资源，而不是说每个对象都拥有自己的数据，或者说根本就不需要去创建对象，这个资源和对象之间是没有关系的（想想Object中的wait与notify）。
+ 希望某个方法不与包含它的类的任何对象联系在一起。总结下来就是说：即使没有创建对象，也能使用属性和调用方法，static目的就是在于解决这个问题。

# 特点
+ 被static修饰的`变量`或者`方法`是独立于该类的任何对象，也就是说，这些变量和方法不属于任何一个实例对象，而是被类的实例对象所共享。
+ 在类被加载的时候，就会去加载被`static`修饰的部分。
+ 被static修饰的变量或者方法是<font color="red">优先于对象</font>存在的，也就是说当一个类加载完毕之后，即便没有创建对象，也可以去访问。
+ 在静态方法中没有this关键字因为静态是随着类的加载而加载，而this是随着对象的创建而存在的。静态比对象优先存在。
+ 静态可以访问静态的，但是静态不能访问非静态的。
+ 非静态的可以去访问静态的。
+ 如果某个成员变量是被所有对象所共享的，那么这个成员变量就应该定义为静态变量。
+ 被static修饰的成员变量叫做静态变量，也叫做类变量，说明这个变量是属于这个类的，而不是属于是对象，没有被static修饰的成员变量叫做实例变量，说明这个变量是属于某个具体的对象的。
+ 静态只能访问静态。
+ 非静态既可以访问非静态的，也可以访问静态的。

# static如何去访问
例如
+ System.out.print()

调用方法
+ 类名.静态方法
+ 对象.静态方法(不推荐)

# 内部类不能存在静态资源
因为内部类是属于对象的，如果存在静态元素，那就间接属于对象了

# 静态变量 static field
+ 实例变量：每次创建对象，都会为每个对象分配成员变量内存空间，实例变量是属于实例对象的，在内存中，创建几次对象，就有几份成员变量。
+ 静态变量：静态变量由于不属于任何实例对象，是属于类的，所以在内存中只会有一份，在类的加载过程中，JVM为静态变量分配一次内存空间。

调用方法
+ 类名.静态变量
+ 对象.静态变量(不推荐的)

# 静态方法 static method
被static修饰的方法也叫做静态方法，因为对于静态方法来说是不属于任何实例对象的，那么就是说在静态方法内部是不能使用this的，因为既然不属于任何对象，那么就更谈不上this了。

# 静态域 static {}
用于初始化静态变量

# 静态内部类 static class
```java
public class OuterClass {

    private int age = 20;

    /**
     * 通用型
     */
    class Inner1{
        public void show(){
            System.out.println(age);
        }
    }

    /**
     * 不想外部访问
     */
    private class Inner2{
        public void show(){
            System.out.println(age);
        }
    }

    static class Inner3{
        public void show(){
            System.out.println("static args");
        }
    }

    public static void main(String[] args) {
        OuterClass.Inner1 inner1 = new OuterClass().new Inner1();
        OuterClass.Inner2 inner2 = new OuterClass().new Inner2();
        OuterClass.Inner3 inner3 = new OuterClass.Inner3();
    }
}
```