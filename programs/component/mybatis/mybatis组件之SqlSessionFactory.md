[TOC]

一旦被创建，SqlSessionFactory实例应该在你的应用程序执行期间都存在。没有理由来处理或重新创建它。
使用SqlSessionFactory的最佳实践是在应用程序运行期间不要重复创建多次。这样的操作将被视为是非常糟糕的。
因此SqlSessionFactory的最佳范围是应用范围。有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。
然而这两种方法都不认为是最佳实践。
这样的话，你可以考虑依赖注入容器，比如Google Guice或Spring。这样的框架允许你创建支持程序来管理单例SqlSessionFactory的生命周期。

