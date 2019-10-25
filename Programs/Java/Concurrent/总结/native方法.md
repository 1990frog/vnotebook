[TOC]

# native方法

native方法（由c或cpp编写），如果想看源码，去网站看openjdk实际实现

# 如何分析native方法

进入github（也可以进入 openJDK网站 ）
点“搜索文件”，搜索对应的c代码类Thread.c
找到native方法对应的方法名
去src/hotspot/share/prims/jvm.cpp里看cpp代码

![20190905205555461_716593482](_v_images/20191022142643768_1313504779.png)

看了以上的说明，对java中断的使用肯定是会了，但我想知道的是阻塞了的线程是如何通过interuppt方法完成停止阻塞并抛出interruptedException的，这就要看Thread中native的interuppt0方法了。

第一步学习Java的JNI调用Native方法。

 

第二步下载openjdk的源代码，找到目录结构里的openjdk-src\jdk\src\share\native\java\lang\Thread.c文件。

```c
#include "jni.h"
#include "jvm.h"

#include "java_lang_Thread.h"

#define THD "Ljava/lang/Thread;"
#define OBJ "Ljava/lang/Object;"
#define STE "Ljava/lang/StackTraceElement;"

#define ARRAY_LENGTH(a) (sizeof(a)/sizeof(a[0]))

static JNINativeMethod methods[] = {
    {"start0",           "()V",        (void *)&JVM_StartThread},
    {"stop0",            "(" OBJ ")V", (void *)&JVM_StopThread},
    {"isAlive",          "()Z",        (void *)&JVM_IsThreadAlive},
    {"suspend0",         "()V",        (void *)&JVM_SuspendThread},
    {"resume0",          "()V",        (void *)&JVM_ResumeThread},
    {"setPriority0",     "(I)V",       (void *)&JVM_SetThreadPriority},
    {"yield",            "()V",        (void *)&JVM_Yield},
    {"sleep",            "(J)V",       (void *)&JVM_Sleep},
    {"currentThread",    "()" THD,     (void *)&JVM_CurrentThread},
    {"countStackFrames", "()I",        (void *)&JVM_CountStackFrames},
    {"interrupt0",       "()V",        (void *)&JVM_Interrupt},
    {"isInterrupted",    "(Z)Z",       (void *)&JVM_IsInterrupted},
    {"holdsLock",        "(" OBJ ")Z", (void *)&JVM_HoldsLock},
    {"getThreads",        "()[" THD,   (void *)&JVM_GetAllThreads},
    {"dumpThreads",      "([" THD ")[[" STE, (void *)&JVM_DumpThreads},
};

#undef THD
#undef OBJ
#undef STE

JNIEXPORT void JNICALL
Java_java_lang_Thread_registerNatives(JNIEnv *env, jclass cls)
{
    (*env)->RegisterNatives(env, cls, methods, ARRAY_LENGTH(methods));
}
```