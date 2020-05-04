[TOC]





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


