[TOC]

Date Manipulation language

# insert
## 语法
```sql
Syntax:
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    {VALUES | VALUE} (value_list) [, (value_list)] ...
    [ON DUPLICATE KEY UPDATE assignment_list]

INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    SET assignment_list
    [ON DUPLICATE KEY UPDATE assignment_list]

INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    SELECT ...
    [ON DUPLICATE KEY UPDATE assignment_list]
```
### [LOW_PRIORITY | DELAYED | HIGH_PRIORITY]
+ LOW_PRIORITY：使用LOW_PRIORITY关键词，则INSERT的执行被延迟，直到没有其它客户端从表中读取为止
+ DELAYED：是立刻返回一个标识，告诉上层程序，数据已经插入了，当表没有被其它线程使用时，此行被插入，真实插入时间就不可控了。所以这样的写法对数据的安全性是没有保障的
+ HIGH_PRIORITY：高优先级
### [INTO | IGNORE]
有唯一索引或主键的话，那就屁用没有
### [ON DUPLICATE KEY UPDATE]
ON DUPLICATE KEY UPDATE为Mysql特有语法，这是个坑，语句的作用，当insert已经存在的记录时，执行Update
```sql
INSERT INTO user_admin_t (_id,password)
VALUES
('1','多条插入1') ,
('UpId','多条插入2')
ON DUPLICATE KEY UPDATE
password =  VALUES(password);
```
## 标准插入
```sql
INSERT INTO users (name, age) VALUES('姚明',25);
insert into [table] (field1,field2) values (...),(...);
```
## set插入
set只能插入一行，标准插入可以插入多行
```sql
INSERT INTO uses SET name = '姚明', age = 25;
```
## select插入
根据查询结果插入
# delete

```sql
DELETE [LOW_PRIORITY] [QUICK] [IGNORE]
    tbl_name[.*] [, tbl_name[.*]] ...
    FROM table_references
    [WHERE where_condition]

DELETE [LOW_PRIORITY] [QUICK] [IGNORE]
    FROM tbl_name[.*] [, tbl_name[.*]] ...
    USING table_references
    [WHERE where_condition]
```
## [LOW_PRIORITY]
## [QUICK]
## [IGNORE]
# update
```sql
UPDATE [LOW_PRIORITY] [IGNORE] table_reference
    SET assignment_list
    [WHERE where_condition]
    [ORDER BY ...]
    [LIMIT row_count]
```
# select
```sql
Syntax:
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
      [HIGH_PRIORITY]
      [STRAIGHT_JOIN]
      [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
      [SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr ...]
    [FROM table_references
      [PARTITION partition_list]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}, ... [WITH ROLLUP]]
    [HAVING where_condition]
    [WINDOW window_name AS (window_spec)
        [, window_name AS (window_spec)] ...]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        export_options
      | INTO DUMPFILE 'file_name'
      | INTO var_name [, var_name]]
    [FOR {UPDATE | SHARE} [OF tbl_name [, tbl_name] ...] [NOWAIT | SKIP LOCKED]
      | LOCK IN SHARE MODE]]
```
## inner join
全连接
## left join
左表全要
## right join
右表全要
## union
## union all
## group by
分组
## having
+ 把结果集按某些列分层不同的组，并对分组后的数据进行聚合操作
+ 可以通过可选的having子句对聚合后的数据进行过滤
```sql
select a.name,b.level,count(*)
from tablea a left tableb b
on a.id = b.id
--where count(*)>3 where子句中不能使用聚合函数
group by a.name,b.level
having count(*) > 3
```
## order by
+ asc升序
+ desc降序
+ order by也可以使用select子句中未出现的列或是函数
## limit
限制返回结果集的行数
+ 常用于数据列表分页
+ 一定要和order by子句配合使用
+ limit 起始偏移量，结果集偏移量
```sql
select * from product order by id desc limit 1,100
```


# sql开发中易犯的错误
+ 使用count(*)判断是否存在符号条件的数据`使用select ... limit 1`
+ 在执行一个更新语句后，使用查询方法判断此更新语句是否有执行成功。`使用row_count()函数判断修改行数`
+ 试图在on条件中过滤不满足条件的记录`在where中进行过滤`
+ 在使用in进行子查询的判断时，在列中未指定正确的表名。如`select a1 from A where a1 in (select a1 from B)`这时尽管B中并不存在a1列数据库也不会报错，而是会列出A表中所有的数据`使用join关联代替子查询`
