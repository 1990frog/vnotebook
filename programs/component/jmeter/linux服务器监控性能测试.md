
# top命令
M按内存排序
P按CPU排序
load average:平均负载，如果cpu是8，这个值超过8就是满负载

# vmstat

# free
free -h

# netstat
-n 拒绝显示别名，能显示数字的全部转化成数字
-l 仅列出有在listen（监听）的服务状态
-p 显示建立相关链接的程序员
-t(tcp) 显示tcp相关选项
-u(udp) 显示udp相关选项

netstat -ntlp

# iostat
对系统磁盘io操作进行监控，它的输出主要显示磁盘的读写操作的统计信息

# sar
# nmon

# crontab
/sbin/service crond status查看定时任务的服务是否启动
start/stop/restart
reload 重写载入配置