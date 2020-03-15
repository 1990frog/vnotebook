[TOC]

```
GET /_search
GET /index/_search
GET /index1,index2/_search
GET /my*/_search
```

# 查询主要有两种形式
+ URI Search
+ Request Body Search

```
GET /index/_search?q=user:alfred
```

```
GET /index/_search
{}
```

---

# Query DSL
基于json定义的查询语言，主要包含如下两种类型：
+ 字段类查询：如term,match,range等，只针对某一个字段进行查询
+ 复合查询：如bool查询等，包含一个或多个字段类查询或者复合查询语句


# 字段类查询
全文匹配
针对text类型的字段进行全文检索，会对查询语句先进行分词处理，如match，match_phrase等query类型

单词匹配
不会对查询语句做分词处理，直接去匹配字段的倒排索引，如term，terms，range等query类型

## 全文匹配
Match Query
通过operator参数可以控制单词间的匹配关系，可选项为or和and，默认是or
```
GET /index/_search
{
    "query":{
        "match":{
            "field":{
                "query":"",
                "operator":"and"
            }
        }
    }
}
```
and：同时包含多个词组的才符合查询条件

通过minimum_should_match参数可以控制需要匹配的单词数
```
GET /index/_search
{
    "query":{
        "match":{
            "field":{
                "query":"",
                "minimum_should_match":"2"# 至少匹配2个
            }
        }
    }
}
```

match phrase query
短语查询
```json
GET index/_search
{
    "query":{
        "match_phrase":{
            "job":{
                "query":"java engineer",
                "slop":"1"
            }
        }
    }
}
```
slop关键字

query-string query

simple-query-string query
类似query string，但是会忽略错误的查询语法，并且仅支持部分查询语法
其常用的逻辑符号如下，不能使用AND,OR,NOT等关键词
+ +代指AND
+ |代指OR
+ -代指NOT

term/terms query

range query
比较关键词：
+ gt-greater than
+ gte-greater than or equal to
+ lt-less than
+ lte-less than or equal to

date math
```
get:now-20y
```

# 复合查询
复合查询是指包含字段类查询或复合查询的类型，主要包括以下几类：
+ constant_score query
+ bool query
+ dis_max query
+ function_score query
+ boosting query

# constant score query
该查询将其内部的查询结果文档得分都设定为1或者boost的值
多用于结合bool查询实现自定义得分
```
GET /index/_search
{
    "query":{
        # 关键词
        "constant_score":{
            # 只能有一个
            "filter":{
                "match":{
                    "username":"alfred"
                }
            }
        }
    }
}
```