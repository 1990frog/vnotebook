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
