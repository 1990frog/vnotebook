[TOC]

# Runnable的缺陷
+ 不能返回一个返回值
+ 在run方法中无法抛出checked Exception（为什么设计成这样？运行在Thread类里面，而在Thread类里面我们不能编码）

# Callable接口
+ 类似于Runnable，被其它线程执行的任务
+ 实现call方法
+ 有返回值

Callable接口源码

# Callable和Future的关系
+ 我们可以用Future.get来获取Callable接口返回的执行结果，还可以通过Future.isDone()来判断任务是否已经执行完了，以及取消这个任务，限时获取任务的结果等
+ 在call()未执行完毕之前，调用get()的线程（假定此时是主线程）会被阻塞，直到call()方法返回了结果后，此时future.get()才会得到该结果，然后主线程才会切换到Runnable状态
+ 所以Future是一个存储器，它存储了call()这个任务的结果，而这个任务的执行时间是无法提前确定的，因为这完全取决于call()方法执行的情况

# Future类（阻塞）

## get()/get(long timeout,TimeUnit unit)方法
get方法的行为取决于Callable任务的状态，只有以下5种情况：
1. 任务正常完成：get方法会立刻返回结果
2. 任务尚未完成（任务还没开始或进行中）：get将阻塞并直到任务完成
3. 人执行过程中抛出Exception：get方法会抛出ExecutionException，这里的抛出异常，是call()执行时产生的那个异常，看到这个异常类型是java.util.concurrent.ExecutionException。
4. 任务被取消：get方法会抛出CancellationException
5. 任务超时：get方法有一个重载方法，是传入一个延迟时间的，如果时间到了还没有获得结果，get方法就会抛出TimeoutException。
## cancel()方法

## isDown()方法
判断线程是否执行完毕（执行成功、失败都算完毕）

## isCancelled()方法
是否取消


# 用法1：线程池的submit方法返回Future对象
首先，我们要给线程池提交我们的任务，提交时线程池会立刻返回给我们一个空的Future容器。当线程的人一旦执行完毕，也就是当我们可以获取结果的时候，线程池便会把该结果填入到之前给我们的哪个Future中去（而不是创建一个新的Future），我们此时便可以从该Future中获得任务执行的结果

idea快捷键：
shift+alt+v：生成变量名
ctrl+alt+f：抽取成员变量

# 用法2：lumbda表达式形式

多个任务，用Future数组来获取结果

get返回的异常类型？
超时
中断
ExecutedException

# cancel方法：取消任务的执行
1. 如果这个任务还没有开始执行，那么这种情况最简单，任务会被正常的取消，未来也不会被执行，方法返回true
2. 如果任务已完成，或者已取消：那么cancel()方法会执行失败，方法返回false
3. 如果这个任务已经开始执行了，那么这个取消方法将不会直接取消该任务，而是会根据我们填的参数mayInterruptIfRunning做判断

# cancel方法：取消任务的执行
Future.cancel(true)适用于：
1. 任务能够处理interrupt
Future.cancel(false)仅用于避免启动尚未启动的任务，适用于：
1. 未能处理interrupt的任务
2. 不清楚任务是否支持取消
3. 需要等待已经开始的任务执行完毕


