# 两种安装方式
+ 原生命令安装
+ 官方工具安装

# 原生命令安装--理解架构
## 步骤
1. 配置开启节点
2. meet（通信）
3. 指派槽
4. 主从

## 配置
```
port ${port}
daemonize yes
dir "/opt/redis/redis/data/"
dbfilename "dump-${port}.rdb"
logfile "${port}.log"
cluster-enabled yes
cluster-config-file nodes-${port}.conf
```
## 开启节点
这里启动6个，3主3从
```
redis-server redis-7000.conf
redis-server redis-7001.conf
redis-server redis-7002.conf
redis-server redis-7003.conf
redis-server redis-7004.conf
redis-server redis-7005.conf
```
## meet
```
cluster meet ip port

redis-cli -h 127.0.0.1 -p 7000 cluster meet 127.0.0.1 7001
redis-cli -h 127.0.0.1 -p 7000 cluster meet 127.0.0.1 7002
redis-cli -h 127.0.0.1 -p 7000 cluster meet 127.0.0.1 7003
redis-cli -h 127.0.0.1 -p 7000 cluster meet 127.0.0.1 7004
redis-cli -h 127.0.0.1 -p 7000 cluster meet 127.0.0.1 7005
```

## cluster节点主要配置
```
cluster-enabled yes
# 最好使用默认配置
cluster-node-timeout 15000
cluster-config-file "nodes.conf"
# 集群中一个节点挂掉了，集群就不提供服务了，这个一般不用
cluster-require-full-coverage no
```

## 分配槽
```
cluster addslots slot [slot ...]
redis-cli -h 127.0.0.1 -p 7000 cluster addslots {0...5461}
redis-cli -h 127.0.0.1 -p 7001 cluster addslots {5462...10922}
redis-cli -h 127.0.0.1 -p 7002 cluster addslots {10923...16383}
```
## 设置主从
```
cluster replicate node-id
redis-cli -h 127.0.0.1 -p 7003 cluster replicate ${node-id-7000}
redis-cli -h 127.0.0.1 -p 7004 cluster replicate ${node-id-7001}
redis-cli -h 127.0.0.1 -p 7005 cluster replicate ${node-id-7002}
```

## 实战
### step-1
写一个配置文件
```conf
port 7000
daemonize yes
dir "/opt/redis/data"
logfile "7000.log"
dbfilename "dump-7000.rdb"
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-require-full-coverage no
```
其他的通过sed命令替换
```
$ sed 's/7000/7001/g' redis-7000.conf > reids-7001.conf
......
```
### step-2
启动所有的server
```
$ reids-server 7000.conf
......
```
查看启动结果
```
$ ps -ef | grep redis | grep 700
```
cli一个server
```
$ redis-cli -p 7000
提示CLUSTERDOWN The cluster is down，未meet，未分配槽
```
查看集群状态
```
$ redis-cli -p 7000 cluster nodes
$ redis-cli -p 7000 cluster info
```
### step-3
互相meet
```
$ reids-cli -p 7000 cluster meet ip port
...
都跟7000meet
```
### step-4
分配槽
可以用命令分配，也可以写脚本，脚本如下：
```bash
start=$1
end=$2
port=$3
for slot in 'seq ${start} ${end}'
do
    echo "slot:${slot}"
    reids-cli -p ${port} cluster addslots ${slot}
done
```
### step-4
通过id主从复制
```
查询全部id
$ redis-cli -p 7000 cluster nodes

$ redis-cli -p 7004 cluster replicate [masterid]
```

# Ruby安装脚本
## 环境准备
下载、编译、安装Ruby
安装rubygem redis
安装redis-trib.rb

## 安装rubygem redis
```
wget http://rubygems.org/downloads/redis-3.3.0.gem
gem install -l reids-3.3.0.gem
gem list --check redis gem
```

##  安装redis-trib.rb



## reids-trib.rb搭建集群


有节点数量判断


# 总结
|    方式    |                优缺点                |
| ---------- | ----------------------------------- |
| 原生命令安装 | 理解redis cluster架构</br>生产环境不使用 |
| 官方工具安装 | 高效、准确</br>生产环境可以使用          |
| 其他       | 可视化部署                            |


# reids-cluster安装
redis5.0之后使用c语言，废弃ruby安装了
```
redis-cli --cluster
redis-cli 5.0.8

Usage: redis-cli [OPTIONS] [cmd [arg [arg ...]]]
  -h <hostname>      Server hostname (default: 127.0.0.1).
  -p <port>          Server port (default: 6379).
  -s <socket>        Server socket (overrides hostname and port).
  -a <password>      Password to use when connecting to the server.
                     You can also use the REDISCLI_AUTH environment
                     variable to pass this password more safely
                     (if both are used, this argument takes predecence).
  -u <uri>           Server URI.
  -r <repeat>        Execute specified command N times.
  -i <interval>      When -r is used, waits <interval> seconds per command.
                     It is possible to specify sub-second times like -i 0.1.
  -n <db>            Database number.
  -x                 Read last argument from STDIN.
  -d <delimiter>     Multi-bulk delimiter in for raw formatting (default: \n).
  -c                 Enable cluster mode (follow -ASK and -MOVED redirections).
  --raw              Use raw formatting for replies (default when STDOUT is
                     not a tty).
  --no-raw           Force formatted output even when STDOUT is not a tty.
  --csv              Output in CSV format.
  --stat             Print rolling stats about server: mem, clients, ...
  --latency          Enter a special mode continuously sampling latency.
                     If you use this mode in an interactive session it runs
                     forever displaying real-time stats. Otherwise if --raw or
                     --csv is specified, or if you redirect the output to a non
                     TTY, it samples the latency for 1 second (you can use
                     -i to change the interval), then produces a single output
                     and exits.
  --latency-history  Like --latency but tracking latency changes over time.
                     Default time interval is 15 sec. Change it using -i.
  --latency-dist     Shows latency as a spectrum, requires xterm 256 colors.
                     Default time interval is 1 sec. Change it using -i.
  --lru-test <keys>  Simulate a cache workload with an 80-20 distribution.
  --replica          Simulate a replica showing commands received from the master.
  --rdb <filename>   Transfer an RDB dump from remote server to local file.
  --pipe             Transfer raw Redis protocol from stdin to server.
  --pipe-timeout <n> In --pipe mode, abort with error if after sending all data.
                     no reply is received within <n> seconds.
                     Default timeout: 30. Use 0 to wait forever.
  --bigkeys          Sample Redis keys looking for keys with many elements (complexity).
  --memkeys          Sample Redis keys looking for keys consuming a lot of memory.
  --memkeys-samples <n> Sample Redis keys looking for keys consuming a lot of memory.
                     And define number of key elements to sample
  --hotkeys          Sample Redis keys looking for hot keys.
                     only works when maxmemory-policy is *lfu.
  --scan             List all keys using the SCAN command.
  --pattern <pat>    Useful with --scan to specify a SCAN pattern.
  --intrinsic-latency <sec> Run a test to measure intrinsic system latency.
                     The test will run for the specified amount of seconds.
  --eval <file>      Send an EVAL command using the Lua script at <file>.
  --ldb              Used with --eval enable the Redis Lua debugger.
  --ldb-sync-mode    Like --ldb but uses the synchronous Lua debugger, in
                     this mode the server is blocked and script changes are
                     not rolled back from the server memory.
  --cluster <command> [args...] [opts...]
                     Cluster Manager command and arguments (see below).
  --verbose          Verbose mode.
  --no-auth-warning  Don't show warning message when using password on command
                     line interface.
  --help             Output this help and exit.
  --version          Output version and exit.

Cluster Manager Commands:
  Use --cluster help to list all available cluster manager commands.

Examples:
  cat /etc/passwd | redis-cli -x set mypasswd
  redis-cli get mypasswd
  redis-cli -r 100 lpush mylist x
  redis-cli -r 100 -i 1 info | grep used_memory_human:
  redis-cli --eval myscript.lua key1 key2 , arg1 arg2 arg3
  redis-cli --scan --pattern '*:12345*'

  (Note: when using --eval the comma separates KEYS[] from ARGV[] items)

When no command is given, redis-cli starts in interactive mode.
Type "help" in interactive mode for information on available commands
and settings.
```
## step-1　创建n个redis-conf
先弄一个
```
port 7000
daemonize yes
dir "/opt/redis/data/"
dbfilename "dump-7000.rdb"
logfile "7000.log"
cluster-enabled yes
cluster-config-file nodes-7000.conf
```
拷贝多个
```
$ sudo sed 's/7000/7001/g' 7000.conf > 7001.conf
$ sudo sed 's/7000/7002/g' 7000.conf > 7002.conf
$ sudo sed 's/7000/7003/g' 7000.conf > 7003.conf
$ sudo sed 's/7000/7004/g' 7000.conf > 7004.conf
$ sudo sed 's/7000/7005/g' 7000.conf > 7005.conf
```
## step-2 启动
```
$ sudo redis-server 7000.conf
```
## step-3 创建集群
```
$ sudo redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
$ yes
```
## step-4 检查集群状态
```
$ redis-cli --cluster check 127.0.0.1:7000
$ cluster info
```
