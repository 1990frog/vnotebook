[TOC]

# 快记
config中save参数
save命令：同步rdb
bgsave命令：异步rdb

# 数据库常见备份策略
快照：
1. MySQL Dump
2. Redis RDB

写日志：
1. MySQL Binlog
2. Hbase HLog
3. Redis AOF
# 什么是RDB
将redis内存中的数据存储到RDB文件（二进制）到硬盘，redis再次启动加载RDB文件（二进制），可以将其理解为快照
![2701342763-5c851b9a308a0_articlex](_v_images/20191212093520384_104500081.png)
# 触发机制——主要三种方式
1. save命令触发（同步）
2. bgsave命令触发（异步）
3. 自动触发（全量复制、debug reload、shutdown）
# save命令
```
redis>save
OK
生成RDB文件（二进制）
```
注意：
save是同步命令，如果执行时间过长，**会造成redis阻塞**
文件策略：
如果使用save生成RDB文件，如果存在老的RDB文件，新文件会替换老文件
# bgsave命令
![1938339134-5c851b9a33c9e_articlex](_v_images/20191212093409101_846180634.png)
fork在大多数情况下是非常快的，不会阻塞到redis
bgsave文件策略复杂度与save是相同的
# redis自动生成RDB
在conf文件中配置

```
save 900 1
save 300 10
save 60 10000
dbfilename dump.rdb
dir ./

stop-writes-on-bgsave-error yes    #如果bgsave出现了问题，那我们停止写入
rdbcompression yes    #rdb文件是否采用压缩模式
rdbchecksum yes    #是否对rdb文件进行检验
```
# 触发机制（不容忽略方式）
1.全量复制
2.debug reload
3.shutdown

# 试验
config配置
```
################################ SNAPSHOTTING 快照配置 ################################
#
# Save the DB on disk:
# 将数据保存在磁盘上：
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   如果给定的秒数和给定的秒数，就会保存DB（针对参数seconds）
#   number of write operations against the DB occurred.
#   发生了针对DB的写操作的数量。（针对参数changes）
#
#   In the example below the behaviour will be to save:
#   在下面的例子中，行为将是保存：
#   after 900 sec (15 min) if at least 1 key changed
#   900秒内至少有1次写入
#   after 300 sec (5 min) if at least 10 keys changed
#   300秒内至少有10次写入
#   after 60 sec if at least 10000 keys changed
#   60秒内至少有10000次写入
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#   注意：你可以通过注释掉所有的“保存”行来完全禁用保存。
#
#   It is also possible to remove all the previously configured save
#   也可以删除先前配置的所有save
#   points by adding a save directive with a single empty string argument
#   通过添加一个带有空字符串参数的save指令
#   like in the following example:
#   如下面的例子：
#
#   save ""

save 900 1
save 300 10
save 60 10000

启动：redis-server redis-6379.conf
```

## 测试redis阻塞
```java
public static void pipeline(){
    long startTime=System.currentTimeMillis();
    Pipeline pipeline = db.pipelined();
    for(int i=10000;i<30000000;i++){
        pipeline.set(i+"",System.currentTimeMillis()+"");
    }
    pipeline.sync();
    long endTime=System.currentTimeMillis();
    System.out.println(endTime-startTime+"ms");
}

执行时间66732ms
```
java一次插入30000000条数据，之后调用keys命令卡死
```
127.0.0.1:6379> keys *    #卡死没有反应

```
keys30000000条数据，耗时如下
```
19776787) "18844982"
19776788) "18189989"
19776789) "17893531"
19776790) "18051402"
19776791) "18229016"
19776792) "18385275"
19776793) "18896039"
19776794) "17433990"
19776795) "17475906"
19776796) "17414527"
19776797) "18546229"
19776798) "19749837"
19776799) "17620024"
19776800) "16814575"
19776801) "19606614"
19776802) "19460753"
19776803) "19551828"
19776804) "18758850"
(139.79s)
```
插入数据之后，内存使用情况
```
127.0.0.1:6379> dbsize
(integer) 30000599
127.0.0.1:6379> info memory
# Memory
used_memory:2181344400
used_memory_human:2.03G
used_memory_rss:2280652800
used_memory_rss_human:2.12G
used_memory_peak:2313023640
used_memory_peak_human:2.15G
used_memory_peak_perc:94.31%
used_memory_overhead:1469305374
used_memory_startup:796232
used_memory_dataset:712039026
used_memory_dataset_perc:32.65%
allocator_allocated:2181841192
allocator_active:2182402048
allocator_resident:2281639936
total_system_memory:16695361536
total_system_memory_human:15.55G
used_memory_lua:37888
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
allocator_frag_ratio:1.00
allocator_frag_bytes:560856
allocator_rss_ratio:1.05
allocator_rss_bytes:99237888
rss_overhead_ratio:1.00
rss_overhead_bytes:-987136
mem_fragmentation_ratio:1.05
mem_fragmentation_bytes:99349432
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:49694
mem_aof_buffer:0
mem_allocator:jemalloc-5.2.1
active_defrag_running:0
lazyfree_pending_objects:0
```

## 测试save阻塞
save窗口
```
127.0.0.1:6379> save
OK
(22.64s)    #阻塞了22.64秒
```
get窗口
```
127.0.0.1:6379> get hello
"world"
(17.56s)
```
save正在运行，get命令阻塞

## bgsave不易阻塞
```
127.0.0.1:6379> save
OK
(22.64s)
127.0.0.1:6379> bgsave
Background saving started

127.0.0.1:6379> get hello
"world"
(17.56s)
127.0.0.1:6379> get hello
"world"
```
bgsave进程
```
redis> ps -ef | grep redis
root      7715     1  3 10:10 ?        00:00:54 redis-server 127.0.0.1:6379
cai       7720  7629  0 10:10 pts/1    00:00:00 redis-cli
cai       9182  9119  0 10:21 pts/2    00:00:00 redis-cli
root     10883  7715 90 10:32 ?        00:00:01 redis-rdb-bgsave 127.0.0.1:6379
cai      10885 10770  0 10:32 pts/3    00:00:00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn redis
```
首先生成个临时文件，如果生成成功，则覆盖原rdb

**更改配置文件要重启redis**

# rdb总结
1. RDB是Redis内存到硬盘的快照，用于持久化。
2. save通常会阻塞Redis。
3. bgsave不会阻塞Redis，但是会fork新进程。
4. save自动配置满足任一就会执行。（但是通常不会使用）
5. 有些触发机制不容忽视