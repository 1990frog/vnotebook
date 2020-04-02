# java常见面试题


24、Static Nested Class 和 Inner Class的不同。
Static Nested Class是被声明为静态（static）的内部类，它可以不依赖于外部类实例被实例化。而通常的内部类需要在外部类实例化后才能实例化。

28、short s1 = 1; s1 = s1 + 1;有什么错? short s1 = 1; s1 += 1;有什么错?
short s1 = 1; s1 = s1 + 1; （s1+1运算结果是int型，需要强制转换类型）
short s1 = 1; s1 += 1;（可以正确编译）

29、Math.round(11.5)等於多少? Math.round(-11.5)等於多少?
Math.round(11.5)==12
Math.round(-11.5)==-11
round方法返回与参数最接近的长整数，参数加1/2后求其floor.

30、String s = new String("xyz");创建了几个String Object?
两个

35、List, Set, Map是否继承自Collection接口
List，Set是，Map不是

44、编程题: 用最有效率的方法算出2乘以8等於几?
2 << 3

45、两个对象值相同(x.equals(y) == true)，但却可有不同的hash code，这句话对不对?
不对，有相同的hash code。

46、当一个对象被当作参数传递到一个方法后，此方法可改变这个对象的属性，并可返回变化后的结果，那么这里到底是值传递还是引用传递?
是值传递。Java 编程语言只有值传递参数。当一个对象实例作为一个参数被传递到方法中时，参数的值就是对该对象的引用。对象的内容可以在被调用的方法中改变，但对象的引用是永远不会改变的。


54、描述一下JVM加载class文件的原理机制?
JVM中类的装载是由ClassLoader和它的子类来实现的,Java ClassLoader 是一个重要的Java运行时系统组件。它负责在运行时查找和装入类文件的类。

92、j2ee常用的设计模式？说明工厂模式。
Java中的23种设计模式：
Factory（工厂模式），      Builder（建造模式），       Factory Method（工厂方法模式），
Prototype（原始模型模式），Singleton（单例模式），    Facade（门面模式），
Adapter（适配器模式），    Bridge（桥梁模式），        Composite（合成模式），
Decorator（装饰模式），    Flyweight（享元模式），     Proxy（代理模式），
Command（命令模式），      Interpreter（解释器模式）， Visitor（访问者模式），
Iterator（迭代子模式），   Mediator（调停者模式），    Memento（备忘录模式），
Observer（观察者模式），   State（状态模式），         Strategy（策略模式），
Template Method（模板方法模式）， Chain Of Responsibleity（责任链模式）
工厂模式：工厂模式是一种经常被使用到的模式，根据工厂模式实现的类可以根据提供的数据生成一组类中某一个类的实例，通常这一组类有一个公共的抽象父类并且实现了相同的方法，但是这些方法针对不同的数据进行了不同的操作。首先需要定义一个基类，该类的子类通过不同的方法实现了基类中的方法。然后需要定义一个工厂类，工厂类可以根据条件生成不同的子类实例。当得到子类的实例后，开发人员可以调用基类中的方法而不必考虑到底返回的是哪一个子类的实例。

94、排序都有哪几种方法？请列举。用JAVA实现一个快速排序。
排序的方法有：插入排序（直接插入排序、希尔排序），交换排序（冒泡排序、快速排序），选择排序（直接选择排序、堆排序），归并排序，分配排序（箱排序、基数排序）
