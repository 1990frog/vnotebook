[TOC]

# 概览
Redis默认不开启。它的出现是为了弥补RDB的不足（同步粒度导致数据的不一致性），所以它采用日志的形式来记录每个写操作，并追加到文件中。Redis重启的会根据日志文件的内容**将写指令从前到后执行一次**以完成数据的恢复工作。

AOF和RDB持久性可以同时启用，而不会出现问题。如果在启动时启用了AOF，Redis将加载AOF，那就是具有更好的持久性保证的文件。
## 操作粒度
aof：每条操作之后
rdb：快照时间点
# 配置
```
//输出路径
dir /bigdiskpath    #保存rdb、log、aof的目录

//aof基本参数
appendonly yes    #使用aof所有功能
appendfilename "appendonly-${port}.aof"    #设置aof名称
appendfsync everysec    #aof同步策略,everysec每秒，no操作系统决定，always每个事件循环

//aof重写
no-appendfsync-on-rewrite yes    #在aof重写的时候是否要做一个aof的判断操作，yes不做操作，aof重写非常消耗性能
auto-aof-rewrite-percentage 100    #aof文件增长比例，指当前aof文件比上次重写的增长比例大小
auto-aof-rewrite-min-size 64mb    #aof文件重写最小的文件大小，即最开始aof文件必须要达到这个文件时才触发，后面的每次重写就不会根据这个变量了(根据上一次重写完成之后的大小).此变量仅初始化启动redis有效.如果是redis恢复时，则lastSize等于初始aof文件大小.

aof-load-truncated yes    #指redis在恢复时，会忽略最后一条可能存在问题的指令。默认值yes。即在aof写入时，可能存在指令写错的问题(突然断电，写了一半)，这种情况下，yes会log并继续，而no会直接恢复失败.
```
# Linux内核参数
另外，与aof重写相关的一个linux内核参数即是 overcommit_memory。即在进行重写时，如何分配子进程内存的问题。(重写是后台重写，会分配子进程).默认值为0,建议设置为1,以保证子进程内存能够分配成功(即使用copyOnWrite内存分配策略，在没有set命令时会和主进程使用同一份内存)，并且不会判断当前内存是否够用。
# AOF的三种策略
| appendfsync 选项的值 |                                                         flushAppendOnlyFile 函数的行为                                                         |        优点         |              缺点              |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------- | ----------------------------- |
| always             | 将 aof_buf 缓冲区中的所有内容写入并同步到 AOF 文件。                                                                                                 | 不丢失数据            | IO开销较大，一般的sata盘只有几百TPS |
| everysec           | 将 aof_buf 缓冲区中的所有内容写入到 AOF 文件， 如果上次同步 AOF 文件的时间距离现在超过一秒钟， 那么再次对 AOF 文件进行同步， 并且这个同步操作是由一个线程专门负责执行的。 | 每秒一次fsync丢1秒数据 | 丢1秒数据                       |
| no                 | 将 aof_buf 缓冲区中的所有内容写入到 AOF 文件， 但并不对 AOF 文件进行同步， 何时同步由操作系统来决定。                                                        | 不用管               | 不可控                          |

## AOF 持久化的效率和安全性
+ 服务器配置 appendfsync 选项的值直接决定 AOF 持久化功能的效率和安全性。
+ 当 appendfsync 的值为 always 时， 服务器在每个事件循环都要将 aof_buf 缓冲区中的所有内容写入到 AOF 文件， 并且同步 AOF 文件， 所以 always 的效率是 appendfsync 选项三个值当中最慢的一个， 但从安全性来说， always 也是最安全的， 因为即使出现故障停机， AOF 持久化也只会丢失一个事件循环中所产生的命令数据。
+ 当 appendfsync 的值为 everysec 时， 服务器在每个事件循环都要将 aof_buf 缓冲区中的所有内容写入到 AOF 文件， 并且每隔超过一秒就要在子线程中对 AOF 文件进行一次同步： 从效率上来讲， everysec 模式足够快， 并且就算出现故障停机， 数据库也只丢失一秒钟的命令数据。
+ 当 appendfsync 的值为 no 时， 服务器在每个事件循环都要将 aof_buf 缓冲区中的所有内容写入到 AOF 文件， 至于何时对 AOF 文件进行同步， 则由操作系统控制。
+ 因为处于 no 模式下的 flushAppendOnlyFile 调用无须执行同步操作， 所以该模式下的 AOF 文件写入速度总是最快的， 不过因为这种模式会在系统缓存中积累一段时间的写入数据， 所以该模式的单次同步时长通常是三种模式中时间最长的： 从平摊操作的角度来看， no 模式和 everysec 模式的效率类似， 当出现故障停机时， 使用 no 模式的服务器将丢失上次同步 AOF 文件之后的所有写命令数据。

##  文件的写入和同步
+ 为了提高文件的写入效率， 在现代操作系统中， 当用户调用 write 函数， 将一些数据写入到文件的时候， 操作系统通常会将写入数据暂时保存在一个内存缓冲区里面， 等到缓冲区的空间被填满、或者超过了指定的时限之后， 才真正地将缓冲区中的数据写入到磁盘里面。
+ 这种做法虽然提高了效率， 但也为写入数据带来了安全问题， 因为如果计算机发生停机， 那么保存在内存缓冲区里面的写入数据将会丢失。
+ 为此， 系统提供了 **fsync 和 fdatasync** 两个同步函数， 它们可以强制让操作系统立即将缓冲区中的数据写入到硬盘里面， 从而确保写入数据的安全性。

# 开发运维常见问题
1. fork操作
2. 进程外开销
3. AOF追加阻塞
4. 单机多实例部署

# fork操作
1. 同步操作
2. 与内存量息息相关：内存越大，耗时越长（与机器类型有关）
3. info:latest_fork_usec 查看持久化时间

# 改善fork
1. 优化使用物理机或者高效支持fork操作的虚拟化技术
2. 控制Redis实例最大可用内存：maxmemory
3. 合理配置Linux内存分配策略：vm.overcommit_memory=1
4. 降低fork频率：例如放宽AOF重写自动触发时机，不必要的全量复制

# 子进程开销和优化
1.cpu
开销：rdb和aof文件生成，属于cpu密集型
优化：不做cpu绑定，不和cpu密集型部署
2.内存
开销：fork内存开销，copy-on-write。
优化：echo never > /sys/kernel/mm/transparent_hugepage/enabled
3.硬盘
开销：aof和rdb文件写入，可以结合iostat，iotop分析

# 硬盘优化
1.不要和高硬盘负载服务部署一起：存储服务、消息队列等
2.no-appendfsync-on-rewrite=yes
3.根据写入量决定磁盘类型：例如ssd
4.单机多实例持久化文件目录可以考虑分盘

# aof追加阻塞
![](https://gitee.com/caijingquan/imagebed/raw/master/1602320211_20191208135325755_1778392891.png)

如果AOF文件fsync同步时间大于2s，Redis主进程就会阻塞；
如果AOF文件fsync同步时间小于2s，Redis主进程就会返回；
其实这样做的目的是为了保证文件安全性的一种策略。

AOF追加阻塞会产生两位问题：
（1）fsync大于2s时候，会阻塞redis主进程，我们都知道redis主进程是用来执行redis命令的，是不能阻塞的。
（2）虽然每秒everysec刷盘策略，但是实际上不是丢失1s数据，实际有可能丢失2s数据。

# aof阻塞定位
## 通过Redis日志定位
```
Asynchronous AOF fsync is taking too long (disk is busy?).
Writing the AOF buffer without waiting for fsync to complete,
this may slow down Redis
```
## 通过Redis命令定位
```
info Persistence
aof_delayed_fsync:100
aof_delayed_fsync:100这个是同步延迟个数历史总数统计，可能查不出来在某个时间发生阻塞，你也可以自己记录这个信息。
```
## 通过Linux命令top定位
查看io的负载情况，一般发生这种问题都是磁盘IO太高导致的问题，top一般就能看到了不需要其他工具。
# Redis 持久化 之 AOF 和 RDB 同时开启
听AOF的，RDB与AOF同时开启 默认无脑加载AOF的配置文件
相同数据集，AOF文件要远大于RDB文件，恢复速度慢于RDB
AOF运行效率慢于RDB,但是同步策略效率好，不同步效率和RDB相同

1、Rewrite 重写
AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制,当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof

2、Redis如何实现重写？
AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。

3、何时重写
重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一定条件才会进行重写。

系统载入时或者上次重写完毕时，Redis会记录此时AOF大小，设为base_size,如果Redis的AOF当前大小>= base_size +base_size*100% (默认)且当前大小>=64mb(默认)的情况下，Redis会对AOF进行重写。

# AOF的优点
备份机制更稳健，丢失数据概率更低。
可读的日志文本，通过操作AOF稳健，可以处理误操作。

# AOF的缺点
比起RDB占用更多的磁盘空间。
恢复备份速度要慢。
每次读写都同步的话，有一定的性能压力。
存在个别Bug，造成不能恢复

