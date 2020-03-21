[TOC]

# QPS
+ queries per seconds没秒钟查询数量
+ show global status like 'Question%';
+ Queries/seconds

```
mysql> show global status like'Question%';
+---------------+---------+
| Variable_name | Value   |
+---------------+---------+
| Questions     | 5742404 |
+---------------+---------+
1 row in set (0.01 sec)
```

# TPS
+ Tranaction per seconds
+ TPS=(Com_commit+Com_rollback)/seconds
+ show global status like 'Com_commit%';
+ show global status like 'Com_rollback%';

# 线程连接数
+ show global status like 'Max_used_connections';#线程最大连接数
+ show global status like 'Threads%';#线程数
+ show variables like 'max_connections';#最大线程数

```
mysql> show global status like 'Threads%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 8     |
| Threads_connected | 1     |
| Threads_created   | 2778  |
| Threads_running   | 2     |
+-------------------+-------+
4 rows in set (0.00 sec)

mysql> show global status like 'Max_used_connections';
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| Max_used_connections | 152   |
+----------------------+-------+
1 row in set (0.01 sec)
mysql> show variables like 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+
1 row in set (0.00 sec)
```

# Query Cache
+ 查询缓存用于缓存select查询结果
+ 当下次接收到相同查询请求时，不再执行实际查询处理而直接返回结果
+ 适用于大量查询，很少改变表中数据的场景

## 开启Query Cache
+ 修改my.cnf
+ 将query_cache_size设置为具体的大小，具体大小是多少取决于查询的实际情况，但最好设置为1024的倍数，参考值32M
+ 新增一行：query_cache_type=0/1/2
+ 如果设置为1，将会缓存所有的结果，除非你的select语句使用SQL_NO_CACHE禁用了查询缓存
+ 如果设置为2，则只缓存在select语句中通过SQL_CACHE指定需要缓存的查询
## Query Cache命中率
+ show status like 'Qcache%';

# 锁定状态
+ show global status like '%lock%';
+ Table_locks_waited/Table_locks_immediate值越大代表表锁造成的阻塞越严重
+ Innodb_row_lock_waits innodb行锁，太大可能是间隙所造成的


# 表锁
# 行锁

# 主从延时
+ 查询主从延时时间：show slave status

# mysql慢查询
+ 执行速度操过定义的时间的查询
+ 不同的系统定义不同的慢查询指标
+ 编辑/etc/my.conf，在[mysqld]域中添加
+ 开启慢查询：slow_query_log=1
+ 慢查询日志路径：slow_query_log_file=/data/mysql/slow.log
+ 慢查询的时长：long_query_time=1
+ 未使用索引的查询也被记录到慢查询日志中
+ log_queries_not_using_indexes=1

# 慢查询日志分析
mysqldumpslow命令
-s 是按照何种方式排序
-t 是top n的意思，即为返回前面多少条的数据
-g 后面可以写一个正则匹配模式，大小写不敏感的

得到返回记录集最多的10个sql
`mysqldumpslow -s r -t 10 slow.log`
得到按照时间排序的前10条里面含有左连接的查询语句
`mysqldumpslow -s t -t 10 -g "left join" slow.log`

# sql语句性能分析
