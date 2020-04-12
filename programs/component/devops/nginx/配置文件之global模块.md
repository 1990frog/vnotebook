[TOC]

配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
```
# user  nobody;

# 开启进程数 <=CPU数
# worker_processes number | auto;
worker_processes  1;

# 错误日志保存位置
# error_log  logs/error.log;
# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;

# 进程号保存文件
# pid        logs/nginx.pid;
```
# user 指定运行Nginx服务器的用户（组）
user是个主模块指令，指定Nginx Worker进程运行以及用户组
+ user：指定可以运行Nginx服务器的用户；
+ group：可选项，可以运行Nginx服务器的用户组。

如果user指令不配置或者配置为user nobody nobody，默认由nobody账户运行。
# worker process 设置worker进程数
woker_processes是个主模块指令，制定了Nginx要开启的进程数。每个Nginx进程平均耗费10M~12M内存。建议指定和CPU的数量一致即可。
指令格式：
+ number : Nginx 进程最多可以产生的worker process 数。
+ auto ： Nginx 进程将自动检测

在按照上面的配置格式配置了之后，假如上面的数目是2，那么启动Nginx服务器后，在后台主机上查看Nginx的进程情况，可以看到应该是有2个Nginx进程。