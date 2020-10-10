[TOC]

# 原理：追求性能
1. 从集群中选一个可运行节点，使用cluster slots初始化槽和节点映射
2. 将cluster slots的结果映射到本地，为每个节点创建JedisPool
3. 准备执行命令

![](https://gitee.com/caijingquan/imagebed/raw/master/1602320348_20200404141432097_925452012.png)
