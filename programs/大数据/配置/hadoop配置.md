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
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://localhost:8082</value>
        </property> 
</configuration>
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
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
        <property>
                <name>dfs.replication</name>
                <value>1</value>
        </property>
        <property>
                <name>dfs.datanode.name.dir</name>
                <value>/home/cai/App/hadoop/tmp/3.2.1/name</value>
        </property>
        <property>
                <name>dfs.permissions</name>
        </property>
</configuration>
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
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->
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
</configuration>

```




## 启动hdfs
第一次执行的时候一定要格式化文件系统，不要重复执行
```
hdfs namenode -format
```
赋权
```
sudo chmod -R 777 /home/cai/App/hadoop/tmp
```



5.启动集群
$HADOOP_HOME/sbin/start-dfs.sh
停止
stop-dfs.sh

6.查看启动成功：
jps
http://127.0.0.1:50070
