[TOC]


# 组成
RocketMQ主要由Broker、NameServer、Producer和Consumer组成的一个集群
+ NameServer：整个集群的注册中心和配置中心，管理集群的元数据。包括Topic信息和路由信息、Producer 和 Consumer 的客户端注册信息、Broker 的注册信息。
+ Broker：负责接收消息的生产和消费请求，并进行消息的持久化和消息的读取。
+ Producer：负责生产消息。
+ Consumer：负责消费消息。

NameServer的主要功能是为整个MQ集群提供服务协调与治理，具体就是记录维护Topic、Broker的信息，及监控Broker的运行状态。为client提供路由能力（具体实现和zk有区别，NameServer是没有leader和follower区别的，不进行数据同步，通过Broker轮训修改信息）。

看下阿里中间件团队对NameServer特点总结：

Name Server是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。

Broker部署相对复杂，Broker分为Master与Slave，一个Master可以对应多个Slave，但是一个Slave只能对应一个Master，Master与Slave的对应关系通过指定相同的BrokerName，不同的BrokerId来定义，BrokerId为0表示Master，非0表示Slave。Master也可以部署多个。每个Broker与Name Server集群中的所有节点建立长连接，定时注册Topic信息到所有Name Server。

Producer与Name Server集群中的其中一个节点（随机选择）建立长连接，定期从Name Server取Topic路由信息，并向提供Topic服务的Master建立长连接，且定时向Master发送心跳。Producer完全无状态，可集群部署。

Consumer与Name Server集群中的其中一个节点（随机选择）建立长连接，定期从Name Server取Topic路由信息，并向提供Topic服务的Master、Slave建立长连接，且定时向Master、Slave发送心跳。Consumer既可以从Master订阅消息，也可以从Slave订阅消息，订阅规则由Broker配置决定。


# 网路
netty

# Topic主题
一类消息的集合，RocketMQ的基本订阅单位
# 消息模型
+ Producer（生产者）
+ Broker（消息代理，存储消息，转发消息）
+ Consumer（消费者，消费消息）
# 部署结构
+ Name Server（名字服务）：生产者/消费者通过名字服务查找各主题相应的Broker IP列表
+ Broker Server（代理服务器）：消费中转角色，负责存储消息、转发消息
# 消息模式
+ Pull Consumer（拉取式消费）：应用调用Consumer的拉取信息方法从Broker Server拉取消息
+ Push Consumer（推动式消费）：Broker收到消息后主动推送给消费端，该模式实时性较高
# Group组
+ Producer Group：生产者集合，一般用于发送一类消息
+ Consumer Group：消费者集合，一般用户接受一类消息进行消费
# 消费传播模式
+ Clustering（集群）：相同Consumer Group的每个Consumer实例平均分摊消息
+ Broadcasting（广播）：相同Consumer Group的每个Consumer实例都接收全量的消息
# 消息类型
普通消息、顺序消息、定时/延时消息、事务消息



