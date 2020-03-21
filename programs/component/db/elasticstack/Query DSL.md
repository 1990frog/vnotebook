[TOC]

# 复合查询（Compound queries）
## 多条件查询（Boolean）

布尔(bool)过滤器--AND、OR、NOT查询：
+ must：必须匹配，贡献算分，与`AND`等价，并且参与计算分值
+ must_not：查询字段，必须不能匹配（Filter Context），与`NOT`等价
+ filter：必须匹配，但是不能贡献算分（Filter Context）
+ should：选择性匹配，贡献算分，与`OR`等价。
+ minimum_should_match：参数定义了至少满足几个子句

+ 子查询可以任意顺序出现
+ 可以嵌套多个查询
+ 如果bool查询中，没有must条件，should中必须满足一条查询

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
多个bool嵌套
```json
POST _search
{
    "query":{
        "bool":{
            "must":{
                "term":{"price":"30"}
            },"should":[
                {
                    "bool":{
                        "must_not":{
                            "term":{
                                "avaliable":"false"
                            }
                        }
                    }
                }
            ],
            "mimimum_should_match":1
        }
    }
}
```

我们可以使用一个bool查询，对所有词条一视同仁（默认最少满足一个or条件）
```json
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
同一层级下的竞争字段，具有相同的权重
通过嵌套bool查询，可以改变对算分的影响
```json
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
控制字段的boosting
```json
POST _search
{
    "query":{
        "bool":{
            "should":[
                {
                    "match":{
                        "title":"apple,ipad",
                        "boost":4
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

Boosting是控制相关度的一种手段
参数boost的含义
+ 当boost>1时，打分的相关度相关性提升
+ 当0<boost<1时，打分的权重相对性降低
+ 当boost<0，贡献负分

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
将任何与任一查询匹配的文档作为结果返回。采用字段上最匹配的评分为最终评分返回

分离（Disjunction）的意思是 或（or） ，这与可以把结合（conjunction）理解成 与（and） 相对应。
分离最大化查询（Disjunction Max Query）指的是： 将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回

tie_breaker参数的意义，在于说，将其他query的分数，乘以tie_breaker的值，然后综合与最高分数的那个query分数，综合在一起进行计算。tie_breaker的值在0~1之间，是个小数。

使用tie_breaker会对评分求和

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
+ 索引和搜索时都会进行分词，查询字符串先传递到一个合适的分词器，然后生成一个供查询的词项列表
+ 查询时候，先会对输入的查询进行分词，然后每个词项逐个进行底层的查询，最终将结果进行合并。并未每个文档生成一个算法。
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

# 使用minimum_should_match
GET /movie/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Lost in Space",
        "minimum_should_match": 2
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
可以指定query对应的type：
+ best_fields
+ must_fields
+ cross_fields

最佳字段（Best Fields）
当字段之间互相竞争，又相互关联。例如title和body这样的字段。评分来自最匹配字段

多数字段（Most Fields）
处理英文内容时：一种常见的手段是，在主字段（English Analyzer），抽取词干，加入同义词，以匹配更多的文档。相同的文本，加入子字段（Standard Analyzer），以提供更加精确的匹配。其他字段作为匹配文档提高相关度的信号。匹配字段越多则越好

混合字段（Cross Fields）
对于某些实体，例如人名，地址，图书信息。需要在多个字段中确定信息，单个字段只能作为整体的一部分。希望在任何这些列出的字段中找到尽可能多的词


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

GET /movie/_search
{
  "query": {
    "multi_match": {
      "query": "Basketball",
      "fields": ["title","overview"],
      "type": "best_fields",
      "tie_breaker": 0.2,
      "minimum_should_match": "20%"
    }
  }
}
```
使用权重
```json
GET /movie/_search
{
  "query": {
    "multi_match": {
      "query": "Basketball",
      "fields": ["title^10","overview"],
      "type": "best_fields",
      "tie_breaker": 0.2,
      "minimum_should_match": "20%"
    }
  }
}
```
跨字段搜索，可以用copy_to解决，但是需要额外的存储空间，无法使用operator
cross_files支持operator，与copy_to对比优势，可以在搜索时为单个字段提升权重
```json
GET /movie/_search
{
  "query": {
    "multi_match": {
      "query": "Basketball",
      "fields": ["title^10","overview","city"],
      "type": "most_fields",
    }
  }
}

GET /movie/_search
{
  "query": {
    "multi_match": {
      "query": "Basketball",
      "fields": ["title^10","overview","city"],
      "type": "cross_fields",
      "operator": "and"
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
在es中，term查询，对输入不做分词。会将输入作为一个整体，在倒排索引中查找准确的词项，并且使用相关度算分公式为每个包含该词项的文档进行相关度算分，可以通过Constant Score将查询转换为一个filtering，避免算分，并利用缓存，提高性能

+ 将Query转成Filter，忽略TF-IDF计算，避免相关性算分的开销
+ Filter可以有效利用缓存

+ 布尔、时间、日期和数字这类结构化数据：有精确的格式，我们可以对这些格式进行逻辑操作。包括比较数字或时间的范围，或判断两个值的大小
+ 结构化的文本可以做精确匹配或者部分匹配：term查询/prefix前缀查询
+ 结构化结果只有”是“或”否“两个值：根据场景需要，可以决定结构化搜索是否需要打分
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


