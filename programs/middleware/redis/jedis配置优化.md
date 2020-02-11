[TOC]

# 参数
Jedis(String host,int port,int connectionTimeout,int soTimeout)
host：Redis节点的所在机器的ip
port：Redis节点的端口
connectTimeout：客户端连接超时
soTimeout：客户端读写超时

初始化Jedis连接池，通常来讲JedisPool是单例的
```
GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
JedisPool jedisPool = new JedisPool(poolConfig,"127.0.0.1",6379)；
```

commo-pool
配置（资源数控制）

maxTotal
资源池最大连接数
默认8
适合的maxTotal
比较难确定的，举个例子：
1.命令平均执行时间0.1ms=0.001s
2.业务需要50000QPS
3.maxTotal理论值=0.001*50000=50个（理想值）。实际上值要偏大一些。
需要考虑的：
1.业务希望redis并发量
2.客户端执行命令时间
3.redis资源：例如nodes（例如应用个数）*maxTotal是不能超过redis的最大连接数。（config get maxclients）
4.资源开销：例如希望控制空闲连接，但是不希望因为连接池的频繁释放创建连接造成不必要开销。

maxIdle
资源池运行最大空闲连接数
默认8
适合的maxIdle
建议：maxIdle=maxTotal
减少创建新连接的开销

minIdle
资源池确保最少空闲连接数
默认0
适合的minIdle
建议预热minIdle
减少第一次启动后的新连接开销

jmxEnabled
是否开启jmx监控，可用于监控
默认true

blockWhenExhausted当资源池连接时是否做连接有效性检测（ping），无效连接会被移除
当资源池用尽后，调用者是否要等待。只有当为true时，下面的maxWaitMillis才会生效
默认true
使用建议：建议使用默认值

maxWaitMilis
当资源池连接用尽后，调用者的最大等待时间（单位为毫秒）
默认值：-1表示永不超时
使用建议：不建议使用默认值

testOnBorrow
向资源池借用连接时是否做连接有效性检测（ping），无效连接会被移除
默认值：false
使用建议：false

testOnReturn
向资源池归还连接时是否做连接有效性检测（ping），无效连接会被移除
默认值：false
建议：false



解决思路
1.慢查询阻塞：池子连接都被hang住。（设置超时时间）
2.资源池参数不合理：例如QPS高、池子小。
3.连接泄露（没有close()）：此类问题比较难定位，例如client list、netstat等，最重要的是代码。
4.DNS异常等。


池？？？？？？？