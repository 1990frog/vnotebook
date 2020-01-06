[TOC]

# 未使用pipeline
![](_v_images/20191128094849466_1313469413.png)
# 使用pipeline
![](_v_images/20191128094920576_928220474.png)

# 性能差别对比
![](_v_images/20191128095112294_1780743983.png)

```java
public class PipelineTest {

    private static Jedis db = RedisUtils.getInstance();

    public static void unpipeline(){
        long startTime=System.currentTimeMillis();
        for(int i=0;i<10000;i++){
            db.set(i+"",System.currentTimeMillis()+"");
        }
        long endTime=System.currentTimeMillis();
        System.out.println(endTime-startTime+"ms");
    }

    public static void pipeline(){
        long startTime=System.currentTimeMillis();
        Pipeline pipeline = db.pipelined();
        for(int i=20000;i<30000;i++){
            pipeline.set(i+"",System.currentTimeMillis()+"");
        }
        pipeline.sync();
        long endTime=System.currentTimeMillis();
        System.out.println(endTime-startTime+"ms");
    }

    public static void main(String[] args) {
        unpipeline();//517ms
        pipeline();//47ms
    }

}
```

# 管道（Pipelining） VS 脚本（Scripting）
大量 pipeline 应用场景可通过 Redis 脚本（Redis 版本 >= 2.6）得到更高效的处理，后者在服务器端执行大量工作。脚本的一大优势是可通过最小的延迟读写数据，让读、计算、写等操作变得非常快（pipeline 在这种情况下不能使用，因为客户端在写命令前需要读命令返回的结果）。
应用程序有时可能在 pipeline 中发送 EVAL 或 EVALSHA 命令。Redis 通过 SCRIPT LOAD 命令（保证 EVALSHA 成功被调用）明确支持这种情况。

# 原生批命令(mset, mget)与Pipeline对比
1、原生批命令是原子性，pipeline是非原子性
(原子性概念:一个事务是一个不可分割的最小工作单位,要么都成功要么都失败。原子操作是指你的一个业务逻辑必须是不可拆分的. 处理一件事情要么都成功，要么都失败，原子不可拆分)
2、原生批命令一命令多个key, 但pipeline支持多命令（存在事务），非原子性
3、原生批命令是服务端实现，而pipeline需要服务端与客户端共同完成

# Pipeline正确使用方式
使用pipeline组装的命令个数不能太多，不然数据量过大，增加客户端的等待时间，还可能造成网络阻塞，可以将大量命令的拆分多个小的pipeline命令完成。
1. 注意每次pipeline携带数据量
2. pipeline每次只能作用在一个Redis节点上
3. 注意M操作与pipeline的差别

# 拓展
北京距离上海1300公里
光速=3*10^8米/秒=30000公里
光纤传输速度约等于光速的2/3
```python
>>> (1300*2)/(30000*2/3)
0.13
```
一次命令的传输时间=(1300*2)/(30000*2/3)=13
