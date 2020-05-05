[TOC]

DDL(Data Definition language)：
+ 建立/修改/删除数据库：create/alter/drop database
+ 建立/修改/删除表：create/alter/drop table
+ 建立/删除索引：create/drop index
+ 清空表：truncate table
+ 重命名表：rename table
+ 建立/修改/删除视图：create/alter/drop view

# 库操作
## 建库
语法：
```sql
Description:
Syntax:

-- 指定数据库名称mysql可以使用关键字database与schema
CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name
-- 可指定数据库的特征，如字符集，排序规则
[create_specification] ...
create_specification:
-- 默认字符集
[DEFAULT] CHARACTER SET [=] charset_name
-- 排序规则
| [DEFAULT] COLLATE [=] collation_name
| DEFAULT ENCRYPTION [=] {'Y' | 'N'}
```
demo：
```sql
# 创建数据库
$ create database mydb;
# 切换数据库
$ use mydb;
```
## 删库
```sql
$ drop database
```
# 表操作
语法太长就不分析了，`mysql> \q create table`
## 普通建表：create table
```sql
create table foo(
id int not null auto_increment,
name varchar(8),
constraint pk_foo primary key(id));
```
## 临摹表：create table like
```sql
create table foo like bar;
```
## 根据数据建表：select
```sql
create table tmp as select id,name from foo;
```
## 指定引擎
```sql
create table mytable (name varchar(32)) engine=InnoDB;
```
## UNIQUE和PRIMARY KEY的区别
一个表可以有多个字段声明为UNIQUE，但只能有一个PRIMARY KEY声明；声明为PRIMAY KEY的列不允许有空值，但是声明为UNIQUE的字段允许空值的存在。
## 删表
```sql
drop table foo
```
## 清空表
```sql
truncate table foo
```
## 重命名表
```sql
rename table
```