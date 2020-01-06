# 什么是线程安全性
当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要任额外的同步或协同，这个类都能表现出正确的行为，那么就称这个类是线程安全的。

无状态的对象一定是线程安全的

竞态条件（Race Condition）
竞争冒险（race hazard）又名竞态条件、竞争条件（race condition），它旨在描述一个系统或者进程的输出依赖于不受控制的事件出现顺序或者出现时机。此词源自于两个信号试着彼此竞争，来影响谁先输出。
举例来说，如果计算机中的两个进程同时试图修改一个共享内存的内容，在没有并发控制的情况下，最后的结果依赖于两个进程的执行顺序与时机。而且如果发生了并发访问冲突，则最后的结果是不正确的。
竞争冒险常见于不良设计的电子系统，尤其是逻辑电路。但它们在软件中也比较常见，尤其是有采用多线程技术的软件。
最常见的静态条件类型就是“先检查后执行（Check-Then-Act）”操作，即通过一个可能失效的观测结果来决定下一步的动作。

要避免竞态条件问题，就必须在某个线程修改该变量时，通过某种方式防止其他线程使用这个状态，从而确保其他线程只能在修改操作完成之前或之后读取和修改状态，而不是在修改状态的过程中。

Operations A and B are atomic with respect to each other if, from the perspective of a thread executing A, when another thread executes B, either all of B has executed or none of it has. An atomic operation is one that is atomic with respect to all operations, including itself, that operate on the same state.

数据竞争（Data Race）
数据竞争是指，如果在访问共享的非final类型的域时没有采用同步来进行协同，那么就会出现数据竞争。当一个线程写入一个变量而另一个线程接下来读取这个变量，或者读取一个之前由另一个线程写入的变量时，并且在这两个线程之间没使用同步，那么就可能出现数据竞争。

---

获取锁是通过轮循？

Reentrancy重入
嵌套synchronized锁