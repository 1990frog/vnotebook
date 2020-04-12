[TOC]

集群完整性
带宽消耗
Pub/Sub广播
数据倾斜
读写分离
数据迁移

# 集群完整性
`cluster-require-full-coverage`默认yes
集群中16384个槽全部可用：保证集群完整性

大多数业务无法容忍，建议设置为no

1000个节点1个出现问题，那么剩余的999个都不许用

# 带宽消耗
官方建议节点不要超过1000个
ping/pong消息
不容忽视的带宽消耗

消息发送频率：节点发现与其它节点最后通信时间超过cluster-node-timeout/2时会直接发送ping消息
消息数据量：slots槽数组（2kb空间）和整个集群1/10的状态数据（10个节点状态数据约1kb）
节点部署的机器规模：集群分布的集群越多且每台集群划分的节点数越均匀，则集群内整体的可用带宽越高

例子：
规模：节点200个、20台物理机（每台10个节点）
cluster-node-timout=15000,ping/pong带宽为25MB
cluster-node-timout=20000,ping/pong带宽低于15MB

# Pub/Sub广播
集群内全部主节点子节点都会收到消息？

# 请求倾斜
超热点数据，可用用多级缓存

# 离线/在线迁移
将单节点数据迁移到集群

在线迁移工具：伪装成slave拿到全量数据
唯品会：redis-migrate-tool
豌豆荚：redis-port

# 集群读写分离
只读连接：集群模式的从节点不接受任何读写请求

Cluster 中的 slave 不接受任何读写请求；
对 slave 的读请求会重定向到负责 slot 的 master；
可以使用连接级别的命令 readonly 从 slave 中读数据；


# 集群VS单机
+ key批量操作支持有限：例如mget、mset必须在一个slot
+ key事务和Lua支持有限：操作的key必须在一个节点
+ key是数据分区的最小粒度：不支持bigkey分区
+ 不支持多个数据库：集群模式下只有一个db 0
+ 复制支持一层：不支持树形复制结果

redis cluster：满足容量和性能的扩展性，很多业务不需要
+ 大多数时客户端性能会降低
+ 命令无法跨节点使用：mget、keys、scan、flush、sinter等
+ lua和事务无法跨节点使用
+ 客户端维护更复杂：sdk和应用本身消耗（例如更多的连接池）

很多业务场景Redis Sentinel已经足够好

```
*** ERROR: Invalid configuration for cluster creation.
*** Redis Cluster requires at least 3 master nodes.
```
redis cluster最少需要3个节点