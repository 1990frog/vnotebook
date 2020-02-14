[TOC]

# 主节点
```redis
port 6379    # 端口
daemonize yes    # 守护线程模式
pidfile /var/run/redis-6379.pid    # 创建一个pid文件
dir /opt/redis/data/6379/    # 工作目录
logfile 6379.log    # 日志
# databases 16 设置数据库数量，默认0
save 60 10000    # 设置RDB
appendonly yes    # 设置AOF
appendfilename "6379.aof"    # 设置AOF文件名称
appendfsync everysec    # 设置AOF策略
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
no-appendfsync-on-rewrite yes    # 在aof重写的时候是否要做一个aof的判断操作，yes不做操作，aof重写非常消耗性能
auto-aof-rewrite-percentage 100    # aof文件增长比例，指当前aof文件比上次重写的增长比例大小
auto-aof-rewrite-min-size 64mb    # aof文件重写最小的文件大小，即最开始aof文件必须要达到这个文件时才触发，后面的每次重写就不会根据这个变量了(根据上一次重写完成之后的大小).此变量仅初始化启动redis有效.如果是redis恢复时，则lastSize等于初始aof文件大小.
```

# 从节点
```redis
port 6380    # 端口
daemonize yes    # 守护线程模式
pidfile /var/run/redis-6380.pid    # 创建一个pid文件
dir /opt/redis/data/6380/    # 工作目录
logfile 6380.log    # 日志
# databases 16 设置数据库数量，默认0

slaveof localhost 6379    # 设置为6370的从节点

save 60 10000    # 设置RDB

appendonly yes    # 设置AOF
appendfilename "6380.aof"    # 设置AOF文件名称
appendfsync everysec    # 设置AOF策略

no-appendfsync-on-rewrite yes    # 在aof重写的时候是否要做一个aof的判断操作，yes不做操作，aof重写非常消耗性能
auto-aof-rewrite-percentage 100    # aof文件增长比例，指当前aof文件比上次重写的增长比例大小
auto-aof-rewrite-min-size 64mb    # aof文件重写最小的文件大小，即最开始aof文件必须要达到这个文件时才触发，后面的每次重写就不会根据这个变量了(根据上一次重写完成之后的大小).此变量仅初始化启动redis有效.如果是redis恢复时，则lastSize等于初始aof文件大小.
```

# 哨兵节点
```

```