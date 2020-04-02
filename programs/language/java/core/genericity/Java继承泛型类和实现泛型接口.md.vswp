vnote_backup_file_826537664 /home/cai/Documents/vnotebook/programs/language/java/core/genericity/Java继承泛型类和实现泛型接口.md
[TOC]

# 泛型也可以继承和实现接口
```java
class Father<T>{}

interface ARB<E>{}

class child<T,E> extends Father<T> implements ARB<E>{}
```
# 泛型继承的四种情况
## 全部继承
子类泛型可以比父类多
```java
public class Test{
    public static void main(String[] args) {
        Father<Integer,String> father = new child<>(1, "2");
        child<String,String ,Boolean> child = new child<>("1","2");
    }
}
class Father<T1, T2>{
    T1 t1;
    T2 t2;

    public Father(T1 t1,T2 t2){
        this.t1 = t1;
        this.t2 = t2;
        System.out.println(this.t1.getClass());
        System.out.println(this.t2.getClass());
    }
}
class child<T1, T2, T3> extends Father<T1, T2> {
    public child(T1 t1, T2 t2) {
        super(t1, t2);
    }
}
```
## 部分继承
```java
class Child<T1,A,B> extends Father<T1,String>{}
```
同样子类泛型也可以比父类多
```java
public class Test{

    public static void main(String[] args) {
        Father<Integer,String> father = new child<>(1, "2");
        child<String,String ,Boolean> child = new child<>("1","2");
    }
}

class Father<T1, T2>{
    T1 t1;
    T2 t2;

    public Father(T1 t1,T2 t2){
        this.t1 = t1;
        this.t2 = t2;
        System.out.println(this.t1.getClass());
        System.out.println(this.t2.getClass());
    }

}
class child<T1, T2, T3> extends Father<T1,String> {//继承时将父类一个泛型实例化
    public child(T1 t1, String t2) {
        super(t1, t2);
    }
}
```
## 实现父类泛型
```java
class Child<A,B> extends Father<Integer,String>{}
```
子类将父类全部实现，子类独有，不再是继承的
```java
public class Test{
    public static void main(String[] args) {
        child<String,String ,String> child = new child<>(1,"2");
        //无论怎么写第一个都是整形，第二个是字符串
    }
}
class Father<T1, T2>{
    T1 t1;
    T2 t2;
    public Father(T1 t1,T2 t2){
        this.t1 = t1;
        this.t2 = t2;
        System.out.println(this.t1.getClass());
        System.out.println(this.t2.getClass());
    }

}
class child<T1, T2, T3> extends Father<Integer,String> {
    public child(Integer t1, String t2) {
        super(t1, t2);
    }
}
```
# 不实现父类泛型
```java
class Child extends Father{}
```
父类所有成员默认为Object类型
```java
public class Test{
    public static void main(String[] args) {
        child<String,String ,String> child = new child<>("1","2");
    }
}
class Father<T1, T2>{
    T1 t1;
    T2 t2;
    public Father(T1 t1,T2 t2){
        this.t1 = t1;
        this.t2 = t2;
        System.out.println(this.t1.getClass());
        System.out.println(this.t2.getClass());
    }
}
class child<T1, T2, T3> extends Father{
    public child(Object t1, Object t2) {
        super(t1, t2);
        // TODO Auto-generated constructor stub
    }
}
```