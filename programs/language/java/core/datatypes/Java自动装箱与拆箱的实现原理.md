[TOC]

# 什么是自动装箱和拆箱
自动装箱就是Java自动将原始类型值转换成对应的对象，比如将int的变量转换成Integer对象，这个过程叫做装箱，反之将Integer对象转换成int类型值，这个过程叫做拆箱。因为这里的装箱和拆箱是自动进行的非人为转换，所以就称作为自动装箱和拆箱。原始类型byte, short, char, int, long, float, double 和 boolean 对应的封装类为Byte, Short, Character, Integer, Long, Float, Double, Boolean。

下面例子是自动装箱和拆箱带来的疑惑
```java
public class Test {  
    public static void main(String[] args) {      
        test();  
    }  

    public static void test() {  
        int i = 40;  
        int i0 = 40;  
        Integer i1 = 40;  
        Integer i2 = 40;  
        Integer i3 = 0;  
        Integer i4 = new Integer(40);  
        Integer i5 = new Integer(40);  
        Integer i6 = new Integer(0);  
        Double d1=1.0;  
        Double d2=1.0;  
          
        System.out.println("i=i0\t" + (i == i0));  
        System.out.println("i1=i2\t" + (i1 == i2));  
        System.out.println("i1=i2+i3\t" + (i1 == i2 + i3));  
        System.out.println("i4=i5\t" + (i4 == i5));  
        System.out.println("i4=i5+i6\t" + (i4 == i5 + i6));      
        System.out.println("d1=d2\t" + (d1==d2));   
          
        System.out.println();          
    }  
} 
```
输出的结果：
```java
i=i0        true
i1=i2       true
i1=i2+i3    true
i4=i5       false
i4=i5+i6    true
d1=d2     false
```
# 自动装箱和拆箱的原理
自动装箱时编译器调用valueOf将原始类型值转换成对象，同时自动拆箱时，编译器通过调用类似intValue(),doubleValue()这类的方法将对象转换成原始类型值。

明白自动装箱和拆箱的原理后，我们带着上面的疑问进行分析下Integer的自动装箱的实现源码。如下：
```java

public static Integer valueOf(int i) {
    //判断i是否在-128和127之间，存在则从IntegerCache中获取包装类的实例，否则new一个新实例
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}


//使用亨元模式，来减少对象的创建（亨元设计模式大家有必要了解一下，我认为是最简单的设计模式，也许大家经常在项目中使用，不知道他的名字而已）
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    //静态方法，类加载的时候进行初始化cache[],静态变量存放在常量池中
    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```
`Integer i1 = 40`自动装箱，相当于调用了`Integer.valueOf(40)`方法。
    首先判断i值是否在-128和127之间，如果在-128和127之间则直接从IntegerCache.cache缓存中获取指定数字的包装类；不存在则new出一个新的包装类。
    IntegerCache内部实现了一个Integer的静态常量数组，在类加载的时候，执行static静态块进行初始化-128到127之间的Integer对象，存放到cache数组中。cache属于常量，存放在java的方法区中。
    如果你不了解方法区请点击这里查看JVM内存模型

接着看下面是java8种基本类型的自动装箱代码实现。如下：
```java
//boolean原生类型自动装箱成Boolean
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}

//byte原生类型自动装箱成Byte
public static Byte valueOf(byte b) {
    final int offset = 128;
    return ByteCache.cache[(int)b + offset];
}

//byte原生类型自动装箱成Byte
public static Short valueOf(short s) {
    final int offset = 128;
    int sAsInt = s;
    if (sAsInt >= -128 && sAsInt <= 127) { // must cache
        return ShortCache.cache[sAsInt + offset];
    }
    return new Short(s);
}

//char原生类型自动装箱成Character
public static Character valueOf(char c) {
    if (c <= 127) { // must cache
        return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}

//int原生类型自动装箱成Integer
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}

//int原生类型自动装箱成Long
public static Long valueOf(long l) {
    final int offset = 128;
    if (l >= -128 && l <= 127) { // will cache
        return LongCache.cache[(int)l + offset];
    }
    return new Long(l);
}

//double原生类型自动装箱成Double
public static Double valueOf(double d) {
    return new Double(d);
}

//float原生类型自动装箱成Float
public static Float valueOf(float f) {
    return new Float(f);
}
```
通过分析源码发现，只有double和float的自动装箱代码没有使用缓存，每次都是new 新的对象，其它的6种基本类型都使用了缓存策略。
使用缓存策略是因为，缓存的这些对象都是经常使用到的（如字符、-128至127之间的数字），防止每次自动装箱都创建一次对象的实例。
而double、float是浮点型的，没有特别的热的（经常使用到的）数据的，缓存效果没有其它几种类型使用效率高。

下面在看下装箱和拆箱问题解惑。
```java
//1、这个没解释的就是true
System.out.println("i=i0\t" + (i == i0));  //true
//2、int值只要在-128和127之间的自动装箱对象都从缓存中获取的，所以为true
System.out.println("i1=i2\t" + (i1 == i2));  //true
//3、涉及到数字的计算，就必须先拆箱成int再做加法运算，所以不管他们的值是否在-128和127之间，只要数字一样就为true
System.out.println("i1=i2+i3\t" + (i1 == i2 + i3));//true  
//比较的是对象内存地址，所以为false
System.out.println("i4=i5\t" + (i4 == i5));  //false
//5、同第3条解释，拆箱做加法运算，对比的是数字，所以为true
System.out.println("i4=i5+i6\t" + (i4 == i5 + i6));//true      
//double的装箱操作没有使用缓存，每次都是new Double，所以false
System.out.println("d1=d2\t" + (d1==d2));//false
```
相信你看到这就应该能明白上面的程序输出的结果为什么是true,false了，只要掌握原理，类似的问题就迎刃而解了，如果这篇文章对您有帮助，就加下关注，点个赞。

