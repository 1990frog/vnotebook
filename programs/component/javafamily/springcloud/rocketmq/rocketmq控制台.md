[TOC]

# 下载
github：`https://github.com/apache/rocketmq-externals.git`

# 更改版本
```xml
<rocketmq.version>4.5.1</rocketmq.version>
```
# 编译失败，注释插件
```xml
<!--<plugin>-->
    <!--<artifactId>maven-javadoc-plugin</artifactId>-->
    <!--<version>2.10.4</version>-->
    <!--<configuration>-->
        <!--<charset>UTF-8</charset>-->
    <!--</configuration>-->
    <!--<executions>-->
        <!--<execution>-->
            <!--<id>attach-javadocs</id>-->
            <!--<goals>-->
                <!--<goal>jar</goal>-->
            <!--</goals>-->
        <!--</execution>-->
    <!--</executions>-->
<!--</plugin>-->
```
# 修改代码
MessageServiceImpl
```java
// DefaultMQPullConsumer consumer = new DefaultMQPullConsumer(MixAll.TOOLS_CONSUMER_GROUP, null);
RPCHook rpcHook = null;
DefaultMQPullConsumer consumer = new DefaultMQPullConsumer(MixAll.TOOLS_CONSUMER_GROUP, rpcHook);
```
# 打包
```
$ mvn clean package -Dmaven.skip.test=true
```

# spring配置文件
```
server.port=8080
rocketmq.config.namesrvAddr=127.0.0.1:9876
```