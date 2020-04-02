[TOC]

不要求数据绝对意义上的准确
不确保强一致

缺陷：
延迟

解决方案：
基于canal管道的应用

# 安装插件
```
$ sudo nvim /logstash/Gemfile

source "https://gems.ruby-china.com/"

$ bin/logstash-plugin install logstash-integration-jdbc
```


# 全量索引构建
在实际的生产环境中，一般我们的全量索引都是以日为单位来更新，以每天业务低峰期的凌晨2点钟，去全量的更新一个索引，更新完成之后以别名alise来更新掉数据库中的索引名
jdbc.conf
```json
input {
    jdbc {
      # mysql 数据库链接,bigtable为数据库名
      jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/bigtable"
      # 用户名和密码
      jdbc_user => "root"
      jdbc_password => "123456"
      # 驱动
      jdbc_driver_library => "/opt/elk/logstash/bin/mysql/mysql-connector-java-8.0.19.jar"
      # 驱动类名
      jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
      # 分页
      jdbc_paging_enabled => "true"
      # 每页5w
      jdbc_page_size => "50000"
      # 执行的sql 文件路径+名称
      statement_filepath => "/opt/elk/logstash/bin/mysql/jdbc.sql"
      # 设置监听间隔  各字段含义（由左至右）分、时、天、月、年，全部为*默认含义为每分钟都更新
      schedule => "* * * * *"
      # 是否记录上次执行结果, 如果为真,将会把上次执行到的 tracking_column 字段的值记录下来,保存到 last_run_metadata_path 指定的文件中
      record_last_run => "true"
      # 是否需要记录某个column 的值,如果record_last_run为真,可以自定义我们需要 track 的 column 名称，此时该参数就要为 true. 否则默认 track 的是 timestamp 的值.
      use_column_value => "true"
      # 如果 use_column_value 为真,需配置此参数. track 的数据库 column 名,该 column 必须是递增的. 一般是mysql主键
      tracking_column => "id"
      last_run_metadata_path => "/opt/elk/logstash/bin/mysql/last_id"

      # 是否清除 last_run_metadata_path 的记录,如果为真那么每次都相当于从头开始查询所有的数据库记录
      clean_run => "false"

      #是否将 字段(column) 名称转小写
      lowercase_column_names => "false"
    }
}

output {
    elasticsearch {
      # ES的IP地址及端口
      hosts => ["localhost:9200"]
      # 索引名称
      index => "ganta"
  	  document_type => "_doc"
      # 自增ID 需要关联的数据库中有有一个id字段，对应索引的id号
      document_id => "%{id}"
    }
    stdout {
      # JSON格式输出
      codec => json_lines
    }
}

```
jdbc.sql
```sql
select 
a.id,
a.name,
(case when a.gender=0 then '男' else '女' end) as gender,
a.age,b.name as teacher,
c.name as level,
d.name as gang,
e.name as student,
f.name as gest,
g.name as wife 
from 
ganta a 
left join ganta b on b.id = a.teacher 
left join level c on c.code = a.level 
left join gangs d on d.code = a.gang 
left join ganta e on e.id = a.student 
left join gest f on f.code = a.gest 
left join ganta g on g.id = a.wife
where id > :sql_last_value
```
执行
```
$ ./logstash -f mysql/jdbc.conf
```

```json
input {
    jdbc {
      # 设置timezone
      jdbc_default_timezone => "Asia/Shanghai"
      # mysql 数据库链接,bigtable为数据库名
      jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/bigtable"
      # 用户名和密码
      jdbc_user => "root"
      jdbc_password => "123456"
      # 驱动
      jdbc_driver_library => "/opt/elk/logstash/bin/mysql/mysql-connector-java-8.0.19.jar"
      # 驱动类名
      jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
      # 分页
      jdbc_paging_enabled => "true"
      # 每页5w
      jdbc_page_size => "50000"
      # 执行的sql 文件路径+名称
      statement_filepath => "/opt/elk/logstash/bin/mysql/jdbc.sql"
      # 设置监听间隔  各字段含义（由左至右）分、时、天、月、年，全部为*默认含义为每分钟都更新
      schedule => "* * * * *"
      # 是否记录上次执行结果, 如果为真,将会把上次执行到的 tracking_column 字段的值记录下来,保存到 last_run_metadata_path 指定的文件中
      record_last_run => "true"
      # 是否需要记录某个column 的值,如果record_last_run为真,可以自定义我们需要 track 的 column 名称，此时该参数就要为 true. 否则默认 track 的是 timestamp 的值.
      use_column_value => "true"
      # 如果 use_column_value 为真,需配置此参数. track 的数据库 column 名,该 column 必须是递增的. 一般是mysql主键
      tracking_column => "id"
      last_run_metadata_path => "/opt/elk/logstash/bin/mysql/last_id"

      # 是否清除 last_run_metadata_path 的记录,如果为真那么每次都相当于从头开始查询所有的数据库记录
      clean_run => "false"

      #是否将 字段(column) 名称转小写
      lowercase_column_names => "false"
    }
}

output {
    elasticsearch {
      # ES的IP地址及端口
      hosts => ["localhost:9200"]
      # 索引名称
      index => "ganta"
  	  document_type => "_doc"
      # 自增ID 需要关联的数据库中有有一个id字段，对应索引的id号
      document_id => "%{id}"
    }
    stdout {
      # JSON格式输出
      codec => json_lines
    }
}

```
