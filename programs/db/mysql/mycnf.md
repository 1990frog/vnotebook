https://www.jianshu.com/p/f74ddf4f4372
```
#以下选项会被MySQL客户端应用读取。注意只有MySQL附带的客户端应用程序保证可以读取这段内容。如果你想你自己的MySQL应用程序获取这些值。需要在MySQL客户端库初始化的时候指定这些选项。
[client]
port=3306
socket=/run/mysqld/mysqld.sock

[mysqld]
user=mysql
port=3306
bind-address=0.0.0.0
server-id=1

socket=/run/mysqld/mysqld.sock
pid_file=/usr/local/mysql/data/mysqld.pid
basedir=/usr/local/mysql
datadir=
tmpdir=




default_storage_engine=InnoDB
character-set-server=utf8




# Skip #
#禁止 MySQL 对外部连接进行 DNS 解析，使用这一选项可以消除 MySQL 进行 DNS 解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用 IP 地址方式，否则 MySQL 将无法正常处理连接请求！
skip_name_resolve=1
#不使用系统锁定，要使用 myisamchk,必须关闭服务器 ,避免 MySQL的外部锁定，减少出错几率增强稳定性。
skip_external_locking=1
#不能使用连接文件，多个客户可能会访问同一个数据库，因此这防止外部客户锁定 MySQL 服务器。 该选项默认开启
skip_symbolic_links=1






explicit_defaults_for_timestamp = off
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
#read_only=on
```