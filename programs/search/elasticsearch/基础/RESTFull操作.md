[TOC]

# put 新增、构建索引
```json
PUT /index/_doc/1
{
  "code":10000,
  "value":"cd"
}
PUT /city
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":3
  },
  "mappings":{
    "properties":{
      "code":{"type":"integer"},
      "value":{"type":"text"}
    }
  }
}
```
## 两种方式
+ 非结构化方式新建索引(no mapping)
+ 使用结构化方式创建索引

## 非结构化方式新建索引
不指定mapping，es会自动对数据类型进行推算，这种方式创建的索引可能会浪费空间如下，默认使用大类型的long
方式1：指定文档id
```json
PUT /index/_doc/1
{
  "code":10000,
  "value":"cd"
}

GET /index/_mapping
{
  "index" : {
    "mappings" : {
      "properties" : {
        "code" : {
          "type" : "long"
        },
        "value" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```
方式2：自动生成文档id
```json
POST /index/_doc
{
  "code":10000,
  "value":"cd"
}
GET /index/_mapping
```

## 使用结构化方式创建索引
+ [settings](settings.md)
+ mappings
+ aliases
+ [基础类型](字段数据类型.md)

添加未定义的列不会出现异常，但是指定的数据类型与mapping设定的不相符就会报错
```json
PUT /city
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":3
  },
  "mappings":{
    "properties":{
      "code":{"type":"integer"},
      "value":{"type":"text"}
    }
  }
}
```
## 索引模板(Index Templates)
index templates帮助你设定mappings和settings，并按照一定的规则，自动匹配到新创建的索引之上
+ 模板仅在一个索引被新创建时，才会产生作用。修改模板不会影响已创建的索引
+ 可以设定多个索引模板，这些设置会被`merge`在一起
+ 你可以指定`order`的数值，控制`merging`的过程

```json
PUT _template/template_test
{
    "index_patterns":["test*],
    "order":1,
    "settings":{
        "number_of_shards":1,
        "number_of_replicas":2
    },
    "mappings":{
        "date_detection":false,
        "numeric_detection":true
    }
}
```
# post 新增、更新
+ update方法不会删除原来的文档，而是实现真正的数据更新
+ post方法/payload需要包含在”doc“中

```json
post index/_update/1
{
    "doc":{
        "field":"value"
    }
}
```
# delete 删除
```json
DELETE index/index_name
```

# 指定查询的索引
|          语法          |      范围       |
| --------------------- | -------------- |
| /_search               | 集群上所有的索引   |
| /index1/_search        | index1          |
| /index1,index2/_search | index1,index2   |
| /index*/_search        | 以index开头的索引 |