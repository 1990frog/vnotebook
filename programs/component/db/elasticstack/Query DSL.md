[TOC]

# Compound queries
## Boolean
布尔(bool)过滤器--AND、OR、NOT查询：
+ must：所有的语句都必须（must）匹配，与`AND`等价，并且参与计算分值
+ must_not：所有的语句都不能（must not）匹配，与`NOT`等价
+ filter：返回的文档必须满足filter子句的条件。但是不会像Must一样，参与计算分值
+ should：至少有一个语句要匹配，与`OR`等价。
+ minimum_should_match：参数定义了至少满足几个子句
```
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
我们可以使用一个bool查询，对所有词条一视同仁
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
但是这个查询会给一份含有quick，red及brown的文档和一份含有quick，red及fox的文档完全相同的分数，然而在合并查询(Combining Queries)中，我们知道bool查询不仅能够决定一份文档是否匹配，同时也能够知道该文档的匹配程度。

下面是更好的查询方式：
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
现在，red和brown会在同一层次上相互竞争，而quick，fox以及red或者brown则是在顶层上相互对象的词条。

不完全的不(Not Quite Not)
在互联网上搜索"苹果"也许会返回关于公司，水果或者各种食谱的结果。我们可以通过排除pie，tart，crumble和tree这类单词，结合bool查询中的must_not子句，将结果范围缩小到只剩苹果公司：
```
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
但是有谁敢说排除了tree或者crumble不会将一份原本和苹果公司非常相关的文档也排除在外了呢？有时，must_not过于严格了。
## Boosting
它接受一个positive查询和一个negative查询。只有匹配了positive查询的文档才会被包含到结果集中，但是同时匹配了negative查询的文档会被降低其相关度，通过将文档原本的_score和negative_boost参数进行相乘来得到新的_score。

因此，negative_boost参数必须小于1.0

```
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
## Constant score
忽略TF/IDF，指定评分
boost默认1.0
```
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

GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "constant_score": {
          "query": { "match": { "description": "wifi" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "garden" }}
        }},
        { "constant_score": {
          "boost": 2
          "query": { "match": { "description": "pool" }}
        }}
      ]
    }
  }
}

# 指定评分
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
## Disjunction max
## Fuction score
# Full text queries
## Intervals
## Match
## Match phrase
## Match phrase prefix
## Multi-match
## Common Terms Query
## Query string
## Simple query string

---

# 直接查询索引
会查询出索引信息（mapping，setting）
```
GET /movie
```
# 通过id查询
```
GET /movie/_doc/1
```
# 查询
```
GET /movie/_search
{
  "query": {
    "match": {
      "title": "basketball"
    }
  }
}

POST /movie/_search
{
  "query": {
    "match": {
      "title": "basketball"
    }
  }
}
```

# term
完全匹配，也就是精确查询，搜索前不会再对搜索词进行分词拆解，如果索引中字段结构化存储使用了分词，那么使用term直接查询可能搜索不到
案例如下：
```
GET /movie/_search
{
  "query": {
    "term": {
      "title": "avatar"
    }
  }
}

{
    "_index" : "movie",
    "_type" : "_doc",
    "_id" : "1",
    "_score" : 10.278965,
    "_source" : {
      "title" : "Avatar",
      "tagline" : "Enter the World of Pandora.",
      "release_date" : "2009/12/10",
      "popularity" : "150.437577",
      "cast" : {
        "character" : "Jake Sully",
        "name" : "Sam Worthington"
       },
       "overview" : "In the 22nd century, a paraplegic Marine is dispatched to the moon Pandora on a unique mission, but becomes torn between following orders and protecting an alien civilization."
      }
}
```
解决此类问题的方式：
1. 将字段的type设置为keyword
2. 将该字段设置成not_analyzed无需分析的

# match
会先对搜索词进行分词,分词完毕后再逐个对分词结果进行匹配，因此相比于term的精确搜索，match是分词匹配搜索，match搜索还有两个相似功能的变种，一个是match_phrase，一个是multi_match
```
GET /movie/_search
{
  "query": {
    "match": {
      "title": "basketball"
    }
  }
}
```
# match_phrase
短语搜索，要求所有的分词必须同时出现在文档中，同时<font color="red">位置必须紧邻一致</font>
```
GET /movie/_search
{
  "query": {
    "match_phrase": {
      "title": "Love Basketball"
    }
  }
}
```
# multi_match
当我们想对多个字段进行匹配，其中一个字段包含分词就被文档就被搜索到时，可以用multi_match
```
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