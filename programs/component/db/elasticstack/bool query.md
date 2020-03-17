[TOC]

# bool子句
|  关键字   |                  含义                  |  类比  |
| -------- | ------------------------------------- | ----- |
| filter   | 只过滤符合条件的文档，不计算相关性得分        | where |
| must     | 文档必须复合must中的所有条件，会影响相关性得分 | and   |
| must_not | 文档必须不符合must_not中的所有条件          | not   |
| should   | 文档可以符合should中的条件，会影响相关性得分  | or    |
都支持数组

## filter
filter查询只过滤符合条件的文档，不会进行相关性算分
es针对filter会有智能缓存，因此其执行效率很高
做简单匹配查询且不考虑算分时，推荐使用filter替代query等
## must
## must not
## should
should使用分两种情况
+ bool查询中只包含should，不包含must查询
+ bool查询中同时包含should和must查询

### 只包含should时，文档必须满足至少一个条件
minimum_should_match可以控制满足条件的个数或者百分比
### 同时包含should和must，文档不必满足should中的条件，但是如果满足条件，会增加相关性得分


# query context VS filter context
当一个查询语句位于Query或者Filter上下文时，es执行的结果会不同

|  上下文类型   |   执行类型  | 使用方式    |
| --- | --- | --- |
|  query   |   查找与查询语句最匹配的文档，对所有文档进行相关性算分并排序  |   query</br>bool中的must和should  |
|  filter   |  查找与查询语句相匹配的文档   |  bool中的filter与must_not</br>constant_score中的filter   |

# must搭配filter使用
将需要算分的放到must中
将不需要算分的放到filter中

must是query上下文，会进行算分
filter是filter上下文，不会影响算分，只会过滤复合条件的文档


# count api
获取复合条件的文档数，endpoint为_count
get index/_count

# source filtering
过滤返回结果中_source中的字段，主要有如下几种方式：
```
get index/_search?_source=username

get index/_search
{
    "_source":false
}

get index/_search
{
    "_source":["username","age"]
}

get index/_search
{
    "_source":{
          "includes":"*i*",
          "excludes":"birth"
    }
}
```


