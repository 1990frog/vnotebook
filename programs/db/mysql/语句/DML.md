[TOC]

Date Manipulation language

# 导出导入
```
# 导出
$ mysqldump -uroot -proot --databases mydb > ~/download/dmp.sql
# 导入
$ mysql -uroot -proot < dmp.sql
```

# 创建用户
基础命令：
```sql
CREATE USER [IF NOT EXISTS]
    user [auth_option] [, user [auth_option]] ...
    DEFAULT ROLE role [, role ] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH resource_option [resource_option] ...]
    [password_option | lock_option] ...
```
密码策略：
```sql
auth_option: {
    IDENTIFIED BY 'auth_string'
  | IDENTIFIED BY RANDOM PASSWORD
  | IDENTIFIED WITH auth_plugin
  | IDENTIFIED WITH auth_plugin BY 'auth_string'
  | IDENTIFIED WITH auth_plugin BY RANDOM PASSWORD
  | IDENTIFIED WITH auth_plugin AS 'auth_string'
}
```
安全传输层协议
```sql
tls_option: {
   SSL
 | X509
 | CIPHER 'cipher'
 | ISSUER 'issuer'
 | SUBJECT 'subject'
}
```
资源限制
```
resource_option: {
    MAX_QUERIES_PER_HOUR count # 设定每小时能查询的最大次数
  | MAX_UPDATES_PER_HOUR count # 设定每小时最大次数
  | MAX_CONNECTIONS_PER_HOUR count
  | MAX_USER_CONNECTIONS count # 最大连接数
}
password_option: {
    PASSWORD EXPIRE [DEFAULT | NEVER | INTERVAL N DAY]
  | PASSWORD HISTORY {DEFAULT | N}
  | PASSWORD REUSE INTERVAL {DEFAULT | N DAY}
  | PASSWORD REQUIRE CURRENT [DEFAULT | OPTIONAL]
  | FAILED_LOGIN_ATTEMPTS N
  | PASSWORD_LOCK_TIME {N | UNBOUNDED}
}

lock_option: {
    ACCOUNT LOCK
  | ACCOUNT UNLOCK
}

>show plugins //查询可用插件
```
远程登录访问限制
```
$ create user newuser@'192.168.1.%' identified with 'mysql_native_password' by '123456'
$ create user newuser@'192.168.1.%' identified by '123456'
```
# 赋予权限
| 权限名称 |       说明        |
| ------- | ---------------- |
| Insert  | 向表中插入数据的权限 |
| Delete  | 删除表中数据的权限   |
| Update  | 修改表中数据的权限   |
| Select  | 查询表中数据的权限   |
| Execute | 执行存储过程的权限   |

赋予在'192.168.1.%'访问控制下的tom用户mysql库user表user和host列的查询权限
```
grant select(user,host) on mysql.user to tom@'192.168.1.%';
```
对列授权，用于对某些用户隐藏敏感信息的情况
```sql
grant select on mysql.user to tom;
grant select on mysql.* to tom;
```

grant命令的注意事项：
+ 8.0使用grant授权的数据库账号必须存在
+ 用户使用grant命令授权必须具有grant option的权限
+ 获取命令\h grant

# 收回权限
```sql
grant select,delete,insert,update on msyql.* to tom@'192.168.1.%'

revoke delete,insert,update on msyql.* from tom@'192.168.1.%'
```

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
