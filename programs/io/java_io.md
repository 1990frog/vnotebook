![io](_v_images/20191121102922535_1790976369.jpg)

以Reader这块来讲解装饰器模式在IO中的应用

+ 抽象类
Reader，是一个抽象类，里面定义了接口
+ 被装饰的类
InputStreamReader、CharArrayReader、PipedReader、StringReader、FileReader
+ 装饰器类
FilterReader、BufferedReader
+ 具体的装饰器类
BufferedReader、PushbackReader
FileReader尽管也在第三行，但是FileReader构不成一个具体的装饰器类，因为它不是BufferedReader的子类也不是FilterReader的子类，不持有Reader的引用
读写IO的时候每次打开文件会非常耗时，我们可以使用BufferedReader装饰FileReader这些被装饰器类，使用缓冲提高性能


![](_v_images/20191121165156097_391536508.png)

![](_v_images/20191121165216749_2002889243.png)

![](_v_images/20191121165233356_463152762.png)

![](_v_images/20191121165252172_462926785.png)

![IO系统](_v_images/20191121165537828_343432901.png)