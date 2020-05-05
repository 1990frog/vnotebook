[TOC]

# values方式
```sql
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    {VALUES | VALUE} (value_list) [, (value_list)] ...
    [ON DUPLICATE KEY UPDATE assignment_list]
```
插入单条
```sql
mysql> insert into table(name,age) values('lilei',16)
```
插入多条
```sql
mysql> insert into table(name,age) values('lilei',16),('hanmeimei',16);
```
# set方式
```sql
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    SET assignment_list
    [ON DUPLICATE KEY UPDATE assignment_list]
```
set只能插入一行，标准插入可以插入多行
```sql
mysql> INSERT INTO uses SET name = '姚明', age = 25;
```
# select方式
```sql
INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    SELECT ...
    [ON DUPLICATE KEY UPDATE assignment_list]
```
# 插入优先级
`[LOW_PRIORITY | DELAYED | HIGH_PRIORITY]`
+ LOW_PRIORITY：使用LOW_PRIORITY关键词，则INSERT的执行被延迟，直到没有其它客户端从表中读取为止
+ DELAYED：是立刻返回一个标识，告诉上层程序，数据已经插入了，当表没有被其它线程使用时，此行被插入，真实插入时间就不可控了。所以这样的写法对数据的安全性是没有保障的
+ HIGH_PRIORITY：高优先级

# [INTO | IGNORE] 没啥用系列
有唯一索引或主键的话，那就屁用没有

# 当数据存在update参数
`[ON DUPLICATE KEY UPDATE]`
ON DUPLICATE KEY UPDATE为Mysql特有语法，这是个坑，语句的作用，当insert已经存在的记录时，执行Update
```sql
INSERT INTO user_admin_t (_id,password)
VALUES
('1','多条插入1') ,
('UpId','多条插入2')
ON DUPLICATE KEY UPDATE
password =  VALUES(password);
```
