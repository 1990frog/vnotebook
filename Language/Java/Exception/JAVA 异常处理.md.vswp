vnote_backup_file_826537664 /home/cai/Documents/vnote_notebooks/vnotebook/Language/Java/Exception/JAVA 异常处理.md
[TOC]

# JAVA 异常类型结构
## Throwable
Throwable 是所有异常类型的基类，Throwable 下一层分为两个分支，Error 和 Exception.

![](_v_images/20191024145151801_520758280.png)

## Error 和 Exeption
### Error
Error 描述了 JAVA 程序运行时系统的内部错误，通常比较严重，除了通知用户和尽力使应用程序安全地终止之外，无能为力，应用程序不应该尝试去捕获这种异常。通常为一些虚拟机异常，如 StackOverflowError 等。

### Exception
Exception 类型下面又分为两个分支，一个分支派生自 RuntimeException，这种异常通常为程序错误导致的异常；另一个分支为非派生自 RuntimeException 的异常，这种异常通常是程序本身没有问题，由于像 I/O 错误等问题导致的异常，每个异常类用逗号隔开。

# 受查异常和非受查异常
## 受查异常
受查异常会在编译时被检测。如果一个方法中的代码会抛出受查异常，则该方法必须包含异常处理，即 try-catch 代码块，或在方法签名中用 throws 关键字声明该方法可能会抛出的受查异常，否则编译无法通过。如果一个方法可能抛出多个受查异常类型，就必须在方法的签名处列出所有的异常类。

通过 throws 关键字声明可能抛出的异常

```java
private static void readFile(String filePath) throws IOException {
    File file = new File(filePath);
    String result;
    BufferedReader reader = new BufferedReader(new FileReader(file));
    while((result = reader.readLine())!=null) {
        System.out.println(result);
    }
    reader.close();
}
```

try-catch 处理异常

```java
private static void readFile(String filePath) {
    File file = new File(filePath);
    String result;
    BufferedReader reader;
    try {
        reader = new BufferedReader(new FileReader(file));
        while((result = reader.readLine())!=null) {
            System.out.println(result);
        }
        reader.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

## 非受查异常
非受查异常不会在编译时被检测。JAVA 中 Error 和 RuntimeException 类的子类属于非受查异常，除此之外继承自 Exception 的类型为受查异常。

# 异常的抛出与捕获
## 直接抛出异常
通常，应该捕获那些知道如何处理的异常，将不知道如何处理的异常继续传递下去。传递异常可以在方法签名处使用 throws 关键字声明可能会抛出的异常。

```java
private static void readFile(String filePath) throws IOException {
    File file = new File(filePath);
    String result;
    BufferedReader reader = new BufferedReader(new FileReader(file));
    while((result = reader.readLine())!=null) {
        System.out.println(result);
    }
    reader.close();
}
```

## 封装异常再抛出
有时我们会从 catch 中抛出一个异常，目的是为了改变异常的类型。多用于在多系统集成时，当某个子系统故障，异常类型可能有多种，可以用统一的异常类型向外暴露，不需暴露太多内部异常细节。

```java
private static void readFile(String filePath) throws MyException {
    try {
        // code
    } catch (IOException e) {
        MyException ex = new MyException("read file failed.");
        ex.initCause(e);
        throw ex;
    }
}
```

## 捕获异常
在一个 try-catch 语句块中可以捕获多个异常类型，并对不同类型的异常做出不同的处理

```java
private static void readFile(String filePath) {
    try {
        // code
    } catch (FileNotFoundException e) {
        // handle FileNotFoundException
    } catch (IOException e){
        // handle IOException
    }
}
同一个 catch 也可以捕获多种类型异常，用 | 隔开

private static void readFile(String filePath) {
    try {
        // code
    } catch (FileNotFoundException | UnknownHostException e) {
        // handle FileNotFoundException or UnknownHostException
    } catch (IOException e){
        // handle IOException
    }
}
```

## 自定义异常

习惯上，定义一个异常类应包含两个构造函数，一个无参构造函数和一个带有详细描述信息的构造函数（Throwable 的 toString 方法会打印这些详细信息，调试时很有用）

```java
public class MyException extends Exception {
    public MyException(){ }
    public MyException(String msg){
        super(msg);
    }
    // ...
}
```

## try-catch-finally

当方法中发生异常，异常处之后的代码不会再执行，如果之前获取了一些本地资源需要释放，则需要在方法正常结束时和 catch 语句中都调用释放本地资源的代码，显得代码比较繁琐，finally 语句可以解决这个问题。

```java
private static void readFile(String filePath) throws MyException {
    File file = new File(filePath);
    String result;
    BufferedReader reader = null;
    try {
        reader = new BufferedReader(new FileReader(file));
        while((result = reader.readLine())!=null) {
            System.out.println(result);
        }
    } catch (IOException e) {
        System.out.println("readFile method catch block.");
        MyException ex = new MyException("read file failed.");
        ex.initCause(e);
        throw ex;
    } finally {
        System.out.println("readFile method finally block.");
        if (null != reader) {
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

调用该方法时，读取文件时若发生异常，代码会进入 catch 代码块，之后进入 finally 代码块；若读取文件时未发生异常，则会跳过 catch 代码块直接进入 finally 代码块。所以无论代码中是否发生异常，fianlly 中的代码都会执行。

若 catch 代码块中包含 return 语句，finally 中的代码还会执行吗？将以上代码中的 catch 子句修改如下：

```java
catch (IOException e) {
    System.out.println("readFile method catch block.");
    return;
}
```

调用 readFile 方法，观察当 catch 子句中调用 return 语句时，finally 子句是否执行

```java
readFile method catch block.
readFile method finally block.
```

可见，即使 catch 中包含了 return 语句，finally 子句依然会执行。若 finally 中也包含 return 语句，finally 中的 return 会覆盖前面的 return.

## try-with-resource
上面例子中，finally 中的 close 方法也可能抛出 IOException, 从而覆盖了原始异常。JAVA 7 提供了更优雅的方式来实现资源的自动释放，自动释放的资源需要是实现了 AutoCloseable 接口的类。

```java
private  static void tryWithResourceTest(){
    try (Scanner scanner = new Scanner(new FileInputStream("c:/abc"),"UTF-8")){
        // code
    } catch (IOException e){
        // handle exception
    }
}
```

try 代码块退出时，会自动调用 scanner.close 方法，和把 scanner.close 方法放在 finally 代码块中不同的是，若 scanner.close 抛出异常，则会被抑制，抛出的仍然为原始异常。被抑制的异常会由 addSusppressed 方法添加到原来的异常，如果想要获取被抑制的异常列表，可以调用 getSuppressed 方法来获取。

# 阿里巴巴异常处理规约

## 1、【强制】 Java 类库中定义的可以通过预检查方式规避的 RuntimeException 异常不应该通过catch 的方式来处理，比如：NullPointerException， IndexOutOfBoundsException 等等。

说明：无法通过预检查的异常除外，比如，在解析字符串形式的数字时，不得不通过 catch NumberFormatException 来实现。

正例：if (obj != null) {…}

反例:try { obj.method(); } catch (NullPointerException e) {…}

## 2、【强制】 异常不要用来做流程控制，条件控制。

说明： 异常设计的初衷是解决程序运行中的各种意外情况，且异常的处理效率比条件判断方式要低很多

## 3、【强制】 catch 时请分清稳定代码和非稳定代码，稳定代码指的是无论如何不会出错的代码。对于非稳定代码的 catch 尽可能进行区分异常类型，再做对应的异常处理。

说明： 对大段代码进行 try-catch，使程序无法根据不同的异常做出正确的应激反应，也不利于定位问题，这是一种不负责任的表现。

正例： 用户注册的场景中，如果用户输入非法字符， 或用户名称已存在， 或用户输入密码过于简单，在程序上作出分门别类的判断，并提示给用户。

## 4、【强制】 捕获异常是为了处理它，不要捕获了却什么都不处理而抛弃之，如果不想处理它，请将该异常抛给它的调用者。最外层的业务使用者，必须处理异常，将其转化为用户可以理解的内容。

## 5、【强制】 有 try 块放到了事务代码中， catch 异常后，如果需要回滚事务，一定要注意手动回滚事务。

## 6、【强制】 finally 块必须对资源对象、流对象进行关闭，有异常也要做 try-catch。

说明： 如果 JDK7 及以上，可以使用 try-with-resources 方式。

## 7、【强制】 不要在 finally 块中使用 return。

说明：finally 块中的 return 返回后方法结束执行，不会再执行 try 块中的 return 语句。

## 8、【强制】 捕获异常与抛异常，必须是完全匹配，或者捕获异常是抛异常的父类。

说明： 如果预期对方抛的是绣球，实际接到的是铅球，就会产生意外情况。

## 9、【推荐】 方法的返回值可以为 null，不强制返回空集合，或者空对象等，必须添加注释充分说明什么情况下会返回 null 值。

说明： 本手册明确防止 NPE 是调用者的责任。即使被调用方法返回空集合或者空对象，对调用者来说，也并非高枕无忧，必须考虑到远程调用失败、 序列化失败、 运行时异常等场景返回null 的情况。

## 10、【推荐】 防止 NPE，是程序员的基本修养，注意 NPE 产生的场景：

1. 返回类型为基本数据类型， return 包装数据类型的对象时，自动拆箱有可能产生 NPE。
反例：public int f() { return Integer 对象}， 如果为 null，自动解箱抛 NPE。
2. 数据库的查询结果可能为 null。
3. 集合里的元素即使 isNotEmpty，取出的数据元素也可能为 null。
4. 远程调用返回对象时，一律要求进行空指针判断，防止 NPE。
5. 对于 Session 中获取的数据，建议 NPE 检查，避免空指针。
6. 级联调用 obj.getA().getB().getC()； 一连串调用，易产生 NPE。

正例： 使用 JDK8 的 Optional 类来防止 NPE 问题。

## 11、【推荐】 定义时区分 unchecked / checked 异常，避免直接抛出 new RuntimeException()，更不允许抛出 Exception 或者 Throwable，应使用有业务含义的自定义异常。

推荐业界已定义过的自定义异常，如：DAOException / ServiceException 等。

## 12、【参考】 对于公司外的 http/api 开放接口必须使用“错误码”； 而应用内部推荐异常抛出；跨应用间 RPC 调用优先考虑使用 Result 方式，封装 isSuccess()方法、 “错误码”、 “错误简短信息”。

说明： 关于 RPC 方法返回方式使用 Result 方式的理由：
1） 使用抛异常返回方式，调用方如果没有捕获到就会产生运行时错误。
2） 如果不加栈信息，只是 new 自定义异常，加入自己的理解的 error message，对于调用端解决问题的帮助不会太多。如果加了栈信息，在频繁调用出错的情况下，数据序列化和传输的性能损耗也是问题。

## 13、【参考】 避免出现重复的代码（Don’t Repeat Yourself） ，即 DRY 原则。

说明： 随意复制和粘贴代码，必然会导致代码的重复，在以后需要修改时，需要修改所有的副本，容易遗漏。必要时抽取共性方法，或者抽象公共类，甚至是组件化。

正例： 一个类中有多个 public 方法，都需要进行数行相同的参数校验操作，这个时候请抽取：private boolean checkParam(DTO dto) {…}

# 常见面试题

## 1. Error 和 Exception 区别是什么？

Error 类型的错误通常为虚拟机相关错误，如系统崩溃，内存不足，堆栈溢出等，编译器不会对这类错误进行检测，JAVA 应用程序也不应对这类错误进行捕获，一旦这类错误发生，通常应用程序会被终止，仅靠应用程序本身无法恢复；

Exception 类的错误是可以在应用程序中进行捕获并处理的，通常遇到这种错误，应对其进行处理，使应用程序可以继续正常运行。

## 2. 运行时异常和一般异常区别是什么？

编译器不会对运行时异常进行检测，没有 try-catch，方法签名中也没有 throws 关键字声明，编译依然可以通过。如果出现了 RuntimeException, 那一定是程序员的错误。

一般一场如果没有 try-catch，且方法签名中也没有用 throws 关键字声明可能抛出的异常，则编译无法通过。这类异常通常为应用环境中的错误，即外部错误，非应用程序本身错误，如文件找不到等。

## 3.NoClassDefFoundError 和 ClassNotFoundException 区别？

NoClassDefFoundError 是一个 Error 类型的异常，是由 JVM 引起的，不应该尝试捕获这个异常。

引起该异常的原因是 JVM 或 ClassLoader 尝试加载某类时在内存中找不到该类的定义，该动作发生在运行期间，即编译时该类存在，但是在运行时却找不到了，可能是变异后被删除了等原因导致；

ClassNotFoundException 是一个受查异常，需要显式地使用 try-catch 对其进行捕获和处理，或在方法签名中用 throws 关键字进行声明。当使用 Class.forName, ClassLoader.loadClass 或 ClassLoader.findSystemClass 动态加载类到内存的时候，通过传入的类路径参数没有找到该类，就会抛出该异常；另一种抛出该异常的可能原因是某个类已经由一个类加载器加载至内存中，另一个加载器又尝试去加载它。

## 4. JVM 是如何处理异常的？

在一个方法中如果发生异常，这个方法会创建一个一场对象，并转交给 JVM，该异常对象包含异常名称，异常描述以及异常发生时应用程序的状态。创建异常对象并转交给 JVM 的过程称为抛出异常。可能有一系列的方法调用，最终才进入抛出异常的方法，这一系列方法调用的有序列表叫做调用栈。

JVM 会顺着调用栈去查找看是否有可以处理异常的代码，如果有，则调用异常处理代码。当 JVM 发现可以处理异常的代码时，会把发生的异常传递给它。如果 JVM 没有找到可以处理该异常的代码块，JVM 就会将该异常转交给默认的异常处理器（默认处理器为 JVM 的一部分），默认异常处理器打印出异常信息并终止应用程序。

## 5. throw 和 throws 的区别是什么？

throw 关键字用来抛出方法或代码块中的异常，受查异常和非受查异常都可以被抛出。
throws 关键字用在方法签名处，用来标识该方法可能抛出的异常列表。一个方法用 throws 标识了可能抛出的异常列表，调用该方法的方法中必须包含可处理异常的代码，否则也要在方法签名中用 throws 关键字声明相应的异常。

## 6. 常见的 RuntimeException 有哪些？

ClassCastException(类转换异常)

IndexOutOfBoundsException(数组越界)

NullPointerException(空指针)

ArrayStoreException(数据存储异常，操作数组时类型不一致)

还有IO操作的BufferOverflowException异常

