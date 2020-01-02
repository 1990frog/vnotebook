[TOC]

# 用到的命令
info replication
flushall

# 单节点redis风险
+ 机器故障
+ 容量瓶颈
+ QPS瓶颈

# master&&slave概览
1.一个master可以有多个slave
2.一个slave只能有一个master
3.数据流向是单向的，master到slave

# 角色
![15083002-5640e35740e1d735](_v_images/20200102163833632_152043510.png)

+ Redis主从复制主要有两个角色，**主机（master）对外提供读写功能**，**从机（slave）对外只提供读功能**，主机定期把数据同步到从机上保证数据一致性。
+ Redis主机数据同步到从机上有两种方式，一种是全量同步(没得runid)，另一种是增量同步。
+ 主从复制不会阻塞master（fork方式），在数据同步时，master还可以继续处理客户端请求，因为redis会产生一个新的进程来解决同步问题。
一个redis也可以是从也可以是主（树状主从），可以减轻主机压力。

![15083002-5e48221da72ffb5a](_v_images/20200102163912433_952841568.png)
# 配置
只需要修改从服务器上的redis.conf文件：
```config
# slaveof <masterip> <masterport>
# 表示当前【从服务器】对应的【主服务器】的IP是192.168.10.135，端口是6379。
slaveof 192.168.10.135 6379
#我们希望从节点只做读的操作，不希望写入，如果写入会造成主从数据不一致
slave-read-only yes
```
启动主服务器
```
redis-server redis.conf
```
启动从服务器
```
# 查看主服务器信息，可以看到有几个从服务器
redis-cli -p 6379 info Replication
```
# 实现原理
+ redis的主从同步分为两种，分为<font color="red">**全量同步**</font>和<font color="red">**增量同步**</font>。
+ <font color="red">**只有从机第一次连接上主机是全量同步**</font>。
+ 断线重连很有可能触发全量同步也有可能是增量同步<font color="red">**（master判断runid是否一致）**</font>。
![15083002-913bfd511faf6546](_v_images/20200102164507869_508329868.jpg)
# runid
+ Redis服务器的随机标识符（用于 Sentinel 和集群），重启后就会改变。
+ 每个redis服务器，不论主服务器还是从服务，都会有自己的运行id。`PSYNC runid`这个命令，runid是指上一次复制主服务器的运行id，如果没有保存这个id，PSYNC的命令会使用”PSYNC ? -1” 这种形式发送给Master，请求主服务器进行全量复制。
# offset（复制偏移量）
主服务器和从服务器会分别维护一个复制偏移量，主服务器每次向从服务器传播N个字节的数据时，就将自己的复制偏移量的值加上N，从服务器每次收到主服务器传播来的N个字节的数据时，就将自己的复制偏移量值加上N。
![15083002-e32972514d0c3c9f](_v_images/20200102164605202_1945018496.png)
# 复制积压缓冲区（同步断线处理）
**复制积压缓冲区是由主服务器维护一个固定长度（fixed-size）先进先出（FIFO）队列，默认大小是1MB。**
它主要的作用就是当主服务器进行命令传播时，不仅将命令发送给所有从服务器，还会将命令入队到复制积压缓冲区。
如果主服务器向从服务器传播数据时发生断线，主服务器会将复制积压缓冲区偏移量的所有数据都发送给从服务器（发送的是断线之后的的数据）。
![15083002-e49c9e86a5fb5a1c](_v_images/20200102164648220_1801146173.png)
# 如果主从节点偏移量不同可能为
+ 网络
+ 阻塞
+ 缓存区出现问题
# PSYNC执行过程
1. Slave接受从客户端发送过来的slaveof命令。
2. 当前服务器判断自己是否保存Master runid<font color="red">**（是否是第一次复制）**</font>。
3. 如果是第一次复制那么当前服务器向Master发送PSYNC ？ -1命令，主动请求Master进行全量同步。
4. 如果已经父之过Master，那么当前从服务器向Master发送PSYNC runid offset命令。
5. Master接收到PSYNC 命令后首先判断runid是否和本机的id一致，如果runid和本机id不一致则返回+FULLRESYNC runid offset命令执行全量同步操作，当前服务器会将runid保存起来，在下次发送PSUNC时使用。
6. 如果判断runid和本机id一致，Master则会再次判断offset偏移量和本机的偏移量相差有没有超过复制积压缓冲区大小，如果没有那么就给Slave发送CONTINUE，此时Slave只需要等待Master传回失去连接期间丢失的命令；
# 全量同步
Redis的全量同步主要分为三个阶段：
+ 同步快照阶段：Master创建并发送快照给Slave，Slave再入快照并解析。Master同时将此阶段产生的新的命令写入到积压缓冲区中。
+ 同步写缓冲阶段：Master向Slave同步存储在缓冲区的写操作命令。
+ 同步增量阶段：Master向SLave同步写操作命令。

![15083002-3a78be2ee836cabe](_v_images/20200102164805034_911861161.png)

# redis主从复制流程图
![redis主从复制流程图](_v_images/20200102172717814_1822282584.png)


# redis修改配置两种方式
| 方式 |    命令    |   配置   |
| ---- | ---------- | -------- |
| 优点 | 不需重启   | 统一配置 |
| 缺点 | 不便于管理 | 需要重启 |


# 查看偏移量
命令：
```
redis>info replication
role:master
connected_slaves:0
master_replid:711710915ff934f4b00e2a5143da342b5df4340f
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0    #偏移量
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```
# 全量复制传递RDB?
![](_v_images/20191210221751563_402309212.png)
# 全量复制开销
![](_v_images/20191210222028033_2074634944.png)

redis2.8之前都是全量复制

# 部分复制
如果发生了抖动
![](_v_images/20191210222326267_1712647051.png)

# 故障处理

# 自动故障转移

# 主从结构——故障转移

## slave宕掉
![](_v_images/20191210222812172_531453535.png)
## master宕掉
![](_v_images/20191210223014498_1553712826.png)