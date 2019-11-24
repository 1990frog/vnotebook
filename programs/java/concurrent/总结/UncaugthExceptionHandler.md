# UncaugthExceptionHandler

UncaugthExceptionHandler接口
void uncaugthException(Thread t,Throwable e);

给程序统一设置
给每个线程单独设置
给线程池设置

Thread.setDefaultUncaughtExceptionHandler(new ...UncaughtExceptionHandler(...))