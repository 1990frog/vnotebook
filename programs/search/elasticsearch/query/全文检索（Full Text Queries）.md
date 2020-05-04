[TOC]

+ 索引和搜索时都会进行分词，查询字符串先传递到一个合适的分词器，然后生成一个供查询的词项列表
+ 查询时候，先会对输入的查询进行分词，然后每个词项逐个进行底层的查询，最终将结果进行合并。并未每个文档生成一个算法。

# Intervals
# 匹配查询（Match）
会先对搜索词进行分词,分词完毕后再逐个对分词结果进行匹配，因此相比于`term`的精确搜索，`match`是分词匹配搜索，`match`搜索还有两个相似功能的变种，一个是`match_phrase`，一个是`multi_match`
拓展关键字`operator`和`minimum_should_match`
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
# 匹配短语（Match phrase）
短语搜索，要求所有的分词必须同时出现在文档中，同时<font color="red">位置必须紧邻一致</font>
拓展关键字`slop`,`analyzer`
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
# 多字段匹配（Multi-match）
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
# 字符串搜索（Query string）
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
# 简化字符串查询（Simple query string）
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
# Match phrase prefix
match_bool_prefix查询内部将输入文本通过指定analyzer分词器处理为多个term，然后基于这些个term进行bool query，除了最后一个term使用前缀查询 其它都是term query。
# Common Terms Query