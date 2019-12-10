[TOC]

# 数据库常见备份策略
快照：
1. MySQL Dump
2. Redis RDB
写日志：
1. MySQL Binlog
2. Hbase HLog
3. Redis AOF

# RDB
# 什么是RDB？
将redis内存中的数据存储到RDB文件（二进制）到硬盘
redis再次启动加载RDB文件（二进制）
# 触发机制——主要三种方式
1. save（同步）
2. bgsave（异步）
3. 自动

# save命令
```
redis>save
OK
生成RDB文件（二进制）
```
save是同步命令，如果执行时间过长，会造成redis阻塞

文件策略：
如果使用save生成RDB文件，如果存在老的RDB文件，新文件会替换老文件

复杂度：O(N)

# bgsave命令
![](_v_images/20191205204400834_1662543076.png)

fork在大多数情况下是非常快的，不会阻塞到redis
bgsave文件策略与复杂度与save是相同的

![](_v_images/20191205204628478_1269437250.png)

# redis自动生成RDB
![](_v_images/20191205205004123_591044714.png)
如上图，60s内，10000条数据发生变化，会生成RDB文件
```
save 900 1
save 300 10
save 60 10000
dbfilename dump.rdb
dir ./
配置
stop-writes-on-bgsave-error yes 如果bgsave出现了问题，那我们停止写入
rdbcompression yes rdb文件是否采用压缩模式
rdbchecksum yes 是否对rdb文件进行检验
```

![](_v_images/20191205205902454_1170609259.png)

# 触发机制——不容忽略方式
1.全量复制
2.debug reload
3.shutdown

# 试验
习惯在redis目录下data/config/redis-{port}.conf 存放我们的config

config配置
```
daemonize yes 是否以守护进程来启动
logfile "{port}.log}" 日志文件
#save 900 1
#save 300 10
#save 60 10000
dbfilename dump-{port}.rdb 定义rdb名称
dir /opt/soft/redis/data 定义rdb目录
```
启动：redis-server redis-6379.conf
```
dbsize 数据量
info memory 内存用量
```

## 测试save阻塞
![](_v_images/20191205211440944_1681875098.png)
![](_v_images/20191205211455114_693872982.png)
![](_v_images/20191205211553218_1466491314.png)
save正在运行，get命令阻塞

## bgsave不易阻塞
查看redis进程
![](_v_images/20191205211737151_571823556.png)
启动bgsave
![](_v_images/20191205211840077_654163319.png)
![](_v_images/20191205211921113_1210036157.png)
![](_v_images/20191205211941657_861164746.png)
会启一个bgsave进程
![](_v_images/20191205212033453_1901485361.png)
先生成个临时文件，如果生成成功，则覆盖原rdb

更改配置文件要重启redis

# rdb总结
1.RDB是Redis内存到硬盘的快照，用于持久化。
2.save通常会阻塞Redis。
3.bgsave不会阻塞Redis，但是会fork新进程。
4.save自动配置满足任一就会执行。（但是通常不会使用）
5.有些触发机制不容忽视