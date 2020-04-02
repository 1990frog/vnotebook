[TOC]

# 情况1：try{return}finally{}
先执行try中代码块，再执行finally中代码，再执行try中return先执行try中代码块，再执行finally 中代码，再执行try中return。
```java
public static int fun(){
    try{
        System.out.println("try block");
        return 1;
    } finally {
        System.out.println("finally block");
    }
}

public static void main(String[] args) {
    System.out.println(fun());
}

//out:
try block
finally block
1
```
# 情况2：try{return}catch{return}finally{return}
+ 有异常：先try若有异常就catch，然后finally，之后执行catch中的return，如果finally中也有return，那么就直接出去不执行catch中的return
+ 无异常：执行finally中的代码块，然后执行finally中的return

```java
public static int fun1(){
    int i = 1;
    try {
        System.out.println("try block");
        i = 1 / 0;
        return 1;
    }catch (Exception e){
        System.out.println("catch block");
        return 2;
    }finally {
        System.out.println("finally block");
    //return 3;
    }
}
public static void main(String[] args) {
    System.out.println(fun1());
}
//out:
try block
catch block
finally block
2
//finally中有return: 输出结果：
try block
catch block
finally block
3
```
try中的异常被抑制
```java
//try中的异常被抑制
public static int fun2(){
    try {
        int a = 1/0;
        return 1;
    }catch(ArithmeticException e) {
        throw new Exception("Exception异常 ");//ignore
    }finally {
        return 3;
    }
}
//out:
3
```
# 情况3：有异常try{return}catch{}finally{return}
```java
public static int fun2(){
    try {
        int a = 1/0;
        return 1;
    }catch(ArithmeticException e) {
        throw new Exception("Exception异常 ");//ignore
    }finally {
        return 3;
    }
}

public static void main(String[] args) {
    System.out.println(fun2());
}
//out:
3
```
# 总结
1. 不要在finally中 写return语句， finally 中的return语句会抑制catch中捕获的异常；
2. 不在finally中抛出异常；
3. finally仅用来释放资源；
4. 尽量将return写在后面，不要写在try-catch-finally 中；
5. 在执行try、catch中的return之前一定会执行finally中的代码（如果finally存在），如果finally中有return语句，就会直接执行finally中的return方法，所以finally中的return语句一定会被执行的。编译器把finally中的return语句标识为一个warning。
6. 匿名内部类是否可以访问方法的局部变量： 如果局部变量被匿名内部类访问，那么该局部变量相当于自动使用了final修饰。定义为final后，编译程序的实现方法：对于匿名内部类对象要访问的所有final类型局部变量，都拷贝成为该对象中的一个数据成员。这样，即使栈中局部变量已死亡，但被定义为final类型的局部变量的值永远不变，因而匿名内部类对象在局部变量死亡后，照样可以访问final类型的局部变量，因为它自己拷贝了一份，且与原局部变量的值始终一致。
