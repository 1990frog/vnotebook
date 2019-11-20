Lambda表达式又称为闭包或匿名方法，形式如下：

(int x, int y) -> x-y；

( ) -> 2；

(String s) -> { System.out.println(s); }；

第一个lambda表达式接收x和y这两个整形参数并返回它们的差；第二个lambda表达式不接收任何参数，直接返回整数2；第三个lambda表达式接收一个字符串并把它打印到控制台。

从上面的例子看，lambda表达式的语法结构和函数几乎是一样的，只是没有函数名。它由参数列表、箭头符号和函数体组成。当然函数体既可以是一个表达式，也可以是一个大括号包围的语句块。

从例子看，Lambda表达式语法确实相当简洁。

那么它的意义何在呢？

Lambda表达式的意义：

对程序员来说最直观的感受就是用Lambda表达式可以简化很多代码。使用它可以很轻松的将很多行代码缩减成一行。

而更大的意义是，Lambda表达式对java性能的提升。

要理解Lambda的意义就先看看Lambda解决了Java里的什么困惑
在java和android里常会看到这种回调接口：

public interface ActionListener{

void actionPerformed(ActionEvent e);

}

使用的时候，要么我们定义一个类来实现ActionListener接口，要么使用匿名内部类。

button.addActionListener(new ActionListener(){ 

public void actionPerformed(ActionEvent e) {

ShowDialog(e.tostring());

}

})

放个概念：函数式接口：只有一个方法的接口。大多数回调接口都拥有这个特征：比如Runnable接口和Comparator接口。

这个ActionListener回调接口例子里的五行代码中仅有一行在做实际工作。冗余代码太多，是写代码层面直观的效率感受。

同时，这种通过匿名内部类的方式也不是一个好的方案。匿名内部类存在着影响应用性能的问题。

对于匿名内部类，编译器会为每一个匿名内部类创建相应的类文件。一般的程序，往往回调接口会有很多。这样就会生成很多的类文件，因为类在使用之前需要加载类文件并进行验证，这个过程就会影响应用的性能。

然后，java就引入Lambda表达式了。上面那个例子，用Lambda表达式后变成如下：

button.addActionListener((ActionEvent e) -> ShowDialog(e.tostring()));

明显代码简洁很多。

在这种情况下，编译器会负责推导这个lambda表达式具体是对应哪个函数式接口类型。注意：lambda表达式只能出现在目标类型为函数式接口的上下文中。

这是Lambda表达式在代码简洁方面的意义。

在效率提升方面的意义是通过把函数式接口的匿名内部类方法的调用改成类静态方法调用来完成的。

编译器在将Lambda表达式转成函数式接口实例时，分析这个方法体是否需要变量捕获（Lambda方法体需要访问外部变量时则为需变量捕获，反之则为变量不捕获）。对于无需变量捕获的Lambda，其方法体会被提取到一个静态方法中，这个静态方法和Lambda表达式位于同一个类中。对于需要变量捕获的Lambda表达式情况有点复杂，最后也会把Lambda表达式提取到一个静态方法中。也就是Lambda表达式虽然推导到相应的函数式接口，但并没有编译成匿名内部类的方式来处理，而做成是直接方法调用。

然后就是，在大多数情况下，Lambda表达式要比匿名内部类性能更优。
