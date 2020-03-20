[TOC]

`GET /index/_mapping`

# 概览
1. Mapping类似数据库中的schema的定义，作用如下：
    + 定义索引中的字段的名称
    + 定义字段的数据类型，例如字符串，数字，布尔......
    + 字段，倒排索引的相关配置，（Analyzed or Not Analyzed，Analyzer）
2. mapping会把json文档映射成lucene所需要的扁平格式
3. 一个mapping属于一个索引的type
    + 每个文档都属于一个type
    + 一个type有一个mapping定义
    + 7.0开始，不需要mapping定义中指定type信息
4. 对已有字段，一旦已经有数据写入，就不再支持修改字段定义，Lucene实现的倒排索引，一旦生成后，就不允许修改。如果希望改变字段类型，必须Reindex API，重建索引

# 语法
```json
PUT /test
{
  "mappings": {
    //自动检测日期类型
    "date_detection": "true",
    //定义日期格式
    "dynamic_date_formats":["MM/dd/yyyy"],
    //数字自动检测
    "numeric_detection": "true",
    "properties": {
      "user":{
        "type": "text",
        "index": "true",
        "fields": {
          "pinyin":{
            "type":"keyword"
          }
        }
      },
      "pwd":{
        "type": "text"
      }
    }
  },
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  }
}
```

# 自定义mapping的一些建议
为了减少输入的工作量，减少出错概率，可以依照以下步骤：
1. 创建一个临时的index，写入一些样本数据
2. 通过访问mapping api获得该临时文件的动态mapping定义
3. 修改后用，使用该配置创建你的索引
4. 删除临时索引

# 多字段类型
```json
"properties": {
    "user":{
    "type": "text",
    "index": "true",
    "fields": {
      "pinyin":{
        "type":"keyword"
      }
    }
}
```

# null_value
+ 需要对Null值实现搜索
+ 只有keyword类型支持设定null_value

```json
PUT index
{
    "mappings":{
        "properties":{
            "user":{
                "type":"keyword",
                "null_value":"NULL"
            }
        }
    }
}
```

# copy_to
+ _all在7中被copy_to所替代
+ 满足一些特定的搜索需求
+ copy_to将字段的数值拷贝到目标字段，实现类似_all的作用
+ copy_to的目标字段不出现在_source中

```json
PUT index
{
    "mappings":{
        "firstName":{
            "type":"text",
            "copy_to":"fullName"
        },"lastName":{
            "type":"text",
            "copy_to":"fullName"
        }
    }
}
```

# index
控制当前字段是否索引，默认为true，即记录索引，false不记录，即不可搜索
```json
"user":{
    "type": "text",
    "index": "true",
    "fields": {
      "pinyin":{
        "type":"keyword"
      }
    }
}
```

# index options
四种不同级别的index options配置，可以控制倒排索引记录的内容：
+ docs记录doc id
+ freqs记录doc id和term frequencies
+ positions记录doc id、term frequencies，term position
+ offsets记录doc id、term frequencies，term position、character offects

text类型默认记录positions，其他默认为docs
记录内容越多，占用存储空间越大


# Dynamic Mapping
+ 在写入文档时候，如果索引不存在，会自动创建索引
+ Dynamic Mapping的机制，使得我们无需手动定义mappings。Elasticsearch会根据文档信息，推算出字段的类型
+ 但是有时候会推算的不对，例如地理位置信息
+ 当类型如果设置不对时，会导致一些功能无法正常运行，例如Range查询

| json类型 |                                        Elasticsearch类型                                        |
| ------- | --------------------------------------------------------------------------------------------- |
| 字符串   | 匹配日期格式，设置成date</br>配置数字设置为float或者long，该选项默认关闭</br>设置为text，并且增加keyword子字段 |
| 布尔值   | boolean                                                                                        |
| 浮点数   | float                                                                                          |
| 整数     | long                                                                                           |
| 对象     | Object                                                                                         |
| 数组     | 由第一个非空数值的类型所决定                                                                        |
| 空值     | 忽略                                                                                           |


# dynamic
```json
PUT movies
{
    "mappings":{
        "_doc":{
            "dynamic":"false"
        }
    }
}
```

|              | true | false | strict |
| ------------ | ---- | ----- | ------ |
| 文档可索引     | yes  | yes   |     no   |
| 字段可索引     | yes  | no    |     no   |
| mapping被更新 | yes  |    no   |    no    |

+ 当dynamic设置为true时，一旦有新增字段的文档写入，mapping也同时被更新
+ 当dynamic被设置成false时候，mapping不会被更新，新增字段的数据无法被索引，但是信息会出现在_source中
+ 当设置成strict模式时候，数据写入直接报错

# Index Template
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