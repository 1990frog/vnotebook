[TOC]

# RDB有什么问题
耗时、耗性能
不可控、丢失数据

![](_v_images/20191207140354991_661003156.png)

![](_v_images/20191207140523206_1841257735.png)

# AOF
![](_v_images/20191207141100338_1880480749.png)

![](_v_images/20191207141147209_1573817606.png)

# AOF的三种策略
always
everysec
no

# always
![](_v_images/20191207141640160_641517644.png)
# everysec
![](_v_images/20191207141717648_2077475983.png)
# no
![](_v_images/20191207141832688_1326583416.png)
# 三种策略选择
always
优点：不丢失数据
缺点： IO开销较大，一般的ata盘只有几百TPS
everysec
优点：每秒一次fsync丢1秒数据
缺点：丢1秒数据
no
优点：不用管
缺点：不可控
# AOF 重写
![](_v_images/20191207142741613_203728326.png)
1.减少硬盘占用量
2.加速恢复速度

# AOF重写实现两种方式
1.bgrewriteaof
2.AOF重写配置

![](_v_images/20191207143506097_2116954724.png)
AOF重写是将redis内存中的数据进行回朔并写入aof文件

# AOF重写配置
配置
auto-aof-rewrite-min-size AOF文件重写需要的尺寸
auto-aof-rewrite-percentage AOF文件增长率
统计
aof_current_size AOF当前尺寸（单位：字节）
aof_base_size AOF上次启动和重写的尺寸（单位：字节）
自动触发时机
aof_current_size>auto-aof-rewrite-min-size
aof_current_size-aof_base_size/aof_base_size>auto-aof-rewrite-percentage

![](_v_images/20191207144237476_1349180936.png)

# AOF配置
```
appendonly yes    #使用aof所有功能
appendfilename "appendonly-${port}.aof"    #设置aof名称
appendfsync everysec    #aof同步策略
dir /bigdiskpath    #保存rdb、log、aof的目录
no-appendfsync-on-rewrite yes    #在aof重写的时候是否要做一个aof的判断操作，yes不做操作，aof重写非常消耗性能
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

aof-load-truncated yes
```

# 实战
```
redis> config set appendonly yes
redis> bgrewriteaof    #手动触发aof
```
config配置，与直接更改config差异？

# RDB与AOF对比
![](_v_images/20191207145515417_510291277.png)

rdb是一个快照的概念，可能会丢失数据，只能恢复到特定的时间节点

# RDB最佳策略
关闭rdb
集中管理
主从，从开？

# AOF最佳策略
"开"：缓存和存储
AOF重写集中管理（分配内存的70,80给redis，剩余的保障开启fork不会出现异常）
选择everysec

# 最佳策略
小分片
缓存或者存储
监控（硬盘、内存、负载、网络）
足够的内存

# 开发运维常见问题
1.fork操作
2.进程外开销
3.AOF追加阻塞
4.单机多实例部署

# fork操作
1.同步操作
2.与内存量息息相关：内存越大，耗时越长（与机器类型有关）
3.info:latest_fork_usec 查看持久化时间

# 改善fork
1.优化使用物理机或者高效支持fork操作的虚拟化技术
2.控制Redis实例最大可用内存：maxmemory
3.合理配置Linux内存分配策略：vm.overcommit_memory=1
4.降低fork频率：例如放宽AOF重写自动触发时机，不必要的全量复制

# 子进程开销和优化
1.cpu
开销：rdb和aof文件生成，属于cpu密集型
优化：不做cpu绑定，不和cpu密集型部署
2.内存
开销：fork内存开销，copy-on-write。
优化：echo never > /sys/kernel/mm/transparent_hugepage/enabled
3.硬盘
开销：aof和rdb文件写入，可以结合iostat，iotop分析

# 硬盘优化
1.不要和高硬盘负载服务部署一起：存储服务、消息队列等
2.no-appendfsync-on-rewrite=yes
3.根据写入量决定磁盘类型：例如ssd
4.单机多实例持久化文件目录可以考虑分盘

# aof追加阻塞
![](_v_images/20191208135325755_1778392891.png)

# aof阻塞定位
Redis日志：
Asynchronous AOF fsync is taking too long(disk is busy?).
Writing the AOF buffer without waiting for fsync to complete,this may slow down Redis.

`info persistence`查看持久化信息,aof次数

# 主从
是不是只有一个主，多个从这种情况

slaveof 127.0.0.1 6379

info replication
![](_v_images/20191210215718384_1173997905.png)

![](_v_images/20191210220027814_1343473929.png)

redis:run_id

![](_v_images/20191210220537211_428995505.png)

flushall

# runid和复制偏移量
重启之后runid会更换
runid是一个标识

主节点偏移量
![](_v_images/20191210221143837_2086251922.png)
从节点偏移量
![](_v_images/20191210221225248_848160514.png)

如果主从节点偏移量不同可能为：
网络，阻塞，缓存区出现问题


# 全量复制
![](_v_images/20191210221751563_402309212.png)

# 全量复制开销
![](_v_images/20191210222028033_2074634944.png)

redis2.8之前都是全量复制

# 部分复制
如果发生了抖动
![](_v_images/20191210222326267_1712647051.png)

# 故障处理

# 自动故障转移

# 主从结构——故障转移

## slave宕掉
![](_v_images/20191210222812172_531453535.png)
## master宕掉
![](_v_images/20191210223014498_1553712826.png)
