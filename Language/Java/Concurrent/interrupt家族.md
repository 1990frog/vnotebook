# interrupt&interrupted$isInterrupted

## interrupt

中断线程

## interrupted

重置中断状态
只能获取调用该方法的线程状态

### 源码

```java
/**
 * Tests whether the current thread has been interrupted.  The
 * <i>interrupted status</i> of the thread is cleared by this method.  In
 * other words, if this method were to be called twice in succession, the
 * second call would return false (unless the current thread were
 * interrupted again, after the first call had cleared its interrupted
 * status and before the second call had examined it).
 *
 * <p>A thread interruption ignored because a thread was not alive
 * at the time of the interrupt will be reflected by this method
 * returning false.
 *
 * @return  <code>true</code> if the current thread has been interrupted;
 *          <code>false</code> otherwise.
 * @see #isInterrupted()
 * @revised 6.0
 */
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}
```

### 例子1：

```java
Thread.currentThread().interrupt();
System.out.println(Thread.currentThread().isInterrupted());
System.out.println(Thread.interrupted());//清除中断状态
System.out.println(Thread.interrupted());
```
输出结果：

```txt
true（中断操作）
true（清除中断状态）
false（清除中断状态）
```

### 例子2：

```java
//Thread.currentThread().interrupt();
System.out.println(Thread.currentThread().isInterrupted());
System.out.println(Thread.interrupted());//清除中断状态
System.out.println(Thread.interrupted());
```

输出结果：

```txt
false
false
false
```

## isInterrupted

获取线程中断状态
