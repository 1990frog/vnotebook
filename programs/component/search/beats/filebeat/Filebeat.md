[TOC]

# 概览
+ 读取日志文件，但不做数据的解析处理
+ 保证数据“at least once"至少被读取一次，即数据不会丢（但是有可能被重复消费）
+ 其他能力
处理多行数据
解析json格式数据
简单的过滤功能

# 使用流程
+ 安装：开箱即用
+ 配置：filebeat.yml
+ 配置模板：index template
+ 配置：kibana dashboards
+ 运行



# filebeat运行
```
./filebeat -e -c filebeat.yml -d "pushlish"
```
输出到stderr，默认输出到syslog和logs/filebeat文件

输出publish相关的debug日志




