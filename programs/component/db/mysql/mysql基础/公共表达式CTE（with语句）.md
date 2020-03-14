[TOC]


公共表达式CTE（Common Table Expressions）
类似子查询

cte生成一个命名临时表，并且只在查询期间有效（oracle with）
cte临时表在一个查询中可以多次引用及自引用

# 概览
CTE与derived table最大的不同之处是：
+ 可以自引用，递归使用（recursive cte
+ 在语句级别生成独立的临时表. 多次调用只会执行一次
+ 一个cte可以引用另外一个cte

一个CTE语句其实和CREATE [TEMPORARY] TABLE类似，但不需要显式的创建或删除，也不需要创建表的权限。更准确的说，CTE更像是一个临时的VIEW

# 语法
```sql
with_clause:
    WITH [RECURSIVE]
        cte_name [(col_name [, col_name] ...)] AS (subquery)
        [, cte_name [(col_name [, col_name] ...)] AS (subquery)] ...
```
# 可以创建多个试图
```sql
WITH cta1 AS (SELECT sum(k) from sbtest1 where id < 100) ,  
           cta2 AS (SELECT SUM(k) from sbtest2 WHERE id < 100) 
SELECT * FROM cta1 JOIN cta2 ;
+----------+----------+
| sum(k)   | SUM(k)   |
+----------+----------+
| 49529621 | 49840812 |
+----------+----------+
1 row in set (0.00 sec)
```
# 递归
```sql
with recursive cte (n) as (select 1 union all select n+1 from cte where n<5)
select * from cte;
```
递归CTE需要加RECURSIVE关键字，使用Union all来产生结果
```sql
SELECT ...定义初始化值，不引用自身, 同时初始化值的列也定义了cte上的列的个数和类型，可以用cast重定义
UNION ALL
SELECT ....返回更多的值，并定义退出循环条件，这里引用了cte自身
```
## 递归斐波那契数列
```sql
WITH RECURSIVE fibonacci (n, fib_n, next_fib_n) AS
(
  SELECT 1, 0, 1
  UNION ALL
  SELECT n + 1, next_fib_n, fib_n + next_fib_n
    FROM fibonacci WHERE n < 10
)
SELECT * FROM fibonacci;

+------+-------+------------+
| n    | fib_n | next_fib_n |
+------+-------+------------+
|    1 |     0 |          1 |
|    2 |     1 |          1 |
|    3 |     1 |          2 |
|    4 |     2 |          3 |
|    5 |     3 |          5 |
|    6 |     5 |          8 |
|    7 |     8 |         13 |
|    8 |    13 |         21 |
|    9 |    21 |         34 |
|   10 |    34 |         55 |
+------+-------+------------+
10 rows in set (0.00 sec)
```
