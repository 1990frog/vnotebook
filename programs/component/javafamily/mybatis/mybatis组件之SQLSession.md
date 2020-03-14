[TOC]

每个线程都应该有它自己的SqlSession实例。

SqlSession的实例不能共享使用，它也是线程不安全的。

因此最佳的范围是请求或方法范围。

绝对不能将SqlSession实例的引用放在一个类的静态字段甚至是实例字段中。

也绝不能将SqlSession实例的引用放在任何类型的管理范围中，比如Serlvet架构中的HttpSession。

如果你现在正用任意的Web框架，要考虑SqlSession放在一个和HTTP请求对象相似的范围内。

换句话说，基于收到的HTTP请求，你可以打开了一个SqlSession，然后返回响应，就可以关闭它了。

关闭Session很重要，你应该确保使用finally块来关闭它。

下面的示例就是一个确保SqlSession关闭的基本模式：
```java
SqlSession session = sqlSessionFactory.openSession();
try {
// do work
} finally {
    session.close();
}
```
在你的代码中一贯地使用这种模式，将会保证所有数据库资源都正确地关闭（假设你没有通过你自己的连接关闭，这会给MyBatis造成一种迹象表明你要自己管理连接资源）。

