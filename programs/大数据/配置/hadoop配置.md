[TOC]
# 下载
`http://archive.cloudera.com/cdh5/cdh`
# hadoop
## 配置hadoop-env.sh
```
export JAVA_HOME = /usr/lib/jvm/java-8-openjdk
```
## 配置core-site.xml
```xml
<property>
<name>fs.defaultFS</name>
<value>hdfs://hadoop000:8020</value>
</property>
```
## 配置hadoop环境变量
```xml
$ vi ~/.bash_profile
export HADOOP_HOME=/opt/hadoop/hadoop-2.6.0-cdh5.15.1
export PATH=$HADOOP_HOME/bin:$PATH
$ source ~/.bash_profile
```
# hdfs
## 配置hdfs-site.xml
```xml
<property>
<name>dfs.replication</name>
<value>1</value>
</property>
<property>
<name>dfs.datanode.name.dir</name>
<value>/home/cai/App/hadoop/tmp/name</value>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>/home/cai/App/hadoop/tmp/dfs/data</value>
</property>
<property>
<name>dfs.permissions</name>
<value>false</value>
</property>
```

# mapreduce
## 配置mapred-site.xml
```xml
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```

# yarn
## 配置yarn-site.xml
```xml
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.resourcemanager.scheduler.class</name>
    <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>
<property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>512</value>
</property>
```




## 启动hdfs
第一次执行的时候一定要格式化文件系统，不要重复执行
```
hdfs namenode -format
```
赋权
```
sudo chmod -R 777 /home/cai/App/tmp
```


5.启动集群
$HADOOP_HOME/sbin/start-dfs.sh
停止
stop-dfs.sh

6.查看启动成功：
jps
http://127.0.0.1:50070
