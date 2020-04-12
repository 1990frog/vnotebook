[TOC]

```
# 每个进程最大连接数（最大连接=连接数x进程数）每个worker允许同时产生多少个链接，默认1024
events {
    use epoll;
    worker_connections  1024;
}
```
配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
# use
指定使用的io模型：
1. select
2. poll
3. kqueue
4. epoll
5. rtsig
6. /dev/poll

其中select 和poll 都是标准的工作模式，kqueue和epoll是高效的工作模式，不同的是epoll用在Linux平台上，而kqueue用在BSD系统中。对于Linux系统，epoll工作模式是首选。
# worker_connections
用于定义nginx每个worker进程的最大连接数，默认1024，最大65536