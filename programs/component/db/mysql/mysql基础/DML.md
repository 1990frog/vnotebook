[TOC]

# DML[Date Manipulation language]

# insert
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

value:
    {expr | DEFAULT}

value_list:
    value [, value] ...

assignment:
    col_name = value

assignment_list:
    assignment [, assignment] ...
```
ON DUPLICATE KEY UPDATE为Mysql特有语法，这是个坑，语句的作用，当insert已经存在的记录时，执行Update
```
INSERT INTO user_admin_t (_id,password) 
VALUES 
('1','多条插入1') ,
('UpId','多条插入2')
ON DUPLICATE KEY UPDATE 
password =  VALUES(password);
```
# delete

# update

# select
```
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

比较运算符

|        比较运算符        |               说明               |
| ---------------------- | ------------------------------- |
| =,>,<,>=,<=,<>,!=       | <>与!=都表示不等于                 |
| BETWEEN [min] AND [max] | 列的值大于等于最小值，小于等于最大值   |
| IS NULL,IS NOT NULL     | 判断列的值是否为null               |
| LIKE,NOT LIKE           | %代表任何数量的字符，_代表任何一个字符 |
| IN,NOT IN              | 判断列的值是否在指定的范围内         |

逻辑运算符

| 逻辑运算符 |                     说明                      |
| -------- | -------------------------------------------- |
| AND,&&   | AND运算符两边的表达式都为真时，返回的结果才为真       |
| OR,`||`  | OR运算符两边的表达式有一条为真，返回的结果就为真       |
| XOR      | XOR运算符两边的表达式一真一假时返回真，两真两假时返回假 |
注：任何运算符和NULL值运算结果都为NULL