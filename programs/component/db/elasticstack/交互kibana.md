[TOC]

# 进入kibana界面
```
http://localhost：5601
```

# Discover

# Dev Tools
```
/*创建索引*/
PUT /test
{
  
}
/*查询索引*/
GET /test
{
  
}
/*删除索引*/
DELETE /test
{
  
}
```
创建test索引之后变成yellow状态
1583670788 12:33:08 elasticsearch yellow 1 1 4 4 0 0 1 0 - 80.0%

![](_v_images/20200308203454141_1002423842.png)

Primaries：主分片数量
Replicas：从节点从分片数量

yellow：主从都在一个node上

```
/*创建索引,只有主分片，没有从分片*/
PUT /test
{
  "settings": {
    "number_of_shards": 1
    , "number_of_replicas": 0
  }
}
```
test.Health(yellow-->green)




# number_of_shareds
# number_of_replicas

master节点只路由写请求，读请求可以每个节点自行处理

# 搭建集群

primaries与repliacs不在一个node上就green