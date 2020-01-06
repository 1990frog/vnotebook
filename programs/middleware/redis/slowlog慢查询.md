[TOC]

Redis提供了5种数据结构,但除此之外,Redis还提供了注入慢查询分析,Redis Shell、Pipeline、事务、与Lua脚本、Bitmaps、HyperLogLog、PubSub、GEO等附加功能,这些功能可以在某些场景发挥很重要的作用.

# redis执行流程
![944418018-594e1d9369319_articlex](_v_images/20191127102247812_1786991374.png)

+ 发送命令
+ 命令排队
+ 命令执行
+ 返回结果
需要注意,慢查询只统计步骤3的时间,所以没有慢查询并不代表客户端没有超时问题.

# 慢查询两个配置
对于慢查询功能,需要明确两件事:

+ 预设阈值怎么设置?
+ 慢查询记录存放在那?

# slowlog-max-len
设置存放多少条慢查询指令

1. 先进先出队列
2. 固定长度
3. 保存在内存中

从字面意思看,slowlog-max-len只是说明了慢查询日志最多存储多少条,并没有说明存放在哪里?实际上Redis使用了一个列表来存储慢查询日志,slowlog-max-len就是列表的最大长度.一个新的命令满足慢查询条件时被插入到这个列表中,当慢查询日志列表已处于其最大长度时,最早插入的一个命令将从列表中移出,例如slowlog-max-len设置长度为64.当有第65条慢查询日志插入的话,那么队头的第一条数据就出列,第65条慢查询就会入列.
```
默认值
config get slow-max-len=128
config get slowlog-log-slower-than=10000
```

# slowlog-log-slower-than
设置慢查询阈值

1. 慢查询阈值（单位：微秒）
2. slowlog-log-slower-than=0，记录所有命令
3. slowlog-log-slower-than<0，不记录任何命令

Redis提供了slowlog-log-slower-than和slowlog-max-len配置来解决这两个问题.从字面意思就可以看出,slowlog-log-slower-than就是这个预设阈值,它的单位是毫秒(1秒=1000000微秒)默认值是10000,假如执行了一条"很慢"的命令(例如key *),如果执行时间超过10000微秒,那么它将被记录在慢查询日志中.

# 修改配置
在Redis中有两种修改配置的方法,一种是修改配置文件,另一种是使用config set命令动态修改.
例如下面使用config set命令将slowlog-log-slower-than设置为20000微妙.slowlog-max-len设置为1024:
```
redis> config set slowlog-log-slower-than 20000
OK
redis> config set slowlog-max-len 1024
redis> config rewrite #Redis Config rewrite 命令对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写。
```

# 慢查询命令
1.slowlog get[n] ：获取慢查询队列
```
127.0.0.1:6379> slowlog get 1
1) 1) (integer) 14 #id
   2) (integer) 1574823494 #发生时间戳
   3) (integer) 3 #命令耗时
   4) 1) "SET" #执行命令和参数
      2) "hello"
      3) "world"
   5) "127.0.0.1:53902"
   6) ""
```
2.slowlog len：获取慢查询队列长度
3.slowlog reset：清空慢查询

# 运维经验
1.slow-max-len不要设置过大，默认10ms，通常设置1ms
2.slow-log-slower-than不要设置过小，通常设置1000左右
3.理解命令生命周期
4.定期持久化慢查询


慢查询功能可以有效地帮助我们找到Redis可能存在的瓶颈,但在实际使用过程中要注意以下几点:
+ slowlog-max-len:线上建议调大慢查询列表,记录慢查询时Redis会对长命令做阶段操作,并不会占用大量内存.增大慢查询列表可以减缓慢查询被剔除的可能,例如线上可设置为1000以上.
+ slowlog-log-slower-than:默认值超过10毫秒判定为慢查询,需要根据Redis并发量调整该值.由于Redis采用单线程相应命令,对于高流量的场景,如果命令执行时间超过1毫秒以上,那么Redis最多可支撑OPS不到1000因此对于高OPS场景下的Redis建议设置为1毫秒.
+ 慢查询只记录命令的执行时间,并不包括命令排队和网络传输时间.因此客户端执行命令的时间会大于命令的实际执行时间.因为命令执行排队机制,慢查询会导致其他命令级联阻塞,因此客户端出现请求超时时,需要检查该时间点是否有对应的慢查询,从而分析是否为慢查询导致的命令级联阻塞.
+ 由于慢查询日志是一个先进先出的队列,也就是说如果慢查询比较多的情况下,可能会丢失部分慢查询命令,为了防止这种情况发生,可以定期执行slowlog get命令将慢查询日志持久化到其他存储中(例如:MySQL、ElasticSearch等),然后可以通过可视化工具进行查询.

