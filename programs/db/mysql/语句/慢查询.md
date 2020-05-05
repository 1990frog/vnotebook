[TOC]

# sql优化的一般步骤
+ 发现慢查询
+ 分析执行计划
+ 优化索引
+ 改写sql
+ 数据库垂直切分
+ 数据库水平切分

# 常见问题发现渠道
+ 用户主动上报应用性能问题
+ 分析慢查询日志发现存在问题的sql
+ 数据库事实监控长时间运行的sql

# 配置慢查询日志
```
set global slow_query_log = [ON|OFF]
set global slow_query_log_file = /sql_log/slowlog.log
set global long_query_time = xx.xxx秒
set global log_queries_not_using_indexes = [ON|OFF]
```

## mysql慢查询工具
官方工具
```
mysqldumpslow [OPTS...] [LOGS...]
```
另外的
```
pt-query-digest [OPTIONS] [FILES] [DSN]
```
安装percona


```
# 默认10s
mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)

# 为了学习分析慢查询，设置为监控所有的sql，工作中一般设置成0.001:100ms
# 需要关闭当前回话重启
mysql> set global long_query_time = 0;
Query OK, 0 rows affected (0.00 sec)


mysql> show variables like 'slow_query_log';
+----------------+-------+
| Variable_name  | Value |
+----------------+-------+
| slow_query_log | OFF   |
+----------------+-------+
1 row in set (0.00 sec)

mysql> show variables like 'slow_query_log_file';
+---------------------+------------------------------+
| Variable_name       | Value                        |
+---------------------+------------------------------+
| slow_query_log_file | /var/lib/mysql/frog-slow.log |
+---------------------+------------------------------+
1 row in set (0.00 sec)

```

```
sudo mysqldumpslow /var/lib/mysql/frog-slow.log

Count: 1  Time=0.01s (0s)  Lock=0.00s (0s)  Rows=1.0 (1), root[root]@localhost
  select * from customer
```

```
$ sudo pt-query-digest /var/lib/mysql/frog-slow.log

# 120ms user time, 20ms system time, 29.94M rss, 37.99M vsz
# Current date: Wed Mar 18 20:12:25 2020
# Hostname: frog
# Files: /var/lib/mysql/frog-slow.log
# Overall: 13 total, 10 unique, 0.00 QPS, 0.00x concurrency ______________
# Time range: 2020-03-18T09:36:39 to 2020-03-18T12:09:05
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time           22ms    57us     7ms     2ms     4ms     2ms     1ms
# Lock time            4ms       0     1ms   286us     1ms   420us   144us
# Rows sent             24       0       9    1.85    8.91    3.06    0.99
# Rows examine       1.21k       0     576   95.31  563.87  201.01    0.99
# Query size           269       7      37   20.69   34.95    9.68   19.13

# Profile
# Rank Query ID                          Response time Calls R/Call V/M   
# ==== ================================= ============= ===== ====== ===== 
#    1 0x55325F1F21F22D574953A7454105...  0.0074 33.0%     1 0.0074  0.00 SELECT customer
#    2 0x489B4CEB2F4301A7132628303F99...  0.0055 24.6%     2 0.0028  0.00 SHOW TABLES
#    3 0x751417D45B8E80EE5CBA2034458B...  0.0047 20.8%     2 0.0023  0.00 SHOW DATABASES
#    4 0xE77769C62EF669AA7DD5F6760F2D...  0.0042 18.5%     2 0.0021  0.00 SHOW VARIABLES
# MISC 0xMISC                             0.0007  3.1%     6 0.0001   0.0 <6 ITEMS>

select * from customer\G
# Query 2: 0.40 QPS, 0.00x concurrency, ID 0x489B4CEB2F4301A7132628303F99240D at byte 2078
# This item is included in the report because it matches --limit.
# Scores: V/M = 0.00
# Time range: 2020-03-18T12:07:38 to 2020-03-18T12:07:43
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count         15       2
# Exec time     24     6ms     2ms     4ms     3ms     4ms     2ms     3ms
# Lock time     35     1ms   240us     1ms   667us     1ms   604us   667us
# Rows sent      8       2       1       1       1       1       0       1
# Rows examine   0      10       5       5       5       5       0       5
# Query size     8      22      11      11      11      11       0      11
# String:
# Databases    customer
# Hosts        localhost
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms  ################################################################
#  10ms
# 100ms
#    1s
#  10s+
show tables\G
```

# 通过实时监控发现问题
管理试图
```
select id,'user','host',DB,command,'time',state,info
from information_schema.PROCESSLIST
where time>60
```