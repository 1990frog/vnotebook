[TOC]

# 大数据之4V特征
+ 数据量（Volume）
+ 基于高度分析的新价值（Value）
+ 速度（Velocity）
+ 多样性，复杂性（Variety）

# 结构化数据
结构化数据可以使用关系型数据库来表示和存储，如MySQL、Oracle、SQL Server等，表现二维形式的数据。可以通过固有键值获取相应信息。一般特点是：数据以行为单位，一行数据表示一个实体的信息，每一行数据的属性是相同的。结构化的数据的存储和排列是很有规律的，这对查询和修改等操作很有帮助。但是，显然，它的扩展性不好（比如，我希望增加一个字段）。

非结构化数据：
非结构化数据,就是没有固定结构的数据，包含全部格式的办公文档、文本、图片、XML、HTML、各类报表、图像和音频/视频信息等等。一般直接整体进行存储，而且一般存储为二进制的数据格式

半结构化数据：
半结构化数据可以通过灵活的键值调整获取相应信息，且数据的格式不固定，如json，同一键值下存储的信息可能是数值型的，可能是文本型的，也可能是字典或者列表。 
半结构化数据，属于同一类实体可以有不同的属性，即使他们被组合在一起，这些属性的顺序并不重要。常见的半结构数据有XML和JSON。

大数据带来的技术变革：
技术驱动：数据量大
存储：文件存储==>分布式存储
计算：单机==>分布式计算
网络：万兆
DB：RDBMS ==> NoSQL（HBase/Redis...）

大数据现存的模式：
手握大数据、没有大数据思维
没有大数据、有大数据思维
既有大数据、又有大数据思维


Google大数据技术：
存储容量：MapReduce
读写速度：BigTable
计算效率：GFS

大数据典型应用：
报表、用户细分、指标监控、指标预警


reliable、scalable、distributed computing


Hadoop：提供分布式的存储（一个文件被拆分成很多个块，并且以副本的方式存储在各个节点中）和计算

分布式文件系统：HDFS
分布式计算框架：MapReduce
分布式资源调度框架：YARN

HDFS是GFS的克隆版
HDFS特点：扩展性&容错性&海量数据存储
 
HDFS：将文件切分成指定大小的数据块并以多副本的存储在多个机器上
文件、块、副本
文件：test.log 200M
块（block）：默认的blocksize是128M，2个块：1个128m，1个72m
副本：HDFS默认3副本

node1:blk1 blk2
node2:blk1 blk2
node3:blk2
node4:blk1
node5:


MapReduce是Google MapReduce的克隆版
MapReduce特点：扩展性&容错性&海量数据离线处理

YARM：Yet Another Resource Negotiator
负责整个集群资源的管理和调度
YARN特点：扩展性&容错性&多框架资源统一调度

Hadoop优势之高可靠性：
数据存储：数据块多副本
数据计算：重新调度作业计算
Hadoop优势之高扩展性：
存储/计算资源不够时，可以横向的线性扩展机器
一个集群中可以包含数以千计的节点
Hadoop优势之其他：
存储在廉价的机器上，降低成本
成熟的生态圈

去IoE


HDFS前提和设计目标：
Hardware Failure 硬件错误
每个机器只存储文件的部分数据，blocksize=128M
block存放在不同的机器上的，由于容错，HDFS默认采用3副本机制

Streaming Data Access 流式数据访问
The emphasis is on high throughput of data access rather than low latency of data access.
Large Data Sets
Moving Computation is Cheaper than Moving Data 

HDFS的架构*****
1）NameNode and DataNodes
2）master/slave的架构
3）NN：the file system namespace
	/home/hadoop/software
		         /app
4）DN：storage
5）HDFS exposes a file system namespcae and allows user 
6）a file is split into one or more blocks
7）blocks are stored in a set of DataNodes 容错
8）NameNode executes file system namespace operations：CRUD
9）determine the mapping of blocks to DataNodes
	a.txt 150M blocksize=128m
	a.txt 拆分成2个block 一个是128m 另一个是22m
	block1存放在哪个DN?block2存放在哪个DN?
	a.txt
		block1:128m,192.xx.xx.1
		block2:22m,192.xx.xx.2
	get a.txt
	这个过程对于用户来说是不暴露的，不感知的
10）通常情况下一个node布置一个组件

版本选择：CDH,HDP,不使用Apath社区版

Hadoop/Hive/Spark相关框架的学习：使用单机版足够
如果使用集群学习会导致：从入门到放弃


hadoop软件常见目录说明：
bin：hadoop客户端名单
etc/hadoop：hadoop相关的配置文件存放目录
sbin：启动hadoop相关进程的脚本
share：常用例子

Hadoop（HDFS）安装：
下载
解压
添加HADOOP_HOME/bin到系统环境变量
修改Hadoop配置文件
hadoop-env.sh
export JAVA_HOME=...
core-site.xml
<property>
<name>fs.defaultFS</name>
<value>hdfs://hadoop000:8020</value>
</property>

hdfs-site.xml
<property>
<name>dfs.replication</name>
<value>1</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>/home/cai/hadoop/hdfs</value>
</property>

slaves
127.0.0.1/hadoop000

启动HDFS：第一次执行的时候一定要格式化文件系统，不要重复执行
$HADOOP_HOME/bin
hdfs namenode -format
启动集群：
$HADOOP_HOME/sbin
./start-dfs.sh
验证：jps
60002 DataNode
60171 SecondaryNameNode
59870 NameNode
http://lcoalhost:50070
如果发现jps ok，但是浏览器不ok，十有八九是防火墙问题
sudo firewall -cmd --state
sudo systemctl stop firewalld.service

停止
./stop-dfs.sh

注意：
start/stop-dfs.sh与hadoop-daemons.sh的关系
start-dfs.sh=
hadoop-daemons.sh start namenode
hadoop-daemons.sh start datanode
hadoop-daemons.sh start secondarynamenode
stop-dfs.sh=
...

HDFS命令行操作*****
hadoop fs 查看全部命令
hadoop fs -ls /
hadoop fs -put
hadoop fs -cat
hadoop fs -text
hadoop fs -copyFromLocal
hadoop fs -moveFromLocal
hadoop fs -get
hadoop fs -mkdir
hadoop fs -mv
hadoop fs -getmerge
hadoop fs -rm
hadoop fs -rmdir
hadoop fs -rm -r


HDFS存储扩展：
put：1file ==>1...n block ==>存放在不同的节点上的
get：去nn上查找这个file对应的元数据信息

jdk.tgz==> blk_xxxx1,blk_xxxx2
cat blk1 >> jdk.tgz
cat blk2 >> jdk.tgz
jdk.tgz = jdk.tgz

HDFS：文件的切割，文件的合并，创建副本

了解底层的存储机制这才是我们真正要学习的东西，掌握API那是毛毛雨
--------------------------------------------------------------------------
centos
/etc/sysconfig/network-scipts/

cat /etc/sysconfig/network
cat /etc/resolv.conf

拷贝本地软件包到服务器：scp

--------------------------------------------------------------------------
ssh-keygen -t rsa 一路回车
cd ~/.ssh
ll
id_rsa 私钥
id_rsa.pub 公钥
known_hosts

cat id_rsa.pub >> authorized_keys

--------------------------------------------------------------------------

一、HDFS
1.安装ssh
systemctl enable sshd.service 开机启动
systemctl start sshd.service 立即启动


1.解压
2.配置hadoop/etc/hadoop/hadoop-env.sh中JAVA_HOME 
3.Configuration
1）etc/hadoop/core-site.xml:
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:8020</value> 
    </property>
</configuration>

2）etc/hadoop/hdfs-site.xml:
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>//副本数 
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/cai/App/tmp</value>
    </property>
    	
</configuration>

3）添加hadoop到根目录环境变量：
$ vi ~/.bash_profile
export HADOOP_HOME=/opt/hadoop/hadoop-2.6.0-cdh5.15.1
export PATH=$HADOOP_HOME/bin:$PATH
$ source ~/.bash_profile


4.启动HDFS：第一次执行的时候一定要格式化文件系统，不要重复执行
$ hdfs namenode -format

4.赋予权限
$ sudo chmod -R 777 /home/cai/App/tmp

5.启动集群
$HADOOP_HOME/sbin/start-dfs.sh
停止
stop-dfs.sh

6.查看启动成功：
jps
http://127.0.0.1:50070



---------------------------------------------------------------------------
# slaves文件
# linux /tmp目录重启会被清空



---------------------------------------------------------------------------



Usage: hadoop fs [generic options]
        [-appendToFile <localsrc> ... <dst>]
        [-cat [-ignoreCrc] <src> ...]
        [-checksum <src> ...]
        [-chgrp [-R] GROUP PATH...]
        [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
        [-chown [-R] [OWNER][:[GROUP]] PATH...]
        [-copyFromLocal [-f] [-p] [-l] <localsrc> ... <dst>]
        [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-count [-q] [-h] [-v] [-x] <path> ...]
        [-cp [-f] [-p | -p[topax]] <src> ... <dst>]
        [-createSnapshot <snapshotDir> [<snapshotName>]]
        [-deleteSnapshot <snapshotDir> <snapshotName>]
        [-df [-h] [<path> ...]]
        [-du [-s] [-h] [-x] <path> ...]
        [-expunge]
        [-find <path> ... <expression> ...]
        [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-getfacl [-R] <path>]
        [-getfattr [-R] {-n name | -d} [-e en] <path>]
        [-getmerge [-nl] <src> <localdst>]
        [-help [cmd ...]]
        [-ls [-C] [-d] [-h] [-q] [-R] [-t] [-S] [-r] [-u] [<path> ...]]
        [-mkdir [-p] <path> ...]
        [-moveFromLocal <localsrc> ... <dst>]
        [-moveToLocal <src> <localdst>]
        [-mv <src> ... <dst>]
        [-put [-f] [-p] [-l] <localsrc> ... <dst>]
        [-renameSnapshot <snapshotDir> <oldName> <newName>]
        [-rm [-f] [-r|-R] [-skipTrash] <src> ...]
        [-rmdir [--ignore-fail-on-non-empty] <dir> ...]
        [-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
        [-setfattr {-n name [-v value] | -x name} <path>]
        [-setrep [-R] [-w] <rep> <path> ...]
        [-stat [format] <path> ...]
        [-tail [-f] <file>]
        [-test -[defsz] <path>]
        [-text [-ignoreCrc] <src> ...]
        [-touchz <path> ...]
        [-usage [cmd ...]]



hadoop fs -ls -R /


HDFS前提、设计目标
Hardware Failure
Hardware failure is the norm rather than the exception. An HDFS instance may consist of hundreds or thousands of server machines, each storing part of the file system’s data. The fact that there are a huge number of components and that each component has a non-trivial probability of failure means that some component of HDFS is always non-functional. Therefore, detection of faults and quick, automatic recovery from them is a core architectural goal of HDFS.

Streaming Data Access
Applications that run on HDFS need streaming access to their data sets. They are not general purpose applications that typically run on general purpose file systems. HDFS is designed more for batch processing rather than interactive use by users. The emphasis is on high throughput of data access rather than low latency of data access. POSIX imposes many hard requirements that are not needed for applications that are targeted for HDFS. POSIX semantics in a few key areas has been traded to increase data throughput rates.
关键是吞吐量，而不是低延迟

Large Data Sets
Applications that run on HDFS have large data sets. A typical file in HDFS is gigabytes to terabytes in size. Thus, HDFS is tuned to support large files. It should provide high aggregate data bandwidth and scale to hundreds of nodes in a single cluster. It should support tens of millions of files in a single instance.

Simple Coherency Model
HDFS applications need a write-once-read-many access model for files. A file once created, written, and closed need not be changed except for appends and truncates. Appending the content to the end of the files is supported but cannot be updated at arbitrary point. This assumption simplifies data coherency issues and enables high throughput data access. A MapReduce application or a web crawler application fits perfectly with this model.

Moving Computation is Cheaper than Moving Data
A computation requested by an application is much more efficient if it is executed near the data it operates on. This is especially true when the size of the data set is huge. This minimizes network congestion and increases the overall throughput of the system. The assumption is that it is often better to migrate the computation closer to where the data is located rather than moving the data to where the application is running. HDFS provides interfaces for applications to move themselves closer to where the data is located.

Portability Across Heterogeneous Hardware and Software Platforms
HDFS has been designed to be easily portable from one platform to another. This facilitates widespread adoption of HDFS as a platform of choice for a large set of applications.


不要写硬编码：非常忌讳
写成可配置的


HDFS的元数据管理：
元数据：HDFS的目录结构以及每个的BLOCK信息（id、副本系数、block存放在那个ND上）
存在什么地方：${hadoop.tmp.dir}/name/......

checkpoint

safemode


-------------------------------------------------------
![Image text](../../../resources/image/hdfs_01.jpg)
NameNode：HDFS群集包含单个NameNode（主服务器），它管理文件系统命名空间并控制客户端对文件的访问权限。
它维护和管理文件系统元数据；例如由哪些块构成文件，以及存储这些块的数据节点。

DataNode可以有多个DataNode，通常是集群中每个节点有一个DataNode，它负责管理着运行节点的存储访问。
HDFS中的DataNode存储实际数据，可以添加更多的DataNode来增加可用空间。

备用NameNode ：备用NameNode服务并非真正的备用NameNode，尽管名称是称为备用NameNode。
具体来说，它并不为NameNode提供高可用性(HA)。









 


































