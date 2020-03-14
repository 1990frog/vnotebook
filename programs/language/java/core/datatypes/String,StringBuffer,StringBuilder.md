[TOC]

# String
String的值是不可变的，这就导致每次对String的操作都会生成新的String对象，这样不仅效率低下，而且大量浪费有限的内存空间。

JVM的常量池最多可放65535个项（2^16）

为啥不能修改那就去看源码，下面
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```
# StringBuilder与StringBuffer
StringBuffer和StringBuilder类的对象能够被多次的修改，并且不产生新的未使用对象。
![test](_v_images/20200307201745270_1283649617.png)
他们都是实现自AbstractStringBuilder类

为啥他们可以被多次修改呢？看下面源码
```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;
}
```
## 区别
+ StringBuilder非线程安全的
+ StringBuffer线程安全的（裹了一个synchronized的渣渣）

```java
/**
 * printout:
 * StringBuffer:1000000
 * AtomicInteger:1000000
 * StringBuilder:981663
 * int:994087
 */
public class StringBufferThreadTest implements Runnable {

    private static StringBuffer sb = new StringBuffer();
    private static StringBuilder sbr = new StringBuilder();
    private static int a;
    private static AtomicInteger b = new AtomicInteger();

    @Override
    public void run() {
        for (int i = 0; i < 100000; i++) {
            sb.append(0);
            sbr.append(0);
            a++;
            b.incrementAndGet();
        }
    }

    public static void main(String[] args) {

        StringBufferThreadTest stringBufferThreadTest = new StringBufferThreadTest();
        ExecutorService executor = Executors.newFixedThreadPool(10);

        for (int i = 0; i < 10; i++) {
            executor.execute(stringBufferThreadTest);
        }
        executor.shutdown();

        while (!executor.isTerminated()){}

        System.out.println("StringBuffer:"+sb.length());
        System.out.println("AtomicInteger:"+b);
        System.out.println("StringBuilder:"+sbr.length());
        System.out.println("int:"+a);

    }
}
```
