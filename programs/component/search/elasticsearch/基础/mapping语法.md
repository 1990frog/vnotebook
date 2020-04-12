[TOC]

`GET /index/_mapping`

# 概览
1. Mapping类似数据库中的schema的定义，作用如下,定义字段名称、类型、分词器
2. mapping会把json文档映射成lucene所需要的扁平格式
3. 一个mapping属于一个索引的type（7.0开始，不需要mapping定义中指定type信息）
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


# 自动检测及动态映射Dynamic Mapping
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


```json
PUT movies
{
    "mappings":{
        "_doc":{
            # true，false，strict
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

# 字段类型
+ Text：被分析索引的字符串类型
+ Keyword：不能被分析只能被精确匹配的字符串类型
+ Date：日期类型，可以配合format一起使用
+ 数字类型：long，integer，short，double等
+ boolean：true，false
+ Array
+ Object：json嵌套
+ ip类型
+ geo_point：地理位置，经纬度

# 多字段类型
处于不同的目的，通过不同的方法索引相同的字段通常非常有用。这也是多字段的目的。例如，一个字符串字段可以映射为text字段用于全文本搜索，也可以映射为keyword字段用于排序或聚合。
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
对于相同索引中具有相同名称的字段，fields设置允许有不同的设置。可以使用PUT映射API将新的多字段添加到已存在的字段中。
```json
PUT /city
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 2
  },
  "mappings": {
    "properties": {
      "area": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart",
        "fields": {
          "pinyin": {
            "type": "text",
            "analyzer": "pinyin_analyzer"
          },
          "abbreviation": {
            "type": "keyword"
          }
        }
      },
      "number": {
        "type": "short"
      }
    }
  }
}
```
# null type
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

# copy to
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




