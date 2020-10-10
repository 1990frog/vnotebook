[TOC]

![Redis Sentinel故障转移](https://gitee.com/caijingquan/imagebed/raw/master/1602320283_20200106141507641_1706868549.png)

# 心得
+ 服务端高可用
+ 客户端高可用
+ Redis主从切换数据丢失？
可以看出，使用Sentinel命令和发布订阅两种机制就能很好的实现和客户端的集成整合：
使用`get-Master-addr-by-name`和`Slaves`指令可以获取当前的Master和Slaves的地址和信息；而当发生故障转移时，即Master发生切换，可以通过订阅的+switch-Master事件获得最新的Master信息。
# 概览
+ Sentinel是特殊的Redis，其默认端口26379，用于监控Redis集群中Master、Slave、Sentinel各成员
+ Master主服务器发生故障的时候，可以实现Master和Slave服务器的切换，保证系统的高可用
+ Sentinel当前最新的稳定版本称为Sentinel2(与之前的Sentinel 1区分开来）

随着Redis2.8的安装包一起发行。安装完Redis2.8后，可以在Redis2.8/src/里面找到Redis-Sentinel的启动程序；**强烈建议：如果你使用的是Redis2.6(Sentinel版本为Sentinel 1)，你最好应该使用Redis2.8版本的Sentinel 2，因为Sentinel 1有很多的Bug，已经被官方弃用，所以强烈建议使用Redis2.8以及Sentinel2**

Redis2.6中Sentinel的bug有哪些？

# 配置持久化（纠正）
Snetinel的状态会被持久化地写入Sentinel的配置文件中。每次当收到一个新的配置时，或者新创建一个配置时，配置会被持久化到硬盘中，并带上配置的版本戳。这意味着，可以安全的停止和重启Sentinel进程。

# Sentinel三大使命
+ 监控（Monitoring）：Sentinel会不断检查Master和Slave是否运行正常
+ 提醒（Notification）：当被监控的某个Redis节点出现问题时， Sentinel可以通过api向管理员或者其他应用程序发送通知（调用脚本）
+ 自动故障转移（Automatic failover）：failover

# 监控（三个定时任务）
## 定时任务一：每1秒每个Sentinel对其他Sentinel和Redis执行ping
+ 心跳检查、失败判定依据
![1173043-20180613115036920-1108346645](https://gitee.com/caijingquan/imagebed/raw/master/1602320282_20200106140437424_177469351.png)
## 定时任务二：每10秒每个Sentinel对Master和Slave执行info
+ 发现Slave节点
+ 确认主从关系

![1173043-20180613113556319-876141444](https://gitee.com/caijingquan/imagebed/raw/master/1602320280_20200106140223328_1665992046.png)
## 定时任务三：每2秒每个Sentinel通过Master节点的channel（发布订阅的频道）交换信息（pub/sub）
+ 通过`__Sentinel__:hello`频道交互
+ 交互对节点的“看法”和自身信息

![1173043-20180613114216222-8932705](https://gitee.com/caijingquan/imagebed/raw/master/1602320281_20200106140339734_2052542565.png)
上图的原理就是:
订阅这个channel的所有Sentinel，一旦其中一个Sentinel发布消息到这个chennel其他订阅这个channel的Sentinel就会收到消息，它们就是这样传递消息

## Sentinel之间和Slaves之间的自动发现机制（利用Master）
虽然Sentinel集群中各个Sentinel都互相连接彼此来检查对方的可用性以及互相发送消息。<font color="red">但是你不用在任何一个Sentinel配置任何其它的Sentinel的节点。因为Sentinel利用了Master的【发布/订阅】机制去自动发现其它也监控了统一Master的Sentinel节点。</font>
1. Sentinel发现Slave：通过向Master发送`info`命令，频率是10s
2. Sentinel发现新Sentinel：通过订阅Master的`__Sentinel__:hello`频道，频率是2s
## 举个例子
假设有一个名为myMaster的地址为192.168.10.202:6379。
一开始，集群中所有的Sentinel都知道这个地址，于是为myMaster的配置打上版本号1。
一段时候后myMaster死了，有一个Sentinel被授权用版本号2对其进行failover。
如果failover成功了，假设地址改为了192.168.10.202:9000，此时配置的版本号为2，进行failover的Sentinel会将新配置广播给其他的Sentinel，由于其他Sentinel维护的版本号为1，发现新配置的版本号为2时，版本号变大了，说明配置更新了，于是就会采用最新的版本号为2的配置。
这意味着Sentinel集群保证了第二种活跃性：一个能够互相通信的Sentinel集群最终会采用版本号最高且相同的配置。

# 提醒
## 执行脚本
当被监控的某个Redis节点出现问题时， Sentinel可以通过api向管理员或者其他应用程序发送通知（调用脚本）
```config
Sentinel notification-script <Master-name> <script-path>
Sentinel notification-script myMaster /var/Redis/notify.sh
```
通知型脚本:
+ 当Sentinel有任何警告级别的事件发生时（**比如说Redis实例的主观失效和客观失效等等**），将会去调用这个脚本，这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。
+ 调用该脚本时，将传给脚本两个参数，一个是事件的类型，一个是事件的描述
+ 如果Sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则Sentinel无法正常启动成功
## 发布与订阅信息（Sentinel的日志文件里可以看到这些信息）
客户端可以将Sentinel看作是一个只提供了订阅功能的Redis服务器：
你不可以使用`PUBLISH`命令向这个服务器发送信息， 但你可以用`SUBSCRIBE`命令或者`PSUBSCRIBE`命令， 通过订阅给定的频道来获取相应的事件提醒。

事件有对应的频道：比如说， 名为`+sdown`的频道就可以接收所有实例进入主观下线（SDOWN）状态的事件。

通过执行`PSUBSCRIBE *`命令可以接收所有事件信息（即订阅所有消息）
## 频道
```
第一个英文单词是频道/事件的名字， 其余的是数据的格式
instance details的格式是：<instance-type> <name> <ip> <port> @ <Master-name> <Master-ip> <Master-port>
```
### +reset-Master <instance details>
当Master被重置时
### +Slave <instance details>
当检测到一个Slave并添加进Slave列表时
### +failover-state-reconf-Slaves <instance details>
Failover状态变为reconf-Slaves状态时
### +failover-detected <instance details>
当failover发生时
### +Slave-reconf-sent <instance details>
Sentinel发送SlaveOF命令把它重新配置时
### +Slave-reconf-inprog <instance details>
Slave被重新配置为另外一个Master的Slave，但数据复制还未发生时
### +Slave-reconf-done <instance details>
Slave被重新配置为另外一个Master的Slave并且数据复制已经与Master同步时
### -dup-Sentinel <instance details>
删除指定Master上的冗余Sentinel时 (当一个Sentinel重新启动时，可能会发生这个事件)
### +Sentinel <instance details>
当Master增加了一个Sentinel时
### +sdown <instance details>
进入SDOWN状态时
### -sdown <instance details>
离开SDOWN状态时
### +odown <instance details>
进入ODOWN状态时
### -odown <instance details>
离开ODOWN状态时
### +new-epoch <instance details>
当前配置版本被更新时
### +try-failover <instance details>
达到failover条件，正等待其他Sentinel的选举
### +elected-leader <instance details>
被选举为去执行failover的时候
### +failover-state-select-Slave <instance details>
开始要选择一个Slave当选新Master时
### no-good-Slave <instance details>
没有合适的Slave来担当新Master
### selected-Slave <instance details>
找到了一个适合的Slave来担当新Master
### failover-state-send-Slaveof-noone <instance details>
当把选择为新Master的Slave的身份进行切换的时候
### failover-end-for-timeout <instance details>
failover由于超时而失败时
### failover-end <instance details>
failover成功完成时
### switch-Master <Master name> <oldip> <oldport> <newip> <newport>
当Master的地址发生变化时。通常这是客户端最感兴趣的消息了
### +tilt
进入Tilt模式
### -tilt
退出Tilt模式

# 故障转移（failover）
## 流程
1. 每个Sentinel进程每秒钟一次的频率向整个集群中**Master**、**Slave**以及其它**Sentinel**进程发送一个`ping`命令<font color="red">【监听】</font>
2. 如果一个实例（instance）距离最后一次有效回复`ping`命令超过`down-after-milliseconds`选项所指定的值，这个实例会被Sentinel进程标记为**主观下线**<font color="red">【初步确认目标】</font>
3. 如果一个Master主服务器被标记为主观下线，则正在监视这个Master主服务器的所有Sentinel进程要以每秒一次的频率确认Master主服务器的确进入了主观下线状态<font color="red">【复查】</font>
4. 当有足够数量的Sentinel进程（大于等于配置文件指定的值）在指定的时间范围内确认Master主服务器进入了主观下线状态， 则Master主服务器会被标记为客观下线<font color="red">【定性】</font>
5. 在一般情况下， 每个Sentinel进程会以每10秒一次的频率向集群中的所有Master主服务器、Slave从服务器发送`info`命令<font color="red">【查询下线】</font>
6. 当Master主服务器被Sentinel进程标记为客观下线时，Sentinel进程向下线的Master主服务器的所有Slave从服务器发送`info`命令的频率会从10秒一次改为每秒一次【一个faiover要想被成功实行，Sentinel必须能够向选为Master的Slave发送`SlaveOF NO ONE`命令，然后能够通过INFO命令看到新Master的配置信息】
7. 若没有足够数量的Sentinel进程同意Master已经下线， Master主服务器的客观下线状态就会被移除。若Master主服务器重新向Sentinel进程发送 `ping`命令返回有效回复，Master主服务器的主观下线状态就会被移除
## sdown&odown
1. 每个Sentinel实例在启动后，都会和已知的Slaves/Master以及其他Sentinels建立TCP连接，并周期性发送PING(默认为1秒)
2. 在交互中，如果Redis-server无法在`down-after-milliseconds`时间内响应或者响应错误信息，都会被认为此Redis-server处于SDOWN状态。
2. 如果SDOWN的server为Master，那么此时Sentinel实例将会向其他Sentinel间歇性(一秒)发送`is-Master-down-by-addr <ip> <port>`指令并获取响应信息，如果足够多的Sentinel实例检测到Master处于SDOWN，那么此时当前Sentinel实例标记Master为ODOWN...其他Sentinel实例做同样的交互操作。
3. 配置项`Sentinel monitor <Mastername> <Masterip> <Masterport> <quorum>`，如果检测到Master处于SDOWN状态的Slave个数达到`<quorum>`，那么此时此Sentinel实例将会认为Master处于ODOWN。每个Sentinel实例将会间歇性(10秒)向Master和Slaves发送`INFO`指令，如果Master失效且没有新Master选出时，每1秒发送一次`INFO`；`INFO`的主要目的就是获取并确认当前集群环境中Slaves和Master的存活情况。
4. 经过上述过程后，所有的Sentinel对Master失效达成一致后，开始failover.
## 主观下线sdown(Subjectively Down)：每个Sentinel节点对Redis节点失败的“偏见”
+ 所谓主观下线，指的是单个Sentinel实例对服务器做出的下线判断，即单个Sentinel认为某个服务下线（有可能是接收不到订阅，网络不通等等原因）
+ 主观下线就是说如果服务器在`down-after-milliseconds`给定的毫秒数之内， 没有返回Sentinel发送的`ping`命令的回复`pong`， 或者返回一个错误， 那么 Sentinel将这个服务器标记为主观下线
+ Sentinel会以每秒一次的频率向所有与其建立了命令连接的实例（Master，从服务，其他Sentinel）发`ping`命令，通过判断是有效回复，还是无效回复来判断实例时候在线（对该Sentinel来说是“主观在线”）
+ Sentinel配置文件中的`down-after-milliseconds`设置了判断主观下线的时间长度，如果实例在`down-after-milliseconds`毫秒内，返回的都是无效回复，那么Sentinel回认为该实例已（主观）下线，修改其flags状态为SRI_S_DOWN。如果多个Sentinel监视一个服务，<font color="red">有可能存在多个Sentinel的`down-after-milliseconds`配置不同，这个在实际生产中要注意</font>
## 客观下线odown(Objectively Down)：所有Sentinel节点对Redis节点失败“达成共识”（超过quorum个统一）
+ 客观下线，指的是多个Sentinel实例在对同一个服务器做出sdown判断， 并且通过`Sentinel is-Master-down-by-addr`命令互相交流之后， 得出的服务器下线判断，然后开启failover
+ 客观下线就是说只有在足够数量的Sentinel都将一个服务器标记为主观下线之后， 服务器才会被标记为客观下线（odown）
+ 只有当Master被认定为客观下线时，**才会发生故障迁移(failover)**
+ 当Sentinel监视的某个服务主观下线后，Sentinel会询问其它监视该服务的Sentinel，看它们是否也认为该服务主观下线，接收到足够数量（这个值可以配置：quorum）的Sentinel判断为主观下线，既任务该服务客观下线，并对其做故障转移操作
+ Sentinel通过发送`Sentinel is-Master-down-by-addr ip port current_epoch runid`，（ip：主观下线的服务id，port：主观下线的服务端口，current_epoch：Sentinel的纪元，runid：*表示检测服务下线状态，如果是Sentinel运行id，表示用来选举领头Sentinel）来询问其它Sentinel是否同意服务下线。
+ 一个Sentinel接收另一个Sentinel发来的`is-Master-down-by-addr`后，提取参数，根据ip和端口，检测该服务时候在该Sentinel主观下线，并且回复`is-Master-down-by-addr`，回复包含三个参数：down_state（1表示已下线，0表示未下线），leader_runid（领头sentinal id），leader_epoch（领头Sentinel纪元）。
+ Sentinel接收到回复后，根据配置设置的下线最小数量，达到这个值，既认为该服务客观下线。
+ 客观下线条件只适用于主服务器(Master)： 对于任何其他类型的Redis实例，Sentinel 在将它们判断为下线前不需要进行协商， 所以从服务器或者其他Sentinel永远不会达到客观下线条件。只要一个Sentinel 发现某个主服务器进入了客观下线状态， 这个Sentinel 就可能会被其他Sentinel 推选出， 并对失效的主服务器执行自动故障迁移操作。

# 自动故障转移（failover）
## 流程
在Leader触发failover之前，首先wait数秒(随即0~5)，以便让其他Sentinel实例准备和调整(有可能多个leader)，如果一切正常，那么leader就需要开始将一个Slave提升为Master，此Slave必须为状态良好(不能处于SDOWN/ODOWN状态)且权重值最低(Redis.conf中)的，当Master身份被确认后，开始failover
1. `+failover-triggered`: Leader开始进行failover，此后紧跟着`+failover-state-wait-start`，wait数秒。
2. `+failover-state-select-Slave`: Leader开始查找合适的Slave
3. `+selected-Slave`: 已经找到合适的Slave
4. `+failover-state-sen-Slaveof-noone`: Leader向Slave发送`Slaveof no one`指令，此时Slave已经完成角色转换，此Slave即为Master
5. `+failover-state-wait-promotition`: 等待其他Sentinel确认Slave
6. `+promoted-Slave`：确认成功
7. `+failover-state-reconf-Slaves`: 开始对Slaves进行reconfig操作。
8. `+Slave-reconf-sent`:向指定的Slave发送`Slaveof`指令，告知此Slave跟随新的Master
9. `+Slave-reconf-inprog`: 此Slave正在执行Slaveof + SYNC过程，如过Slave收到`+Slave-reconf-sent`之后将会执行Slaveof操作。
10. `+Slave-reconf-done`: 此Slave同步完成，此后leader可以继续下一个Slave的reconfig操作。循环G）
11. `+failover-end`: 故障转移结束
12. `+switch-Master`：故障转移成功后，各个Sentinel实例开始监控新的Master。
## 选择“合适的”Slave节点
1. 选择Slave-priority（Slave节点优先级）最高的Slave节点，如果存在则返回，不存在则继续（优先级为0永远不会被选为master）
2. 选择复制偏移量最大的Slave节点（复制的最完整），如果存在则返回，不存在则继续
3. 选择runId最小的Slave节点（年龄最大的节点）

注意：
一个Redis无论是Master还是Slave，都必须在配置中指定一个Slave优先级。要注意到Master也是有可能通过failover变成Slave的。
如果一个Redis的Slave优先级配置为0，那么它将永远不会被选为Master。但是它依然会从Master哪里复制数据。

# 总结
+ Redis Sentinel是Redis的高可用实现方案：故障发现、故障自动转移、配置中心、客户端通知
+ Redis Sentinel从2.8版本开始才正式生产可用，之前版本生产不可用
+ 尽可能在不同物理机上部署Redis Sentinel所有节点
+ Redis Sentinel中的Sentinel节点个数应该为大于等于3且最好为奇数
+ Redis Sentinel中的数据节点与普通数据节点没有区别
+ 客户端初始化时连接的是Sentinel节点集合，不再是具体的Redis节点，但Sentinel只是配置中心不是代理
+ Redis Sentinel通过三个定时任务实现了Sentinel节点对于主节点、从节点、其余Sentinel节点的监控
+ Redis Sentinel在对节点做失败判定时分为主观下线和客观下线
+ 看懂Redis Sentinel故障转移日志对于Redis Sentinel以及问题排查非常有帮助
+ Redis Sentinel实现读写分离高可用可以依赖Sentinel节点的消息通知，获取Redis数据节点的状态变化
