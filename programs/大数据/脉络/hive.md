[TOC]

# Hive vs Hadoop
hive数据存储：Hive的数据是存储在HDFS上的，Hive的库和表是对HDFS上数据的映射
hive元数据存储：元数据存储一般在外部关系库（Mysql），与Presto Impala等共享
hive语句的执行过程：将HQL转换为MapReduce任务运行

# Hive vs 数据库
|         |   Hive    |  RDBMS   |
| ------- | --------- | -------- |
| 查询语言 | HQL       | SQL      |
| 数据存储 | HDFS      | 本地磁盘 |
| 索引     | 无        | 有       |
| 执行     | MapReduce | Executor |
| 执行延时 | 高        | 低       |
| 数据规模 | 大        |     小     |

# 数据类型
基本数据类型：int、float、double、string、boolean、bigint等
复杂类型：array、map、struct

# hive分区
hive将海量数据按某几个字段进行分区，查询时不必加载全部数据
分区对应到hdfs就是hdfs的目录
分区分为静态分区和动态分区两种

# hive语法
指定分隔符分区字段
![](_v_images/20200806185224420_7199.png)

# 连接方式
hive client
hive -e "sql"
hive -f "sql file"
beeline

复杂结构：kv之间用`:`进行分割，集合用`,`分割


```sql
hive (default)> create table test(
              > name string,
              > id string
              > )
              > row format delimited
              > fields terminated by '|'
              > lines terminated by '\n';
OK
Time taken: 0.567 seconds
```

```
chengdu|1
beijing|2
guangzhou|3
```

```
hive (default)> load data local inpath '/home/cai/Desktop/test.csv' overwrite into table test;
Loading data to table default.test
OK
Time taken: 1.01 seconds
```

```
hive (default)> select * from test;
OK
test.name       test.id
chengdu 1
beijing 2
guangzhou       3
Time taken: 0.756 seconds, Fetched: 3 row(s)
```

ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':'
LINES TERMINATED BY '\n'



# hive数据模型
内部表：和传统数据库的table概念类似，对应hdfs上存储目录，删除表时，删除元数据和表数据
外部表：指向已经存在的hdfs数据，删除时只删除元数据信息


创建外部表关键字


# 数据模型
分区表：Partition对应普通数据库对Partition列的密集索引，将数据按照Partition列存储到不同目录，便于并行分析，减少数据量
分桶表：对数据进行hash，放到不同文件存储，方便抽样和join查询


抽样查询
```
hive> select * from bucket_table TABLESAMPLE(bucket 1 out of 2 on user_name);
```
