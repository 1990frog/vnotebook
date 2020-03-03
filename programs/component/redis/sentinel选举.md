# Sentinel选举
## Sentinel的"仲裁会"【选举】？？？？？？？？？？？？？？
前面我们谈到，当一个Master被Sentinel集群监控时，需要为它指定一个参数，这个参数指定了当需要判决Master为不可用，并且进行failover时，所需要的Sentinel数量，可以称这个参数为票数（quorum）
```
Sentinel monitor <MasterName> <ip> <port> <quorum>
```
不过，当failover主备切换真正被触发后，failover并不会马上进行，还需要Sentinel中的大多数Sentinel授权后才可以进行failover。
当ODOWN时，failover被触发。
failover一旦被触发，尝试去进行failover的Sentinel会去获得“大多数”Sentinel的授权（如果票数比大多数还要大的时候，则询问更多的Sentinel)
这个区别看起来很微妙，但是很容易理解和使用。例如，集群中有5个Sentinel，票数被设置为2，当2个Sentinel认为一个Master已经不可用了以后，将会触发failover，但是，进行failover的那个Sentinel必须先获得至少3个Sentinel的授权才可以实行failover。
如果票数被设置为5，要达到ODOWN状态，必须所有5个Sentinel都主观认为Master为不可用，要进行failover，那么得获得所有5个Sentinel的授权。


## 选举领头Sentinel（即领导者选举）
一个Redis服务被判断为客观下线时，多个监视该服务的Sentinel协商，选举一个领头Sentinel，对该Redis服务进行故障转移操作。选举领头Sentinel遵循以下规则：
1. 所有的Sentinel都有公平被选举成领头的资格
2. 所有的Sentinel都有且只有一次将某个Sentinel选举成领头的机会（在一轮选举中），一旦选举某个Sentinel为领头，不能更改
3. Sentinel设置领头Sentinel是先到先得，一旦当前Sentinel设置了领头Sentinel，以后要求设置Sentinel为领头请求都会被拒绝
4. 每个发现服务客观下线的Sentinel，都会要求其他Sentinel将自己设置成领头
5. 当一个Sentinel（源Sentinel）向另一个Sentinel（目Sentinel）发送is-Master-down-by-addr ip port current_epoch runid命令的时候，runid参数不是*，而是Sentinel运行id，就表示源Sentinel要求目标Sentinel选举其为领头
6. 源Sentinel会检查目标Sentinel对其要求设置成领头的回复，如果回复的leader_runid和leader_epoch为源Sentinel，表示目标Sentinel同意将源Sentinel设置成领头
7. 如果某个Sentinel被半数以上的Sentinel设置成领头，那么该Sentinel既为领头
8. 如果在限定时间内，没有选举出领头Sentinel，暂定一段时间，再选举【要考虑Sentinel个数保障选举成功rac算法】
## 为什么要选领导者
简单来说，就是因为只能有一个Sentinel节点去完成故障转移。
Sentinel is-Master-down-by-addr这个命令有两个作用，一是确认下线判定，二是进行领导者选举。
选举过程：
1. 每个做主观下线的Sentinel节点向其他Sentinel节点发送上面那条命令，要求将它设置为领导者。
2. 收到命令的Sentinel节点如果还没有同意过其他的Sentinel发送的命令（还未投过票），那么就会同意，否则拒绝。
3. 如果该Sentinel节点发现自己的票数已经过半且达到了quorum的值，就会成为领导者
4. 如果这个过程出现多个Sentinel成为领导者，则会等待一段时间重新选举。
## 领导者选举
原因：只有一个Sentinel节点完成故障转移
选举：通过Sentinel is-Master-down-by-addr命令都希望成为领导者
1. 每个做主观下线的Sentinel节点向其他Sentinel节点发送命令，要求将它设置为领导者
2. 收到命令的Sentinel节点如果没有同意通过其他Sentinel节点发送的命令，那么将同意该请求，否则拒绝
3. 如果该Sentinel节点发现自己的票数已经超过Sentinel集合半数且超过quorum，那么它将称为领导者
4. 如果此过程有多个Sentinel节点成为了领导者，那么将等待一段时间重新进行选举
<font color="red">rac算法</font>