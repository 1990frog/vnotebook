[TOC]

# 下载
[清华源](https://mirrors.tuna.tsinghua.edu.cn/)

# 将mysql jdbc驱动存放在hive指定目录下
```
sudo cp /home/cai/.m2/repository/mysql/mysql-connector-java/8.0.20/mysql-connector-java-8.0.20.jar /usr/local/hive/lib/
```

# hive两个配置文件
## hive-env.sh
```
export JAVA_HOME=/soft/home/jdk1.8.0_191
export HADOOP_HOME=/soft/home/hadoop-2.8.5
export HIVE_CONF_DIR=/soft/home/apache-hive-2.3.6-bin/conf


export HADOOP_HOME=/usr/local/hadoop/3.2.1
export HIVE_CONF_DIR=/usr/local/hive/conf
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk
```
## hive-site.xml
在hive-site.xml的配置当中，我们主要设置了hive元数据库的链接信息，我们使用的是mysql数据库，所以制定了mysql数据库的jdbc地址、驱动、用户和密码等等。 还配置了Hive在HDFS上的一些相关的目录。接下来我们需要在HDFS上创建相关的目录。

hdfs dfs -mkdir /hive/warehouse
hdfs dfs -mkdir /hive/log
hdfs dfs -mkdir /hive/tmp
hdfs dfs -chmod -R 777 /hive

```
<configuration> 
  <property> 
    <name>javax.jdo.option.ConnectionURL</name>  
    <value>jdbc:mysql://localhost:3306/metastore?createDatabaseIfNotExist=true</value>  
    <description>the URL of the MySQL database</description> 
  </property>  
  <property> 
    <name>javax.jdo.option.ConnectionDriverName</name>  
    <value>com.mysql.jdbc.Driver</value>  
    <description>Driver class name for a JDBC metastore</description> 
  </property>  
  <property> 
    <name>javax.jdo.option.ConnectionUserName</name>  
    <value>root</value> 
  </property>  
  <property> 
    <name>javax.jdo.option.ConnectionPassword</name>  
    <value>root</value> 
  </property>  
  <property> 
    <name>hive.metastore.warehouse.dir</name>  
    <value>/hive/warehouse</value> 
   </property>
</configuration>
```