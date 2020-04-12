[TOC]

# 概览
定期收集操作系统、软件或服务的指标数据
存储在Elasticsearch中进行实时的分析

logs：用于记录离散的事件，具有随机性。例如，应用程序的调试信息或错误信息等
Metrics：用于记录度量或可聚合的数据，具有计划性。例如，服务的响应时长等

# 组成
Module：metricsbeat收集指标的对象，比如linux、windows、mysqld等
Metricset：metricbeat收集指标集合，该集合以减少收集指标调用次数为划分依据，1个module可以有多个metricset


# 下载
# Configuration
启动mysql模块
```
$ ./metricbeat modules enable mysql
```
配置输出
```yml
output.elasticsearch:
  hosts: ["myEShost:9200"]
```
启动
```
sudo ./metricbeat -e -c metricbeat.yml
```