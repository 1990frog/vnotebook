[TOC]

即登录到sentinel节点后执行的命令，比如执行`redis-cli -h sentinel_ip -p26379`命令后，才可以执行下面命令
# ping：返回pong
```
ping
```
# masters：列出所有被监视的主服务器，以及这些主服务器的当前状态
```
SENTINEL masters
```
# slaves：列出给定主服务器的所有从服务器，以及这些从服务器的当前状态
```
SENTINEL slaves <mastername>
```
# get-master-addr-by-name：返回给定名字的主服务器的 IP 地址和端口号
如果这个主服务器正在执行故障转移操作， 或者针对这个主服务器的故障转移操作已经完成， 那么这个命令返回新的主服务器的 IP 地址和端口号
```
SENTINEL get-master-addr-by-name <master name>
```
# reset：重置所有名字和给定模式pattern相匹配的主服务器
pattern 参数是一个Glob风格的模式。 重置操作清楚主服务器目前的所有状态， 包括正在执行中的故障转移， 并移除目前已经发现和关联的， 主服务器的所有从服务器和Sentinel
```
SENTINEL reset <pattern>
```
# failover：当主服务器失效时， 在不询问其他 Sentinel 意见的情况下， 强制开始一次自动故障迁移
发起故障转移的Sentinel会向其他Sentinel发送一个新的配置，其他Sentinel会根据这个配置进行相应的更新
```
SENTINEL failover <master name>
```
# monitor：这个命令告诉sentinel去监听一个新的master
```
SENTINEL monitor <name> <ip> <port> <quorum>
```
# remove：命令sentinel放弃对某个master的监听
```
SENTINEL remove <name>
```
# set：这个命令很像Redis的CONFIG SET命令，用来改变指定master的配置。支持多个[option:value]
只要是配置文件中存在的配置项，都可以用SENTINEL SET命令来设置。这个还可以用来设置master的属性，比如说quorum(票数)，而不需要先删除master，再重新添加master
```
SENTINEL set <name> <option> <value>
SENTINEL SET objects-cache-master down-after-milliseconds 1000
```
# get-master-addr-by-name：获取当前的主服务器IP地址和端口号
```
SENTINEL get-master-addr-by-name <master name>
```
# slaves：获取所有的Slaves信息
```
SENTINEL slaves <master name>
```
