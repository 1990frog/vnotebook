# mooc

并发是众多框架的原理和基础

Spring中对线程池、单例的应用
数据库中乐观锁思想
Log4j2对阻塞队列的应用

面试技巧：
从一个点引申出并发知识架构，展现知识储备和深刻理解，为面试加分

如何从宏观和微观两个方面提高技术，有哪些途径？
如何了解技术领域的前沿动态？
如何在业务缠身中得到更多成长？
如何分析Java中native层的c/cpp代码？
为何自顶向下学习？

start方法流程
1.检查线程状态，只有NEW状态下的线程才能继续，否则会抛出IllegalThreadStateException（在运行中或者已结束的线程，都不能再次启动，代码演示详见CantStartTwice类）
2.被加入线程组
3.调用start()方法启动线程
注意点：
start方法是被synchronized修饰的方法，可以确保线程安全；
由JVM创建的main方法线程和system组线程，并不会通过start来启动。

native方法（由c或cpp编写）
如果想看源码，去网站看openjdk实际实现

---

停止线程
原理介绍：使用interrupt来通知，而不是强制

Java中停止线程的原则是什么？

在Java中，最好的停止线程的方式是使用中断interrupt，但是这仅仅是会通知到被终止的线程“你该停止运行了”，被终止的线程自身拥有决定权（决定是否、以及何时停止），这依赖于请求停止方和被停止方都遵守一种约定好的编码规范。

任务和线程的启动很容易。在大多数时候,我们都会让它们运行直到结束,或者让它们自行停止。然而,有时候我们希望提前结束任务或线程,或许是因为用户取消了操作,或者服务需要被快速关闭，或者是运行超时或出错了。

要使任务和线程能安全、快速、可靠地停止下来,并不是一件容易的事。Java没有提供任何机制来安全地终止线程。但它提供了中断(Interruption),这是一种协作机制,能够使一个线程终止另一个线程的当前工作。

这种协作式的方法是必要的，我们很少希望某个任务、线程或服务立即停止,因为这种立即停止会使共享的数据结构处于不一致的状态。相反,在编写任务和服务时可以使用一种协作的方式:当需要停止时,它们首先会清除当前正在执行的工作,然后再结束。这提供了更好的灵活性,因为任务本身的代码比发出取消请求的代码更清楚如何执行清除工作。

生命周期结束(End-of-Lifecycle)的问题会使任务、服务以及程序的设计和实现等过程变得复杂,而这个在程序设计中非常重要的要素却经常被忽略。一个在行为良好的软件与勉强运的软件之间的最主要区别就是,行为良好的软件能很完善地处理失败、关闭和取消等过程。
本章将给出各种实现取消和中断的机制,以及如何编写任务和服务,使它们能对取消请求做出响应。

中断sleep应该处理异常

实际开发中的两种最佳实践

优先选择：传递中断
不想或无法传递：恢复中断
不应屏蔽中断

添加异常到方法签名

处理中断的最好方法是什么？
优先选择在方法上抛出异常。
用throws InterruptedException 标记你的方法，不采用try 语句块捕获异常，以便于该异常可以传递到顶层，让run方法可以捕获这一异常，例如：

```
void subTask() throws InterruptedException
    sleep(delay);
}
```
由于run方法内无法抛出checked Exception（只能用try catch），顶层方法必须处理该异常，避免了漏掉或者被吞掉的情况，增强了代码的健壮性。

如果不能抛出中断，要怎么做？
如果不想或无法传递InterruptedException（例如用run方法的时候，就不让该方法throws InterruptedException），那么应该选择在catch 子句中调用Thread.currentThread().interrupt() 来恢复设置中断状态，以便于在后续的执行依然能够检查到刚才发生了中断。
代码演示详见视频，在这里，线程在sleep期间被中断，并且由catch捕获到该中断，并重新设置了中断状态，以便于可以在下一个循环的时候检测到中断状态，正常退出。

响应中断的方法总结列表
Object.wait()/wait(long)/wait(long,int)
Thread.sleep(long)/sleep(long,int)
Thread.join()/join(long)/join(long,int)

java.util.concurrent.BlockingQueue.take()/put(E)
java.util.concurrent.locks.Lock.lockInterruptibly()
java.util.concurrent.CountDownLatch.await()
java.util.concurrent.CyclicBarrier.await()
java.util.concurrent.Exchanger.exchange(V)

java.nio.channels.InterruptibleChannel相关方法
java.nio.channels.Selector的相关方法

为什么要用interrupt来停止线程：
被中断的线程它本身拥有响应中断的权利，因为有些线程的某些代码是非常重要的，我们必须要等待这些线程处理完成之后或者他们准备好之后，再由他们自己去主动终止，或者它们真的不想理会我们的中断这也是完全ok的，我们不应该茹莽的使用stop方法，而是通过使用interrupt方法来发出一个信号，由它们自己处理，这样使我们的线程代码，在实际中更加安全，也完成了清理工作，数据的完整性也得到了保障

![](_v_images/20190903225518559_2071670296.png)

非受检查异常：

Error:  
VirtualMachineError
AWTError

Exception:
RuntimeException

受检查异常：
IOException

错误停止方法：
1.被弃用的stop,suspend和resume方法
2.用volatile设置boolean标记位

如何分析native方法
进入github（也可以进入 openJDK网站 ）
点“搜索文件”，搜索对应的c代码类Thread.c
找到native方法对应的方法名
去src/hotspot/share/prims/jvm.cpp里看cpp代码

![](_v_images/20190905205555461_716593482.png)

停止线程相关重要函数解析

判断是否已被中断相关方法
static boolean interrupted()
```
return isInterrupted(true);
返回中断状态，并清除中断状态
```
作用对象是当前运行它的线程，执行线程，该方法不关心被哪个对象调用，只关心执行它的线程

boolean isInterrupted()
```
return isInterrupted(false);
返回中断状态
```
Thread.interrupted()的目的对象
Thread可看做this

停止线程：

如何停止线程
1.原理：用interrupt来请求、好处
2.想停止线程，要请求方、被停止方、子方法被调用方相互配合
3.最后再说错误的方法：stop/suspend已废弃，volatile的boolean无法处理长时间阻塞的情况

如何处理不可中断的阻塞
并没有通用的方法，针对特定的情况，使用特定的方法
使用可以响应中断的方法

线程的一生——6个状态（生命周期）
有哪6种状态 ？
每个状态是什么含义？
状态间的转化图示
阻塞状态是什么？

6种状态：
1.New：已创建，但没启动，未执行Run方法
2.Runnable：调用了start方法，未分配资源也是此状态，Runnable可运行的，也可能是已运行中的
3.Blocked：进入synchronize修饰的代码块，仅synchronize
4.Waiting：
5.Timed Waiting
6.Terminated

![](_v_images/20190905212431539_634278368.png =522x)