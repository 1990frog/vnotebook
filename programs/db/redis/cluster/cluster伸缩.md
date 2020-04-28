[TOC]

# 伸缩原理
槽和数据在节点之间的移动

# 扩容集群
+ 准备新节点
    1. 集群模式
    2. 配置和其他节点统一
    3. 启动后是孤儿节点
+ 加入集群
+ 迁移槽和数据

## 加入集群
加入
```
$ cluster meet 127.0.0.1 port
$ cluster meet 127.0.0.1 port
```
观察集群配置
```
$ cluster nodes
```
## 加入集群-作用
+ 为它迁移槽和数据实现扩容
+ 作为从节点负责故障转移

## 迁移槽和数据
+ 槽迁移计划
+ 迁移数据
+ 添加从节点

# 扩容实战
创建两个新的conf
```
$ seq 's/7000/7006/g' 7000.conf > 7006.conf
$ seq 's/7000/7007/g' 7000.conf > 7007.conf
```
meet
```
$ reids-cli -p 7000 cluster meet 127.0.0.1 7006
$ reids-cli -p 7000 cluster meet 127.0.0.1 7007
```
设置主从
```
$ redis-cli -p 7007 replicate 7006redisid
```
实现槽的分配

# 集群收缩
+ 下线迁移槽
+ 忘记节点
+ 关闭节点

## 忘记节点
```
redis-cli> cluster forget {downNodeId}
```