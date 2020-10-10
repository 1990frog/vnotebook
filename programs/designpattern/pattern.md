
设计模式的一个重要原则就是：
别改代码，只需要添代码，以前所有的老代码，都是有价值的，需要尽力保留new一个对象时，new的过程是宝贵的如何创建老对象的知识点（有的new很复杂，包括了很多参数），如果这个代码被修改了，那么保留的老对象也不知道怎么使用了，整个体系残缺了所以要想办法保留老对象的new过程，把这个new过程保存分布到一系列工厂类里，就是所谓的工厂模式，一般有三种方式来封装简单工厂：把对象的创建放到一个工厂类中，通过参数来创建不同的对象。这个缺点是每添一个对象，就需要对简单工厂进行修改（尽管不是删代码，仅仅是添一个switch case，但仍然违背了“不改代码”的原则）工厂方法：每种产品由一种工厂来创建，一个工厂保存一个new基本完美，完全遵循 
“不改代码”的原则抽象工厂：仅仅是工厂方法的复杂化，保存了多个new大工程才用的上


GoF (Gang of Four，四人组， 《Design Patterns: Elements of Reusable Object-Oriented Software》/《设计模式》一书的作者：Erich Gamma、Richard Helm、Ralph Johnson、John Vlissides)并没有把MVC提及为一种设计模式，而是把它当做“一组用于构建用户界面的类集合”。在他们看来，它其实是其它三个经典的设计模式的演变：观察者模式(Observer)(Pub/Sub), 策略模式(Strategy)和组合模式(Composite)。根据MVC在框架中的实现不同可能还会用到工厂模式(Factory)和装饰器(Decorator)模式。我在另一本免费的书“JavaScript Design Patterns For Beginners”中讲述了这些模式，如果你有兴趣可以阅读更多信息。正如我们所讨论的，models表示应用的数据，而views处理屏幕上展现给用户的内容。为此，MVC在核心通讯上基于推送/订阅模型(惊讶的是 在很多关于MVC的文章中并没有提及到)。当一个model变化时它对应用其它模块发出更新通知(“publishes”)，订阅者 (subscriber)——通常是一个Controller，然后更新对应的view。观察者——这种自然的观察关系促进了多个view关联到同一个 model。对于感兴趣的开发人员想更多的了解解耦性的MVC(根据不同的实现)，这种模式的目标之一就是在一个主题和它的观察者之间建立一对多的关系。当这个 主题改变的时候，它的观察者也会得到更新。Views和controllers的关系稍微有点不同。Controllers帮助views对不同用户的输 入做不同的响应，是一个非常好的策略模式列子。嗯嗯，知道为什么MVC没有被GOF当作【一种】模式来对待了吧？因为它实际上是三种模式的合体！

![设计模式](https://gitee.com/caijingquan/imagebed/raw/master/1602317308_20191121170024326_475589840.png)