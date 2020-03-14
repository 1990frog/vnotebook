加锁和解锁的原理：深入JVM看字节码

```java
/**
 * 反编译字节码
 */
public class Decompilation14 {
    private Object object = new Object();

    public void insert(Thread thread){
        synchronized (object){

        }
    }
}
```
javac o.java//编译
javap -verbose o.class//反汇编

```java
Classfile /home/cai/Code/Java/study/java8/src/main/java/src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14.class
  Last modified 2019-10-28; size 533 bytes
  MD5 checksum 7fa3db82ee15d6b10466d4e604784e9b
  Compiled from "Decompilation14.java"
public class src.main.java.concurrency.moocwukong.synchronizedlock.Decompilation14
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #2.#20         // java/lang/Object."<init>":()V
   #2 = Class              #21            // java/lang/Object
   #3 = Fieldref           #4.#22         // src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14.object:Ljava/lang/Object;
   #4 = Class              #23            // src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14
   #5 = Utf8               object
   #6 = Utf8               Ljava/lang/Object;
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               insert
  #12 = Utf8               (Ljava/lang/Thread;)V
  #13 = Utf8               StackMapTable
  #14 = Class              #23            // src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14
  #15 = Class              #24            // java/lang/Thread
  #16 = Class              #21            // java/lang/Object
  #17 = Class              #25            // java/lang/Throwable
  #18 = Utf8               SourceFile
  #19 = Utf8               Decompilation14.java
  #20 = NameAndType        #7:#8          // "<init>":()V
  #21 = Utf8               java/lang/Object
  #22 = NameAndType        #5:#6          // object:Ljava/lang/Object;
  #23 = Utf8               src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14
  #24 = Utf8               java/lang/Thread
  #25 = Utf8               java/lang/Throwable
{
  public src.main.java.concurrency.moocwukong.synchronizedlock.Decompilation14();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: new           #2                  // class java/lang/Object
         8: dup
         9: invokespecial #1                  // Method java/lang/Object."<init>":()V
        12: putfield      #3                  // Field object:Ljava/lang/Object;
        15: return
      LineNumberTable:
        line 6: 0
        line 7: 4

  public void insert(java.lang.Thread);
    descriptor: (Ljava/lang/Thread;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=2
         0: aload_0
         1: getfield      #3                  // Field object:Ljava/lang/Object;
         4: dup
         5: astore_2
         6: monitorenter
         7: aload_2
         8: monitorexit
         9: goto          17
        12: astore_3
        13: aload_2
        14: monitorexit
        15: aload_3
        16: athrow
        17: return
      Exception table:
         from    to  target type
             7     9    12   any
            12    15    12   any
      LineNumberTable:
        line 10: 0
        line 12: 7
        line 13: 17
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 12
          locals = [ class src/main/java/concurrency/moocwukong/synchronizedlock/Decompilation14, class java/lang/Thread, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4
}
SourceFile: "Decompilation14.java"
```

monitorexit数量多于monitorenter？
因为可能正常退出、也可能抛异常退出

# Monditorenter和Monditorexit指令
计数器为0
可重入计数器累计
计数器非0，其他阻塞

# 原理
获取和释放锁的时机：内置锁