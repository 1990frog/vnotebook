[TOC]

# syntax
```sql
Syntax:
SELECT
    // 结果集投影，去重
    [ALL | DISTINCT | DISTINCTROW ]
    // 优先级
      [HIGH_PRIORITY]
      [STRAIGHT_JOIN]
      [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
      [SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr ...]
    // 查询表
    [FROM table_references
    // 分表
      [PARTITION partition_list]
    // 查询条件
    [WHERE where_condition]
    // 聚合
    [GROUP BY {col_name | expr | position}, ... [WITH ROLLUP]]
    [HAVING where_condition]
    [WINDOW window_name AS (window_spec)
        [, window_name AS (window_spec)] ...]
    // 排序
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    // 导出
    [INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        export_options
      | INTO DUMPFILE 'file_name'
      | INTO var_name [, var_name]]
    [FOR {UPDATE | SHARE} [OF tbl_name [, tbl_name] ...] [NOWAIT | SKIP LOCKED]
      | LOCK IN SHARE MODE]]
```
# join
## inner join
全连接
## left join
左表全要
## right join
右表全要
# 并联查询
## union 去重
## union all 不去重
# 聚合
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
# order by
+ asc升序
+ desc降序
+ order by也可以使用select子句中未出现的列或是函数

# limit
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
