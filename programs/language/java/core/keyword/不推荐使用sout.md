[TOC]

# 优点
方便

# 缺点
存在锁，影响性能

# 源码
System类
```java
public void println(double x) {
        synchronized (this) {
            print(x);
            newLine();
        }
    }


public final class System {
    /** Don't let anyone instantiate this class */
    private System() {
    }
    public final static InputStream in = null;
    public final static PrintStream out = null;

    //都是在底层赋值
    private static native void registerNatives();
    static {
        registerNatives();
    }

}
```
PrintStream类
```java
public void println(int x) {
    synchronized (this) {
        print(x);
        newLine();
    }
}
public void print(long l) {
    write(String.valueOf(l));
}
private void write(String s) {
    try {
        synchronized (this) {
            ensureOpen();
            textOut.write(s);
            textOut.flushBuffer();
            charOut.flushBuffer();
            if (autoFlush && (s.indexOf('\n') >= 0))
                out.flush();
        }
    }
    catch (InterruptedIOException x) {
        Thread.currentThread().interrupt();
    }
    catch (IOException x) {
        trouble = true;
    }
}
```