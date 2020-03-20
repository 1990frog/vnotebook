[TOC]

# 复合查询（Compound queries）
## 多条件查询（Boolean）
布尔(bool)过滤器--AND、OR、NOT查询：
+ must：所有的语句都必须（must）匹配，与`AND`等价，并且参与计算分值
+ must_not：所有的语句都不能（must not）匹配，与`NOT`等价
+ filter：返回的文档必须满足filter子句的条件。但是不会像Must一样，参与计算分值
+ should：至少有一个语句要匹配，与`OR`等价。
+ minimum_should_match：参数定义了至少满足几个子句
```json
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```
我们可以使用一个bool查询，对所有词条一视同仁（默认最少满足一个or条件）
```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "text": "quick" }},
        { "term": { "text": "brown" }},
        { "term": { "text": "red"   }},
        { "term": { "text": "fox"   }}
      ]
    }
  }
}
```
上述方式会将不同类型的条件放到一个维度进行比较

下面是更好的查询方式
```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "text": "quick" }},
        { "term": { "text": "fox"   }},
        {
          "bool": {
            "should": [
              { "term": { "text": "brown" }},
              { "term": { "text": "red"   }}
            ]
          }
        }
      ]
    }
  }
}
```
不完全的不(Not Quite Not)
在互联网上搜索"苹果"也许会返回关于公司，水果或者各种食谱的结果。我们可以通过排除pie，tart，crumble和tree这类单词，结合bool查询中的must_not子句，将结果范围缩小到只剩苹果公司：
```json
GET /_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "text": "apple"
        }
      },
      "must_not": {
        "match": {
          "text": "pie tart fruit crumble tree"
        }
      }
    }
  }
}
```
## 调整相关度（Boosting）
它接受一个positive查询和一个negative查询。只有匹配了positive查询的文档才会被包含到结果集中，但是同时匹配了negative查询的文档会被降低其相关度，通过将文档原本的_score和negative_boost参数进行相乘来得到新的_score。

因此，negative_boost参数必须小于1.0

```json
GET /_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "text" : "apple"
                }
            },
            "negative" : {
                 "term" : {
                     "text" : "pie tart fruit crumble tree"
                }
            },
            "negative_boost" : 0.5
        }
    }
}
```
## 常数分值（Constant score）
忽略TF/IDF，指定评分。boost默认1.0分

两种使用方法：
+ query+constant_score+filter
+ query+bool+[must/should/filter/must_not]+constant_score+filter
```json
GET /_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                "term" : { "user" : "kimchy"}
            },
            "boost" : 1.2
        }
    }
}

GET /movie/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "constant_score":
          {
            "filter": {
              "term": {
                "title": "space"
              }
            },
            "boost": 10
          }
        },
        {
          "match":
          {
            "title": "Basketball"
          }
        }
      ]
    }
  }
}
```
## 分离最大化查询（Disjunction max）
分离（Disjunction）的意思是 或（or） ，这与可以把结合（conjunction）理解成 与（and） 相对应。
分离最大化查询（Disjunction Max Query）指的是： 将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回

tie_breaker参数的意义，在于说，将其他query的分数，乘以tie_breaker的值，然后综合与最高分数的那个query分数，综合在一起进行计算。tie_breaker的值在0~1之间，是个小数。

```json
GET /movie/_search
{
  "query": {
    "dis_max": {
      "tie_breaker": 0.7,
      "boost": 1.2,
      "queries": [
        {"match": {"title": "space"}},
        {"match": {"tagline": "Robinson"}}
      ]
    }
  }
}
```

## 函数打分（Fuction score）用不上先不学了

# 全文检索（Full text queries）
## Intervals
## 匹配查询（Match）
会先对搜索词进行分词,分词完毕后再逐个对分词结果进行匹配，因此相比于term的精确搜索，match是分词匹配搜索，match搜索还有两个相似功能的变种，一个是match_phrase，一个是multi_match
```json
# 不使用连接符简写
GET /movie/_search
{
  "query": {
    "match": {
      "title": "Lost in Space",
    }
  }
}

# 使用连接符
GET /movie/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Lost in Space",
        "operator": "and"
      }
    }
  }
}
```
## 匹配短语（Match phrase）
短语搜索，要求所有的分词必须同时出现在文档中，同时<font color="red">位置必须紧邻一致</font>
```json
# 简写
GET /movie/_search
{
  "query": {
    "match_phrase": {
      "tagline":"Space will never be the same"
    }
  }
}
# 容错，允许短语间隔一个token
GET /movie/_search
{
  "query": {
    "match_phrase": {
      "tagline":{
        "query": "Space never be the same",
        "slop": 1
      }
    }
  }
}
# 指定analyzer
GET /movie/_search
{
  "query": {
    "match_phrase": {
      "tagline":{
        "query": "Space never be the same",
        "analyzer":"my_analyzer",
        "slop": 1
      }
    }
  }
}
```
## Match phrase prefix
match_bool_prefix 查询内部将输入文本通过指定analyzer分词器处理为多个term，然后基于这些个term进行bool query，除了最后一个term使用前缀查询 其它都是term query。
## 多字段匹配（Multi-match）
当我们想对多个字段进行匹配，其中一个字段包含分词就被文档就被搜索到时，可以用multi_match
```json
GET /movie/_search
{
  "query": {
    "multi_match": {
      "query": "Basketball",
      "fields": ["title","overview"]
    }
  }
}
```
## Common Terms Query
## 字符串搜索（Query string）
方便使用AND OR NOT
```json
GET /movie/_search
{
  "query": {
    "query_string": {
      "fields": ["title"],
      "query": "steve AND jobs"
    }
  }
}
```
range类似between
```
GET /movie/_search
{
  "query": {
    "bool": {
      "filter": 
        [
          {"term":{"title":"steve"}},  
          {"term":{"cast.name":"gaspard"}},  
          {"range":{"relase_date":{"lte":"2015/01/01"}}},
          {"range":{"popularity":{"gte":"25"}}}
        ]
    }
  }
}
```
## 简化字符串查询（Simple query string）
```json
GET /_search
{
  "query": {
    "simple_query_string" : {
        "query": "\"fried eggs\" +(eggplant | potato) -frittata",
        "fields": ["title^5", "body"],
        "default_operator": "and"
    }
  }
}

```
# 匹配全部（match all）
```json
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```
# 词汇等级查询（Term-level queries）
## Exists
## Fuzzy
## IDs
## Prefix
## 数值范围查询（Range）
```json
GET /_search
{
    "query": {
        "range" : {
            "age" : {
                "gte" : 10,
                "lte" : 20,
                "boost" : 2.0
            }
        }
    }
}
```
## Regexp
## 单词检索（Term）
完全匹配，也就是精确查询，搜索前不会再对搜索词进行分词拆解，如果索引中字段结构化存储使用了分词，那么使用term直接查询可能搜索不到
解决此类问题的方式：
1. 将字段的type设置为keyword
2. 将该字段设置成not_analyzed无需分析的
```json
GET /movie/_search
{
  "query": {
    "term": {
      "title": "avatar"
    }
  }
}
```
## 多单词检索（Terms）
```json
GET /movie/_search
{
  "query": {
    "term": {
      "title": ["avatar","hello"]
    }
  }
}
```
## Terms set
## Type Query
## Wildcard


