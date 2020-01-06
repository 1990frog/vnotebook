[TOC]

![redis sentinel故障转移](_v_images/20200106141507641_1706868549.png)
# 服务端高可用
# 客户端高可用
# 哨兵（Sentinel）机制
+ sentinel进程是用于监控redis集群中master主服务器工作的状态
+ sentinel是特殊的redis，其默认端口26379
+ master主服务器发生故障的时候，可以实现master和slave服务器的切换，保证系统的高可用**（主从切换会存在损耗？）**
# Sentinel版本
Sentinel当前最新的稳定版本称为Sentinel2(与之前的Sentinel 1区分开来）
随着redis2.8的安装包一起发行。安装完Redis2.8后，可以在redis2.8/src/里面找到redis-sentinel的启动程序
**强烈建议：如果你使用的是redis2.6(sentinel版本为sentinel 1)，你最好应该使用redis2.8版本的sentinel 2，因为sentinel 1有很多的Bug，已经被官方弃用，所以强烈建议使用redis2.8以及sentinel2**
# Sentinel状态持久化
snetinel的状态会被持久化地写入sentinel的配置文件中。每次当收到一个新的配置时，或者新创建一个配置时，配置会被持久化到硬盘中，并带上配置的版本戳。这意味着，可以安全的停止和重启sentinel进程。
# 哨兵的作用
## 监控（Monitoring）
sentinel会不断检查master和slave是否运行正常
## 提醒（Notification）
当被监控的某个redis节点出现问题时, sentinel可以通过api向管理员或者其他应用程序发送通知
## 自动故障转移（Automatic <font color="red">failover</font>）
当master不能正常操作时哨兵会开始一次故障转移
1. 它会将失效的master的其中一个slave升级为新的master，并让其他slave改为复制新的master。<font color="red">(此时是全量复制？redis4之后好像有骚操作)</font>
2. 当客户端试图连接失效的master时，集群会向客户端显示新的master的地址
3. master和slave切换后，master的redis.conf、slave的reids.conf和senisentinel的sentinel.conf配置文件的内容都会相应的改变，即，master主服务器的redis.conf配置文件中会多一行slaveof的配置，sentinel.conf的监控目标会随之调换。

![15083002-718b61c5cddef066](_v_images/20200106113140851_1755463594.png)
# sentinel三个定时任务
## 每10秒每个sentinel对master和slave执行info
+ 发现slave节点
+ 确认主从关系

![1173043-20180613113556319-876141444](_v_images/20200106140223328_1665992046.png)
## 每2秒每个sentinel通过master节点的channel（发布订阅的频道）交换信息（pub/sub）
+ 通过__sentinel__:hello频道交互
+ 交互对节点的“看法”和自身信息

![1173043-20180613114216222-8932705](_v_images/20200106140339734_2052542565.png)
上图的原理就是:
订阅这个channel的所有sentinel,一旦其中一个sentinel发布消息到这个chennel其他订阅这个channel的sentinel就会收到消息,它们就是这样传递消息
## 每1秒每个sentinel对其他sentinel和redis执行ping
+ 心跳检查、失败判定依据
![1173043-20180613115036920-1108346645](_v_images/20200106140437424_177469351.png)
# Sentinel故障判断流程
1. 每个sentinel进程每秒钟一次的频率向整个集群中**master**、**slave**以及其它**sentinel**进程发送一个`ping`命令<font color="red">【监听】</font>
2. 如果一个实例（instance）距离最后一次有效回复`ping`命令超过`down-after-milliseconds`选项所指定的值，这个实例会被sentinel进程标记为**主观下线**<font color="red">【初步确认目标】</font>
3. 如果一个master主服务器被标记为主观下线，则正在监视这个master主服务器的所有sentinel进程要以每秒一次的频率确认master主服务器的确进入了主观下线状态<font color="red">【复查】</font>
4. 当有足够数量的sentinel进程（大于等于配置文件指定的值）在指定的时间范围内确认master主服务器进入了主观下线状态， 则master主服务器会被标记为客观下线<font color="red">【定性】</font>
5. 在一般情况下， 每个sentinel进程会以每10秒一次的频率向集群中的所有master主服务器、slave从服务器发送`info`命令<font color="red">【查询下线】</font>
6. 当master主服务器被sentinel进程标记为客观下线时，sentinel进程向下线的master主服务器的所有slave从服务器发送`info`命令的频率会从10秒一次改为每秒一次【一个faiover要想被成功实行，sentinel必须能够向选为master的slave发送`SLAVEOF NO ONE`命令，然后能够通过INFO命令看到新master的配置信息】
7. 若没有足够数量的sentinel进程同意master已经下线， master主服务器的客观下线状态就会被移除。若master主服务器重新向sentinel进程发送 `ping`命令返回有效回复，master主服务器的主观下线状态就会被移除【变成slave？】

![15083002-31cc20da18483120](_v_images/20200106113257312_1308292119.png)
# 主观下线（sdown）和客观下线（odown）
```config
sentinel monitor <masterName> <ip> <port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 2
```
四个参数：<font color="red">多个哨兵是如何发现彼此的</font>
+ masterName这个是对某个master+slave组合的一个区分标识（一套sentinel是可以监听多套master+slave这样的组合的）【sentinel的配置是会动态变化的】
+ ip和port就是master节点的ip和端口号。
+ quorum这个参数是进行客观下线的一个依据，意思是至少有quorum个sentinel主观的认为这个master有故障，才会对这个master进行下线以及故障转移。因为有的时候，某个sentinel节点可能因为自身网络原因，导致无法连接master，而此时master并没有出现故障，所以这就需要多个sentinel都一致认为该master有问题，才可以进行下一步操作，这就保证了公平性和高可用。
```config
sentinel down-after-milliseconds <masterName> <timeout>
sentinel down-after-milliseconds mymaster 30000
```
这个配置其实就是进行主观下线的一个依据，masterName这个参数不用说了，timeout是一个毫秒值，表示：如果这台sentinel超过timeout这个时间都无法连通master包括slave（<font color="red">slave不需要客观下线，因为不需要故障转移</font>）的话，就会主观认为该master已经下线（实际下线需要客观下线的判断通过才会下线）

那么，多个sentinel之间是如何达到共识的呢？
这就是依赖于前面说的第二个定时任务，某个sentinel先将master节点进行一个主观下线，然后会将这个判定通过`sentinel is-master-down-by-addr`这个命令问对应的节点是否也同样认为该addr的master节点要做客观下线。

最后当达成这一共识的sentinel个数达到前面说的quorum设置的这个值时，就会对该master节点下线进行故障转移。
quorum的值一般设置为sentinel个数的二分之一加1，例如3个sentinel就设置2。
<font color="red">rac算法</font>
## 主观下线（Subjectively Down， 简称 SDOWN）：每个sentinel节点对Redis节点失败的“偏见”
+ 所谓主观下线，指的是单个sentinel实例对服务器做出的下线判断，即单个sentinel认为某个服务下线（有可能是接收不到订阅，网络不通等等原因）。
+ 主观下线就是说如果服务器在`down-after-milliseconds`给定的毫秒数之内， 没有返回sentinel发送的`ping`命令的回复， 或者返回一个错误， 那么 sentinel将这个服务器标记为主观下线。
+ sentinel会以每秒一次的频率向所有与其建立了命令连接的实例（master，从服务，其他sentinel）发`ping`命令，通过判断ping回复是有效回复，还是无效回复来判断实例时候在线（对该sentinel来说是“主观在线”）。
+ sentinel配置文件中的`down-after-milliseconds`设置了判断主观下线的时间长度，如果实例在`down-after-milliseconds`毫秒内，返回的都是无效回复，那么sentinel回认为该实例已（主观）下线，修改其flags状态为SRI_S_DOWN。如果多个sentinel监视一个服务，有可能存在多个sentinel的`down-after-milliseconds`配置不同，这个在实际生产中要注意。
## 客观下线（Objectively Down， 简称 ODOWN）：所有sentinel节点对Redis节点失败“达成共识”（超过quorum个统一）
+ 客观下线，指的是多个sentinel实例在对同一个服务器做出SDOWN判断， 并且通过`SENTINEL is-master-down-by-addr`命令互相交流之后， 得出的服务器下线判断，然后开启failover。
+ 客观下线就是说只有在足够数量的sentinel都将一个服务器标记为主观下线之后， 服务器才会被标记为客观下线（ODOWN）。
+ 只有当master被认定为客观下线时，**才会发生故障迁移**。
+ 当sentinel监视的某个服务主观下线后，sentinel会询问其它监视该服务的sentinel，看它们是否也认为该服务主观下线，接收到足够数量（这个值可以配置：quorum）的sentinel判断为主观下线，既任务该服务客观下线，并对其做故障转移操作。
+ sentinel通过发送`SENTINEL is-master-down-by-addr ip port current_epoch runid`，（ip：主观下线的服务id，port：主观下线的服务端口，current_epoch：sentinel的纪元，runid：*表示检测服务下线状态，如果是sentinel 运行id，表示用来选举领头sentinel）来询问其它sentinel是否同意服务下线。
+ 一个sentinel接收另一个sentinel发来的`is-master-down-by-addr`后，提取参数，根据ip和端口，检测该服务时候在该sentinel主观下线，并且回复`is-master-down-by-addr`，回复包含三个参数：down_state（1表示已下线，0表示未下线），leader_runid（领头sentinal id），leader_epoch（领头sentinel纪元）。
+ sentinel接收到回复后，根据配置设置的下线最小数量，达到这个值，既认为该服务客观下线。
+ 客观下线条件只适用于主服务器： 对于任何其他类型的redis 实例，sentinel 在将它们判断为下线前不需要进行协商， 所以从服务器或者其他sentinel永远不会达到客观下线条件。只要一个sentinel 发现某个主服务器进入了客观下线状态， 这个sentinel 就可能会被其他sentinel 推选出， 并对失效的主服务器执行自动故障迁移操作。
## 主观下线（SDOWN）和客观下线（ODOWN）的更多细节
sentinel对于不可用有两种不同的看法：
+ 主观不可用(SDOWN)
+ 客观不可用(ODOWN)

SDOWN是sentinel自己主观上检测到的关于master的状态，ODOWN需要一定数量的sentinel达成一致意见才能认为一个master客观上已经宕掉，各个sentinel之间通过命令`SENTINEL is_master_down_by_addr`来获得其它sentinel对master的检测结果。

从sentinel的角度来看，如果发送了ping心跳后，在一定时间内没有收到合法的回复，就达到了SDOWN的条件。这个时间在配置中通过`is-master-down-after-milliseconds`参数配置。
当sentinel发送PING后，以下回复之一都被认为是合法的：
```
1）PING replied with +PONG.
2）PING replied with -LOADING error.
3）PING replied with -MASTERDOWN error.
```
其它任何回复（或者根本没有回复）都是不合法的。

从SDOWN切换到ODOWN不需要任何一致性算法，只需要一个<font color="red">gossip协议</font>：如果一个sentinel收到了足够多的sentinel发来消息告诉它某个master已经down掉了，SDOWN状态就会变成ODOWN状态。如果之后master可用了，这个状态就会相应地被清理掉。
正如之前已经解释过了，真正进行failover需要一个授权的过程，但是所有的failover都开始于一个ODOWN状态。
<font color="red">ODOWN状态只适用于master，对于不是master的redis节点sentinel之间不需要任何协商，slaves和sentinel不会有ODOWN状态。</font>

<font color="red">如果slave与sentinel全部都战损了呢？？？</font>
# 配置版本号<font color="red">看不懂？？？</font>
**为什么要先获得大多数sentinel的认可时才能真正去执行failover呢？**
当一个sentinel被授权后，它将会获得宕掉的master的一份最新配置版本号，当failover执行结束以后，这个版本号将会被用于最新的配置。因为大多数sentinel都已经知道该版本号已经被要执行failover的sentinel拿走了，所以其他的sentinel都不能再去使用这个版本号。这意味着，每次failover都会附带有一个独一无二的版本号。我们将会看到这样做的重要性。而且，sentinel集群都遵守一个规则：如果sentinel A推荐sentinel B去执行failover，B会等待一段时间后，自行再次去对同一个master执行failover，这个等待的时间是通过failover-timeout配置项去配置的。从这个规则可以看出，sentinel集群中的sentinel不会再同一时刻并发去failover同一个master，第一个进行failover的sentinel如果失败了，另外一个将会在一定时间内进行重新进行failover，以此类推。
redis sentinel保证了活跃性：如果大多数sentinel能够互相通信，最终将会有一个被授权去进行failover.
redis sentinel也保证了安全性：每个试图去failover同一个master的sentinel都会得到一个独一无二的版本号。
# Sentinel之间和Slaves之间的自动发现机制
虽然sentinel集群中各个sentinel都互相连接彼此来检查对方的可用性以及互相发送消息。<font color="red">但是你不用在任何一个sentinel配置任何其它的sentinel的节点</font>。因为sentinel利用了master的发布/订阅机制去自动发现其它也监控了统一master的sentinel节点。
+ 通过向名为__sentinel__:hello的管道中发送消息来实现。
+ 同样，你也不需要在sentinel中配置某个master的所有slave的地址，sentinel会通过询问master来得到这些slave的地址的。
+ 每个sentinel通过向每个master和slave的【发布/订阅】频道__sentinel__:hello每秒发送一次消息，来宣布它的存在。
+ 每个sentinel也订阅了每个master和slave的频道__sentinel__:hello的内容，来发现未知的sentinel，当检测到了新的sentinel，则将其加入到自身维护的master监控列表中。
+ 每个sentinel发送的消息中也包含了其当前维护的最新的master配置。如果某个sentinel发现
+ 自己的配置版本低于接收到的配置版本，则会用新的配置更新自己的master配置。
+ 在为一个master添加一个新的sentinel前，sentinel总是检查是否已经有sentinel与新的sentinel的进程号或者是地址是一样的。如果是那样，这个sentinel将会被删除，而把新的sentinel添加上去。
# 配置传播
+ 一旦一个sentinel成功地对一个master进行了failover，它将会把关于master的最新配置通过广播形式（pub/sub）通知其它sentinel，其它的sentinel则更新对应master的配置
+ 一个faiover要想被成功实行，sentinel必须能够向选为master的slave发送`SLAVEOF NO ONE`命令，然后能够通过INFO命令看到新master的配置信息。
当将一个slave选举为master并发送`SLAVEOF NO ONE`后，即使其它的slave还没针对新master重新配置自己，failover也被认为是成功了的，然后所有sentinels将会发布新的配置信息。
+ 新配在集群中相互传播的方式，就是为什么我们需要当一个sentinel进行failover时必须被授权一个版本号的原因。
+ 每个sentinel使用【发布/订阅】的方式持续地传播master的配置版本信息，配置传播的【发布/订阅】管道是：`__sentinel__:hello`
+ 因为每一个配置都有一个版本号，所以以版本号最大的那个为标准。
## 举个例子
假设有一个名为mymaster的地址为192.168.10.202:6379。
一开始，集群中所有的sentinel都知道这个地址，于是为mymaster的配置打上版本号1。
一段时候后mymaster死了，有一个sentinel被授权用版本号2对其进行failover。
如果failover成功了，假设地址改为了192.168.10.202:9000，此时配置的版本号为2，进行failover的sentinel会将新配置广播给其他的sentinel，由于其他sentinel维护的版本号为1，发现新配置的版本号为2时，版本号变大了，说明配置更新了，于是就会采用最新的版本号为2的配置。
这意味着sentinel集群保证了第二种活跃性：一个能够互相通信的sentinel集群最终会采用版本号最高且相同的配置。


# sentinel选举
## sentinel的"仲裁会"【选举】？？？？？？？？？？？？？？
前面我们谈到，当一个master被sentinel集群监控时，需要为它指定一个参数，这个参数指定了当需要判决master为不可用，并且进行failover时，所需要的sentinel数量，可以称这个参数为票数（quorum）
```
sentinel monitor <masterName> <ip> <port> <quorum>
```
不过，当failover主备切换真正被触发后，failover并不会马上进行，还需要sentinel中的大多数sentinel授权后才可以进行failover。
当ODOWN时，failover被触发。
failover一旦被触发，尝试去进行failover的sentinel会去获得“大多数”sentinel的授权（如果票数比大多数还要大的时候，则询问更多的sentinel)
这个区别看起来很微妙，但是很容易理解和使用。例如，集群中有5个sentinel，票数被设置为2，当2个sentinel认为一个master已经不可用了以后，将会触发failover，但是，进行failover的那个sentinel必须先获得至少3个sentinel的授权才可以实行failover。
如果票数被设置为5，要达到ODOWN状态，必须所有5个sentinel都主观认为master为不可用，要进行failover，那么得获得所有5个sentinel的授权。


## 选举领头sentinel（即领导者选举）
一个redis服务被判断为客观下线时，多个监视该服务的sentinel协商，选举一个领头sentinel，对该redis服务进行故障转移操作。选举领头sentinel遵循以下规则：
1. 所有的sentinel都有公平被选举成领头的资格
2. 所有的sentinel都有且只有一次将某个sentinel选举成领头的机会（在一轮选举中），一旦选举某个sentinel为领头，不能更改
3. sentinel设置领头sentinel是先到先得，一旦当前sentinel设置了领头sentinel，以后要求设置sentinel为领头请求都会被拒绝
4. 每个发现服务客观下线的sentinel，都会要求其他sentinel将自己设置成领头
5. 当一个sentinel（源sentinel）向另一个sentinel（目sentinel）发送is-master-down-by-addr ip port current_epoch runid命令的时候，runid参数不是*，而是sentinel运行id，就表示源sentinel要求目标sentinel选举其为领头
6. 源sentinel会检查目标sentinel对其要求设置成领头的回复，如果回复的leader_runid和leader_epoch为源sentinel，表示目标sentinel同意将源sentinel设置成领头
7. 如果某个sentinel被半数以上的sentinel设置成领头，那么该sentinel既为领头
8. 如果在限定时间内，没有选举出领头sentinel，暂定一段时间，再选举【要考虑sentinel个数保障选举成功rac算法】
## 为什么要选领导者
简单来说，就是因为只能有一个sentinel节点去完成故障转移。
sentinel is-master-down-by-addr这个命令有两个作用，一是确认下线判定，二是进行领导者选举。
选举过程：
1. 每个做主观下线的sentinel节点向其他sentinel节点发送上面那条命令，要求将它设置为领导者。
2. 收到命令的sentinel节点如果还没有同意过其他的sentinel发送的命令（还未投过票），那么就会同意，否则拒绝。
3. 如果该sentinel节点发现自己的票数已经过半且达到了quorum的值，就会成为领导者
4. 如果这个过程出现多个sentinel成为领导者，则会等待一段时间重新选举。
## 领导者选举
原因：只有一个sentinel节点完成故障转移
选举：通过sentinel is-master-down-by-addr命令都希望成为领导者
1. 每个做主观下线的sentinel节点向其他sentinel节点发送命令，要求将它设置为领导者
2. 收到命令的sentinel节点如果没有同意通过其他sentinel节点发送的命令，那么将同意该请求，否则拒绝
3. 如果该sentinel节点发现自己的票数已经超过sentinel集合半数且超过quorum，那么它将称为领导者
4. 如果此过程有多个sentinel节点成为了领导者，那么将等待一段时间重新进行选举
<font color="red">rac算法</font>


# sentinel集群
## sentinel集群注意事项
1. 只有sentinel集群中大多数服务器认定master主观下线时master才会被认定为客观下线，才可以进行故障迁移，也就是说，即使不管我们在sentinel monitor中设置的数是多少，就算是满足了该值，只要达不到大多数，就不会发生故障迁移。
2. 官方建议sentinel至少部署三台，且分布在不同机器。这里主要考虑到sentinel的可用性，假如我们只部署了两台sentinel，且quorum设置为1，也可以实现自动故障迁移，但假如其中一台sentinel挂了，就永远不会触发自动故障迁移，因为永远达不到大多数sentinel认定master主观下线了。
3. sentinel monitor配置中的master IP尽量不要写127.0.0.1或localhost，因为客户端，如jedis获取master是根据这个获取的，若这样配置，jedis获取的ip则是127.0.0.1，这样就可能导致程序连接不上master
4. 当sentinel 启动后会自动的修改sentinel.conf文件，如已发现的master的slave信息，和集群中其它sentinel 的信息等,这样即使重启sentinel也能保持原来的状态。注意，当集群服务器调整时，如更换sentinel的机器，或者新配置一个sentinel，请不要直接复制原来运行过得sentinel配置文件，因为其里面自动生成了以上说的那些信息，我们应该复制一个新的配置文件或者把自动生成的信息给删掉。
5. 当发生故障迁移的时候，master的变更记录与slave更换master的修改会自动同步到redis的配置文件，这样即使重启redis也能保持变更后的状态。



# 故障转移（sentinel领导者节点完成）
1. 从slave节点中选出一个“合适的”节点作为新的master节点
2. 对上面的slave节点执行slaveof no one命令让其成为master节点
3. 向剩余的slave节点发送命令，让它们成为新master节点的slave节点，复制规则和parallel-syncs参数有关
4. 更新对原来master节点配置为slave，并保持对其“关注”，当其恢复后命令它去复制新的master节点

# 选择“合适的”slave节点
1. 选择slave-priority（slave节点优先级）最高的slave节点，如果存在则返回，不存在则继续
2. 选择复制偏移量最大的slave节点（复制的最完整），如果存在则返回，不存在则继续
3. 选择runId最小的slave节点





# Redis Sentinel开发运维常见问题
1. 节点运维
2. 高可用读写分离

## 节点运维
+ 主节点
+ 从节点
+ sentinel节点

机器下线：例如过保等情况
机器性能不足：例如cpu、内存、硬盘、网络
节点自身故障：例如服务不稳定等

主节点故障转移命令（手动故障转移：忽略客观、主观那些）
```
sentinel failover <masterName>
```
![](_v_images/20200105123512157_837528507.png)
## 节点运维
从节点：临时下线还是永久下线，例如是否做一些清理工作。但是要考虑读写分离的情况。
sentinel节点：同上
## 节点上线
主节点：sentinel failover进行替换
从节点：slaveof即可，sentinel节点可以感知
sentinel节点：参考其它sentinel节点启动即可
## 高可用读写分离
![](_v_images/20200105125155703_1050341019.png)





# sentinel和redis身份验证
当一个master配置为需要密码才能连接时，客户端和slave在连接时都需要提供密码。
master通过requirepass设置自身的密码，不提供密码无法连接到这个master。
slave通过masterauth来设置访问master时的密码。
但是当使用了sentinel时，由于一个master可能会变成一个slave，一个slave也可能会变成master，所以需要同时设置上述两个配置项。


# Sentinel命令（即登录到sentinel节点后执行的命令，比如执行"redis-cli -h sentinel_ip -p26379"命令后，才可以执行下面命令）
PING ：返回 PONG 。
SENTINEL masters ：列出所有被监视的主服务器，以及这些主服务器的当前状态；
SENTINEL slaves <master name> ：列出给定主服务器的所有从服务器，以及这些从服务器的当前状态；
SENTINEL get-master-addr-by-name <master name> ： 返回给定名字的主服务器的 IP 地址和端口号。 如果这个主服务器正在执行故障转移操作， 或者针对这个主服务器的故障转移操作已经完成， 那么这个命令返回新的主服务器的 IP 地址和端口号；
SENTINEL reset <pattern> ： 重置所有名字和给定模式 pattern 相匹配的主服务器。 pattern 参数是一个 Glob 风格的模式。 重置操作清楚主服务器目前的所有状态， 包括正在执行中的故障转移， 并移除目前已经发现和关联的， 主服务器的所有从服务器和 Sentinel ；
SENTINEL failover <master name> ： 当主服务器失效时， 在不询问其他 Sentinel 意见的情况下， 强制开始一次自动故障迁移。 （不过发起故障转移的 Sentinel 会向其他 Sentinel 发送一个新的配置，其他 Sentinel 会根据这个配置进行相应的更新）

SENTINEL MONITOR <name> <ip> <port> <quorum> 这个命令告诉sentinel去监听一个新的master
SENTINEL REMOVE <name> 命令sentinel放弃对某个master的监听
SENTINEL SET <name> <option> <value> 这个命令很像Redis的CONFIG SET命令，用来改变指定master的配置。支持多个<option><value>。例如以下实例：SENTINEL SET objects-cache-master down-after-milliseconds 1000
只要是配置文件中存在的配置项，都可以用SENTINEL SET命令来设置。这个还可以用来设置master的属性，比如说quorum(票数)，而不需要先删除master，再重新添加master。例如：SENTINEL SET objects-cache-master quorum 5

客户端可以通过SENTINEL get-master-addr-by-name <master name>获取当前的主服务器IP地址和端口号，以及SENTINEL slaves <master name>获取所有的Slaves信息。

# 增加或删除Sentinel
由于有sentinel自动发现机制，所以添加一个sentinel到你的集群中非常容易，你所需要做的只是监控到某个Master上，然后新添加的sentinel就能获得其他sentinel的信息以及master所有的slaves。
如果你需要添加多个sentinel，建议你一个接着一个添加，这样可以预防网络隔离带来的问题。你可以每个30秒添加一个sentinel。最后你可以用SENTINEL MASTER mastername来检查一下是否所有的sentinel都已经监控到了master。
删除一个sentinel显得有点复杂：因为sentinel永远不会删除一个已经存在过的sentinel，即使它已经与组织失去联系很久了。
要想删除一个sentinel，应该遵循如下步骤：
1）停止所要删除的sentinel
2）发送一个SENTINEL RESET * 命令给所有其它的sentinel实例，如果你想要重置指定master上面的sentinel，只需要把*号改为特定的名字，注意，需要一个接一个发，每次发送的间隔不低于30秒。
3）检查一下所有的sentinels是否都有一致的当前sentinel数。使用SENTINEL MASTER mastername 来查询。

# 删除旧master或者不可达slave
sentinel永远会记录好一个Master的slaves，即使slave已经与组织失联好久了。这是很有用的，因为sentinel集群必须有能力把一个恢复可用的slave进行重新配置。
并且，failover后，失效的master将会被标记为新master的一个slave，这样的话，当它变得可用时，就会从新master上复制数据。
然后，有时候你想要永久地删除掉一个slave(有可能它曾经是个master)，你只需要发送一个`SENTINEL RESET master`命令给所有的sentinels，它们将会更新列表里能够正确地复制master数据的slave。

# 发布与订阅信息（sentinel的日志文件里可以看到这些信息）
客户端可以将 Sentinel 看作是一个只提供了订阅功能的 Redis 服务器： 你不可以使用 PUBLISH 命令向这个服务器发送信息， 但你可以用 SUBSCRIBE 命令或者 PSUBSCRIBE 命令， 通过订阅给定的频道来获取相应的事件提醒。

一个频道能够接收和这个频道的名字相同的事件。 比如说， 名为 +sdown 的频道就可以接收所有实例进入主观下线（SDOWN）状态的事件。

通过执行 "PSUBSCRIBE * "命令可以接收所有事件信息（即订阅所有消息）。

以下列出的是客户端可以通过订阅来获得的频道和信息的格式： 第一个英文单词是频道/事件的名字， 其余的是数据的格式。

注意， 当格式中包含 instance details 字样时， 表示频道所返回的信息中包含了以下用于识别目标实例的内容.

以下是所有可以收到的消息的消息格式，如果你订阅了所有消息的话。第一个单词是频道的名字，其它是数据的格式。
注意：以下的instance details的格式是：
<instance-type> <name> <ip> <port> @ <master-name> <master-ip> <master-port>

```
+reset-master <instance details> -- 当master被重置时.
    +slave <instance details> -- 当检测到一个slave并添加进slave列表时.
    +failover-state-reconf-slaves <instance details> -- Failover状态变为reconf-slaves状态时
    +failover-detected <instance details> -- 当failover发生时
    +slave-reconf-sent <instance details> -- sentinel发送SLAVEOF命令把它重新配置时
    +slave-reconf-inprog <instance details> -- slave被重新配置为另外一个master的slave，但数据复制还未发生时。
    +slave-reconf-done <instance details> -- slave被重新配置为另外一个master的slave并且数据复制已经与master同步时。
    -dup-sentinel <instance details> -- 删除指定master上的冗余sentinel时 (当一个sentinel重新启动时，可能会发生这个事件).
    +sentinel <instance details> -- 当master增加了一个sentinel时。
    +sdown <instance details> -- 进入SDOWN状态时;
    -sdown <instance details> -- 离开SDOWN状态时。
    +odown <instance details> -- 进入ODOWN状态时。
    -odown <instance details> -- 离开ODOWN状态时。
    +new-epoch <instance details> -- 当前配置版本被更新时。
    +try-failover <instance details> -- 达到failover条件，正等待其他sentinel的选举。
    +elected-leader <instance details> -- 被选举为去执行failover的时候。
    +failover-state-select-slave <instance details> -- 开始要选择一个slave当选新master时。
    no-good-slave <instance details> -- 没有合适的slave来担当新master
    selected-slave <instance details> -- 找到了一个适合的slave来担当新master
    failover-state-send-slaveof-noone <instance details> -- 当把选择为新master的slave的身份进行切换的时候。
    failover-end-for-timeout <instance details> -- failover由于超时而失败时。
    failover-end <instance details> -- failover成功完成时。
    switch-master <master name> <oldip> <oldport> <newip> <newport> -- 当master的地址发生变化时。通常这是客户端最感兴趣的消息了。
    +tilt -- 进入Tilt模式。
    -tilt -- 退出Tilt模式。
```

可以看出，使用Sentinel命令和发布订阅两种机制就能很好的实现和客户端的集成整合：
使用get-master-addr-by-name和slaves指令可以获取当前的Master和Slaves的地址和信息；而当发生故障转移时，即Master发生切换，可以通过订阅的+switch-master事件获得最新的Master信息。

sentinel.conf中的notification-script
在sentinel.conf中可以配置多个sentinel notification-script <master name> <shell script-path>, 如sentinel notification-script mymaster ./check.sh
这个是在群集failover时会触发执行指定的脚本。脚本的执行结果若为1，即稍后重试（最大重试次数为10）；若为2，则执行结束。并且脚本最大执行时间为60秒，超时会被终止执行。

目前会存在该脚本被执行多次的问题，网上查找资料获得的解释是：脚本分为两个级别， SENTINEL_LEADER 和 SENTINEL_OBSERVER ，前者仅由领头 Sentinel 执行（一个 Sentinel），而后者由监视同一个 master 的所有 Sentinel 执行（多个 Sentinel）。

无failover时的配置纠正
即使当前没有failover正在进行，sentinel依然会使用当前配置去设置监控的master。特别是：
1）根据最新配置确认为slaves的节点却声称自己是master(上文例子中被网络隔离后的的redis3)，这时它们会被重新配置为当前master的slave。
2）如果slaves连接了一个错误的master，将会被改正过来，连接到正确的master。

Slave选举与优先级
当一个sentinel准备好了要进行failover，并且收到了其他sentinel的授权，那么就需要选举出一个合适的slave来做为新的master。

slave的选举主要会评估slave的以下几个方面：
1）与master断开连接的次数
2）Slave的优先级
3）数据复制的下标(用来评估slave当前拥有多少master的数据)
4）进程ID

如果一个slave与master失去联系超过10次，并且每次都超过了配置的最大失联时间(down-after-milliseconds)，如果sentinel在进行failover时发现slave失联，那么这个slave就会被sentinel认为不适合用来做新master的。
更严格的定义是，如果一个slave持续断开连接的时间超过
(down-after-milliseconds * 10) + milliseconds_since_master_is_in_SDOWN_state
就会被认为失去选举资格。

符合上述条件的slave才会被列入master候选人列表，并根据以下顺序来进行排序：
1）sentinel首先会根据slaves的优先级来进行排序，优先级越小排名越靠前。
2）如果优先级相同，则查看复制的下标，哪个从master接收的复制数据多，哪个就靠前。
3）如果优先级和下标都相同，就选择进程ID较小的那个。

一个redis无论是master还是slave，都必须在配置中指定一个slave优先级。要注意到master也是有可能通过failover变成slave的。
如果一个redis的slave优先级配置为0，那么它将永远不会被选为master。但是它依然会从master哪里复制数据。

# 故障转移
所谓故障转移就是当master宕机，选一个合适的slave来晋升为master的操作，redis-sentinel会自动完成这个，不需要我们手动来实现。

一次故障转移操作大致分为以下流程：
发现主服务器已经进入客观下线状态。
对我们的当前集群进行自增， 并尝试在这个集群中当选。
如果当选失败， 那么在设定的故障迁移超时时间的两倍之后， 重新尝试当选。 如果当选成功， 那么执行以下步骤：
选出一个从服务器，并将它升级为主服务器。
向被选中的从服务器发送 SLAVEOF NO ONE 命令，让它转变为主服务器。
通过发布与订阅功能， 将更新后的配置传播给所有其他 Sentinel ， 其他 Sentinel 对它们自己的配置进行更新。
向已下线主服务器的从服务器发送 SLAVEOF 命令， 让它们去复制新的主服务器。
当所有从服务器都已经开始复制新的主服务器时， 领头 Sentinel 终止这次故障迁移操作。
每当一个 Redis 实例被重新配置（reconfigured） —— 无论是被设置成主服务器、从服务器、又或者被设置成其他主服务器的从服务器 —— Sentinel 都会向被重新配置的实例发送一个 CONFIG REWRITE 命令， 从而确保这些配置会持久化在硬盘里。

Sentinel 使用以下规则来选择新的主服务器：
在失效主服务器属下的从服务器当中， 那些被标记为主观下线、已断线、或者最后一次回复 PING 命令的时间大于五秒钟的从服务器都会被淘汰。
在失效主服务器属下的从服务器当中， 那些与失效主服务器连接断开的时长超过 down-after 选项指定的时长十倍的从服务器都会被淘汰。
在经历了以上两轮淘汰之后剩下来的从服务器中， 我们选出复制偏移量（replication offset）最大的那个从服务器作为新的主服务器； 如果复制偏移量不可用， 或者从服务器的复制偏移量相同， 那么带有最小运行 ID 的那个从服务器成为新的主服务器。

Sentinel 自动故障迁移的一致性特质
Sentinel 自动故障迁移使用 Raft 算法来选举领头（leader） Sentinel ， 从而确保在一个给定的纪元（epoch）里， 只有一个领头产生。

这表示在同一个纪元中， 不会有两个 Sentinel 同时被选中为领头， 并且各个 Sentinel 在同一个纪元中只会对一个领头进行投票。

更高的配置纪元总是优于较低的纪元， 因此每个 Sentinel 都会主动使用更新的纪元来代替自己的配置。

简单来说， 可以将 Sentinel 配置看作是一个带有版本号的状态。 一个状态会以最后写入者胜出（last-write-wins）的方式（也即是，最新的配置总是胜出）传播至所有其他 Sentinel 。

举个例子， 当出现网络分割（network partitions）时， 一个 Sentinel 可能会包含了较旧的配置， 而当这个 Sentinel 接到其他 Sentinel 发来的版本更新的配置时， Sentinel 就会对自己的配置进行更新。

如果要在网络分割出现的情况下仍然保持一致性， 那么应该使用 min-slaves-to-write 选项， 让主服务器在连接的从实例少于给定数量时停止执行写操作， 与此同时， 应该在每个运行 Redis 主服务器或从服务器的机器上运行 Redis Sentinel 进程。

Sentinel 状态的持久化
Sentinel 的状态会被持久化在 Sentinel 配置文件里面。每当 Sentinel 接收到一个新的配置， 或者当领头 Sentinel 为主服务器创建一个新的配置时， 这个配置会与配置纪元一起被保存到磁盘里面。这意味着停止和重启 Sentinel 进程都是安全的。

Sentinel 在非故障迁移的情况下对实例进行重新配置
即使没有自动故障迁移操作在进行， Sentinel 总会尝试将当前的配置设置到被监视的实例上面。 特别是：

根据当前的配置， 如果一个从服务器被宣告为主服务器， 那么它会代替原有的主服务器， 成为新的主服务器， 并且成为原有主服务器的所有从服务器的复制对象。
那些连接了错误主服务器的从服务器会被重新配置， 使得这些从服务器会去复制正确的主服务器。
不过， 在以上这些条件满足之后， Sentinel 在对实例进行重新配置之前仍然会等待一段足够长的时间， 确保可以接收到其他 Sentinel 发来的配置更新， 从而避免自身因为保存了过期的配置而对实例进行了不必要的重新配置。

总结来说，故障转移分为三个步骤：

1）从下线的主服务的所有从服务里面挑选一个从服务，将其转成主服务
sentinel状态数据结构中保存了主服务的所有从服务信息，领头sentinel按照如下的规则从从服务列表中挑选出新的主服务；
删除列表中处于下线状态的从服务；
删除最近5秒没有回复过领头sentinel info信息的从服务；
删除与已下线的主服务断开连接时间超过 down-after-milliseconds*10毫秒的从服务，这样就能保留从的数据比较新（没有过早的与主断开连接）；
领头sentinel从剩下的从列表中选择优先级高的，如果优先级一样，选择偏移量最大的（偏移量大说明复制的数据比较新），如果偏移量一样，选择运行id最小的从服务。

2）已下线主服务的所有从服务改为复制新的主服务
挑选出新的主服务之后，领头sentinel 向原主服务的从服务发送 slaveof 新主服务 的命令，复制新master。

3）将已下线的主服务设置成新的主服务的从服务，当其回复正常时，复制新的主服务，变成新的主服务的从服务
同理，当已下线的服务重新上线时，sentinel会向其发送slaveof命令，让其成为新主的从。

温馨提示：还可以向任意sentinel发生sentinel failover <masterName> 进行手动故障转移，这样就不需要经过上述主客观和选举的过程。

# sentinel.conf文件配置参数解释
```
1）sentinel monitor mymaster 192.168.10.202 6379 2
Sentine监听的maste地址，第一个参数是给master起的名字，第二个参数为master IP，第三个为master端口，第四个为当该master挂了的时候，若想将该master判为失效，
在Sentine集群中必须至少2个Sentine同意才行，只要该数量不达标，则就不会发生故障迁移。也就是说只要有2个sentinel认为master下线，就认为该master客观下线，
启动failover并选举产生新的master。通常最后一个参数不能多于启动的sentinel实例数。
 
这个配置是sentinel需要监控的master/slaver信息，格式为sentinel monitor <mastername> <masterIP> <masterPort> <quorum> 
其中<quorum>应该小于集群中slave的个数，当失效的节点数超过了<quorum>,则认为整个体系结构失效
 
不过要注意， 无论你设置要多少个 Sentinel 同意才能判断一个服务器失效， 一个 Sentinel 都需要获得系统中多数（majority） Sentinel 的支持， 才能发起一次自动故障迁移，
并预留一个给定的配置纪元 （configuration Epoch ，一个配置纪元就是一个新主服务器配置的版本号）。
  
换句话说， 在只有少数（minority） Sentinel 进程正常运作的情况下， Sentinel 是不能执行自动故障迁移的。
-----------------------------------------------------------------------------------------------
2）sentinel down-after-milliseconds mymaster 30000
表示master被当前sentinel实例认定为失效的间隔时间。
master在多长时间内一直没有给Sentine返回有效信息，则认定该master主观下线。也就是说如果多久没联系上redis-servevr，认为这个redis-server进入到失效（SDOWN）状态。
  
如果服务器在给定的毫秒数之内， 没有返回 Sentinel 发送的 PING 命令的回复， 或者返回一个错误， 那么 Sentinel 将这个服务器标记为主观下线（subjectively down，简称 SDOWN ）。
不过只有一个 Sentinel 将服务器标记为主观下线并不一定会引起服务器的自动故障迁移： 只有在足够数量的 Sentinel 都将一个服务器标记为主观下线之后， 服务器才会被标记为客观下线
（objectively down， 简称 ODOWN ）， 这时自动故障迁移才会执行。
将服务器标记为客观下线所需的 Sentinel 数量由对主服务器的配置决定。
-----------------------------------------------------------------------------------------------
3）sentinel parallel-syncs mymaster 2
当在执行故障转移时，设置几个slave同时进行切换master，该值越大，则可能就有越多的slave在切换master时不可用，可以将该值设置为1，即一个一个来，这样在某个
slave进行切换master同步数据时，其余的slave还能正常工作，以此保证每次只有一个从服务器处于不能处理命令请求的状态。
  
parallel-syncs 选项指定了在执行故障转移时， 最多可以有多少个从服务器同时对新的主服务器进行同步， 这个数字越小， 完成故障转移所需的时间就越长。
  
如果从服务器被设置为允许使用过期数据集（参见对 redis.conf 文件中对 slave-serve-stale-data 选项的说明）， 那么你可能不希望所有从服务器都在同一时间向新的主服务器发送同步请求，
因为尽管复制过程的绝大部分步骤都不会阻塞从服务器， 但从服务器在载入主服务器发来的 RDB 文件时， 仍然会造成从服务器在一段时间内不能处理命令请求： 如果全部从服务器一起对新的主
服务器进行同步， 那么就可能会造成所有从服务器在短时间内全部不可用的情况出现。
 
当新master产生时，同时进行"slaveof"到新master并进行"SYNC"的slave个数。 
默认为1,建议保持默认值 
在salve执行salveof与同步时，将会终止客户端请求。 
此值较大，意味着"集群"终止客户端请求的时间总和和较大。 
此值较小,意味着"集群"在故障转移期间，多个salve向客户端提供服务时仍然使用旧数据。 
-----------------------------------------------------------------------------------------------
4）sentinel can-failover mymaster yes
在sentinel检测到O_DOWN后，是否对这台redis启动failover机制
-----------------------------------------------------------------------------------------------
5）sentinel auth-pass mymaster 20180408
设置sentinel连接的master和slave的密码，这个需要和redis.conf文件中设置的密码一样
-----------------------------------------------------------------------------------------------
6）sentinel failover-timeout mymaster 180000
failover过期时间，当failover开始后，在此时间内仍然没有触发任何failover操作，当前sentinel将会认为此次failoer失败。 
执行故障迁移超时时间，即在指定时间内没有大多数的sentinel 反馈master下线，该故障迁移计划则失效
-----------------------------------------------------------------------------------------------
7）sentinel config-epoch mymaster 0
选项指定了在执行故障转移时， 最多可以有多少个从服务器同时对新的主服务器进行同步。这个数字越小， 完成故障转移所需的时间就越长。
-----------------------------------------------------------------------------------------------
8）sentinel notification-script mymaster /var/redis/notify.sh
当failover时，可以指定一个"通知"脚本用来告知当前集群的情况。
脚本被允许执行的最大时间为60秒，如果超时，脚本将会被终止(KILL)
-----------------------------------------------------------------------------------------------
9）sentinel leader-epoch mymaster 0
同时一时间最多0个slave可同时更新配置,建议数字不要太大,以免影响正常对外提供服务。
```

# 基于以上细节知识梳理，总结出sentinel的工作原理
```
首先要能理解SDOWN和ODOWN这两个词的含义，上面已经详细介绍了它们俩。在此再提一下：
SDOWN:subjectively down,直接翻译的为"主观"失效,即当前sentinel实例认为某个redis服务为"不可用"状态.
ODOWN:objectively down,直接翻译为"客观"失效,即多个sentinel实例都认为master处于"SDOWN"状态,那么此时master将处于ODOWN,ODOWN可以简单理解为master已经被集群确定
为"不可用",将会开启failover.
 
SDOWN适合于master和slave,但是ODOWN只会使用于master;当slave失效超过"down-after-milliseconds"后,那么所有sentinel实例都会将其标记为"SDOWN"。
 
1) SDOWN与ODOWN转换过程:
每个sentinel实例在启动后,都会和已知的slaves/master以及其他sentinels建立TCP连接,并周期性发送PING(默认为1秒)
在交互中,如果redis-server无法在"down-after-milliseconds"时间内响应或者响应错误信息,都会被认为此redis-server处于SDOWN状态。
如果SDOWN的server为master,那么此时sentinel实例将会向其他sentinel间歇性(一秒)发送"is-master-down-by-addr <ip> <port>"指令并获取响应信息,如果足够多的
sentinel实例检测到master处于SDOWN,那么此时当前sentinel实例标记master为ODOWN...其他sentinel实例做同样的交互操作。
配置项"sentinel monitor <mastername> <masterip> <masterport> <quorum>",如果检测到master处于SDOWN状态的slave个数达到<quorum>,那么此时此sentinel实例将会认为
master处于ODOWN。每个sentinel实例将会间歇性(10秒)向master和slaves发送"INFO"指令,如果master失效且没有新master选出时,每1秒发送一次"INFO";"INFO"的主要目的就是
获取并确认当前集群环境中slaves和master的存活情况。
 
经过上述过程后,所有的sentinel对master失效达成一致后,开始failover.
 
2) Sentinel与slaves"自动发现"机制:
在sentinel的配置文件中(local-sentinel.conf),都指定了port,此port就是sentinel实例侦听其他sentinel实例建立链接的端口.在集群稳定后,最终会每个sentinel实例之间都
会建立一个tcp链接,此链接中发送"PING"以及类似于"is-master-down-by-addr"指令集,可用用来检测其他sentinel实例的有效性以及"ODOWN"和"failover"过程中信息的交互.
 
在sentinel之间建立连接之前,sentinel将会尽力和配置文件中指定的master建立连接.sentinel与master的连接中的通信主要是基于pub/sub来发布和接收信息,发布的信息内容包
括当前sentinel实例的侦听端口:
+sentinel sentinel 127.0.0.1:26579 127.0.0.1 26579 .... 
 
发布的主题名称为"__sentinel__:hello";同时sentinel实例也是"订阅"此主题,以获得其他sentinel实例的信息.由此可见,环境首次构建时,在默认master存活的情况下,所有的
sentinel实例可以通过pub/sub即可获得所有的sentinel信息,此后每个sentinel实例即可以根据+sentinel信息中的"ip+port"和其他sentinel逐个建立tcp连接即可.不过需要提醒
的是,每个sentinel实例均会间歇性(5秒)向"__sentinel__:hello"主题中发布自己的ip+port,目的就是让后续加入集群的sentinel实例也能或得到自己的信息。
根据上文,我们知道在master有效的情况下,即可通过"INFO"指令获得当前master中已有的slave列表;此后任何slave加入集群,master都会向"主题中"发布"+slave 127.0.0.1:6579 ..",
那么所有的sentinel也将立即获得slave信息,并和slave建立链接并通过PING检测其存活性.
 
补充一下,每个sentinel实例都会保存其他sentinel实例的列表以及现存的master/slaves列表,各自的列表中不会有重复的信息(不可能出现多个tcp连接),对于sentinel将使用ip+port
做唯一性标记,
对于master/slaver将使用runid做唯一性标记,其中redis-server的runid在每次启动时都不同.
 
3) Leader选举:
其实在sentinels故障转移中，仍然需要一个"Leader"来调度整个过程：master的选举以及slave的重配置和同步。当集群中有多个sentinel实例时，如何选举其中一个sentinel为leader呢？
 
在配置文件中"can-failover""quorum"参数，以及"is-master-down-by-addr"指令配合来完成整个过程。
A) "can-failover"用来表明当前sentinel是否可以参与"failover"过程，如果为"YES"则表明它将有能力参与"Leader"的选举，否则它将作为"Observer"，observer参与leader选举投票但
不能被选举；
B) "quorum"不仅用来控制master ODOWN状态确认，同时还用来选举leader时最小"赞同票"数；
C) "is-master-down-by-addr"，它可以用来检测"ip + port"的master是否已经处于SDOWN状态，不过此指令不仅能够获得master是否处于SDOWN，同时它还额外的返回当前sentinel
本地"投票选举"的Leader信息(runid);
 
每个sentinel实例都持有其他的sentinels信息，在Leader选举过程中(当为leader的sentinel实例失效时，有可能master server并没失效，注意分开理解)，sentinel实例将从所有的
sentinels集合中去除"can-failover = no"和状态为SDOWN的sentinels，在剩余的sentinels列表中按照runid按照"字典"顺序排序后，取出runid最小的sentinel实例，并将它"投票选举"
为Leader，并在其他sentinel发送的"is-master-down-by-addr"指令时将推选的runid追加到响应中。每个sentinel实例都会检测"is-master-down-by-addr"的响应结果，如果"投票选举"的
leader为自己，且状态正常的sentinels实例中，"赞同者"的自己的sentinel个数不小于(>=) 50% + 1,且不小与<quorum>，那么此sentinel就会认为选举成功且leader为自己。
在sentinel.conf文件中，我们期望有足够多的sentinel实例配置"can-failover yes"，这样能够确保当leader失效时，能够选举某个sentinel为leader，以便进行failover。如果leader无法产生，
比如较少的sentinels实例有效，那么failover过程将无法继续.
 
4) failover过程:
在Leader触发failover之前，首先wait数秒(随即0~5)，以便让其他sentinel实例准备和调整(有可能多个leader??),如果一切正常，那么leader就需要开始将一个salve提升为master，此slave
必须为状态良好(不能处于SDOWN/ODOWN状态)且权重值最低(redis.conf中)的，当master身份被确认后，开始failover
A）"+failover-triggered": Leader开始进行failover，此后紧跟着"+failover-state-wait-start"，wait数秒。
B）"+failover-state-select-slave": Leader开始查找合适的slave
C）"+selected-slave": 已经找到合适的slave
D） "+failover-state-sen-slaveof-noone": Leader向slave发送"slaveof no one"指令，此时slave已经完成角色转换，此slave即为master
E） "+failover-state-wait-promotition": 等待其他sentinel确认slave
F）"+promoted-slave"：确认成功
G）"+failover-state-reconf-slaves": 开始对slaves进行reconfig操作。
H）"+slave-reconf-sent":向指定的slave发送"slaveof"指令，告知此slave跟随新的master
I）"+slave-reconf-inprog": 此slave正在执行slaveof + SYNC过程，如过slave收到"+slave-reconf-sent"之后将会执行slaveof操作。
J）"+slave-reconf-done": 此slave同步完成，此后leader可以继续下一个slave的reconfig操作。循环G）
K）"+failover-end": 故障转移结束
L）"+switch-master"：故障转移成功后，各个sentinel实例开始监控新的master。

```

# redis sentinel 主从切换(failover)的容灾环境部署记录
redis主从复制简单来说：
A）Redis的复制功能是支持多个数据库之间的数据同步。一类是主数据库（master）一类是从数据库（slave），主数据库可以进行读写操作，当发生写操作的时候自动将数据同步到从数据库，而从数据库一般是只读的，并接收主数据库同步过来的数据，一个主数据库可以有多个从数据库，而一个从数据库只能有一个主数据库。
B）通过redis的复制功能可以很好的实现数据库的读写分离，提高服务器的负载能力。主数据库主要进行写操作，而从数据库负责读操作。

Redis主从复制流程简图
![907596-20180508170240323-1601239355](_v_images/20200106144202300_930634649.png)
redis主从复制的大致过程：
1）当一个从数据库启动时，会向主数据库发送sync命令，
2）主数据库接收到sync命令后会开始在后台保存快照（执行rdb操作），并将保存期间接收到的命令缓存起来
3）当快照完成后，redis会将快照文件和所有缓存的命令发送给从数据库。
4）从数据库收到后，会载入快照文件并执行收到的缓存的命令。

注意：redis2.8之前的版本：当主从数据库同步的时候从数据库因为网络原因断开重连后会重新执行上述操作，不支持断点续传。redis2.8之后支持断点续传。

0）Redis主从结构支持一主多从+n个sentinel模式，信息如下：
```
192.168.10.202   redis-master    redis（6379）、sentinel（26379）
192.168.10.203   redis-slave01   redis（6379）、sentinel（26379）
192.168.10.205   redis-slave02   redis（6379）、sentinel（26379） 
  
关闭三个节点机器的iptables和selinux（所有节点机器上都要操作）
[root@redis-master ~]# /etc/init.d/iptables stop
[root@redis-master ~]# vim /etc/sysconfig/selinux
......
SELINUX=disabled
[root@redis-master ~]# setenforce 0
[root@redis-master ~]# getenforce
Permissive
  
注意：本案例采用1主2从+3 sentinel的集群模式，所有从节点的配置都一样。
```
a）redis服务器上各自存在一个Sentinel，监控本机redis的运行情况，并通知给闭路环上其它的redis节点；
b）当master发生异常（例如：宕机和断电等）导致不可运行时，Sentinel将通知给其它节点，而剩余节点上的Sentinel将重新选举出新的master，而原来的master重新恢复正常后，则一直扮演slave角色；
c）规定整个架构体系中，master提供读写服务，而slave只提供读取服务。






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