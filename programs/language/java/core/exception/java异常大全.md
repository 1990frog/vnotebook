我们在进行JAVA项目开发时，经常会面对一些常见的异常处理情况，接下来我会根据课下查阅学习的资料，来进行一些总结。

　　1.空指针异常（java.lang.nullpointerexception）

发生该情况一般是字符串变量未初始化，数组未初始化，类对象未初始化等。还有一种情况是当该对象为空时你并没有判断是否为空值，这个错误我在之前的web习题上犯过，因此为了避免这种情况，除了检查是否初始化之外，如有必要则要加上判断是否为null的if语句。

　　2.指定的类不存在（java.lang.ClassNotFoundException）

出现这个错误的原因之一是缺包，这时只要下载并导入相应的包即可；当我们已经把包导入的时候，又报了这种错误的情况下，就需要开启自己的编辑器去调整设置了；在使用tomcat的时候，先检查lib中是否导入了jar。

　　3.字符串转换为数字异常（java.lang.NumberFormatException）

这个错误就是字符串中出现非数字型字符时，转换为数字时发生异常；除此之外，如果字符串转换为数字时超过了类型的范围（比如string转int和string转double），也会出现这个错误。解决该问题的方法就是在转换之前先对字符串进行检查。

　　4.数组下标越界异常（java.lang.IndexOutOfBoundsException）

顾名思义，你想取的数组元素在数组中并没有定义出来，比如定义了一个长度为5的数组a，当你想取a[6]元素时肯定会出错。解决这类问题就是要注意数组的长度，有时候为了减少空间浪费我们会使用动态数组构建方法，这时在对数组进行操作时建议先用length获取其数组长度，从而规避错误。

　　5.数学运算异常（java.lang.ArithmeticException）

除数为0时会报出该错误，解决方法：避免除数为0。这个错误解读为“出现异常的运算条件”，除了除数为0的情况之外，可能还有其他的异常情况，届时具体情况具体分析。

　　6.没有访问权限（java.lang.IllegalAccessException）

权限问题，在程序访问某方法时注意一下访问权限即可（public/private），这种错误在使用package时容易发生。

　　7.方法的参数错误（java.lang.IllegalArgumentException）

在调用带有参数的方法时，请注意传递的参数是否正确。

　　8.数据类型转换异常（java.lang.ClassCastException）

在进行强制类型转换时容易发生该错误，在进行转换前先对类型进行判别，规避错误。

　　9.文件未找到异常（java.lang.FileNotFoundException）

当程序试图打开一个不存在的文件进行读写操作时会报出该错误，通常由FileInputStream,FileOutputStream,RandomAccessFile的构造器声明发出，即使文件存在，但因某个原因无法访问，也会报出该错误。

　　10.数组存储异常（java.lang.ArrayStoreException）

假如在int型数组里存入string类型的变量，就会报错，解决方案是在存入对象时查明类型，或者在存入前先进行类型转换。

　　11.方法不存在异常（java.lang.NoSuchMethodException）

程序所要调用的方法不存在。解决方法：不调用或者构造其方法。

　　12.文件已结束异常（java.lang.EOFException）

程序输入过程中遇到文件或流的结尾引发该异常，此异常用于检查是否达到文件或流结尾。

　　13.实例化异常（java.lang.InstantiationException）

类创建新对象时无法通过构造器进行实例化引发的异常。解决方案：构造方法。

　　14.被中止异常（java.lang.InterruptedException）

通过其他线程的Thread的interrupt方法中止另一个线程时报出的错误。解决方法：一，不做处理，直接抛出；二，捕获异常，再次调用interrupt方法，将中断状态重新设置为true。

　　15.不支持克隆异常（java.lang.CloneNotSupportedException）

如果没有实现Cloneable接口便调用了clone方法，报出该错误；若类不支持Cloneable接口，调用时也会出现该错误。解决方法：实现Cloneable接口。

　　16.输入输出异常（IOException）

该异常为Exception的一个分支，通常发生在文件的数据读写上。

　　17.错误（java.lang.Error）

所有错误的基类，用于标识严重的程序运行问题。通常原因是访问外部资源时出现一系列问题，解决方案也需要围绕访问外部资源这一重点展开。

