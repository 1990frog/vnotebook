
Completion Suggester提供了“自动完成”（Auto Complete）的功能。用户每输入一个字符，就需要即时发送一个查询请求到后端查找匹配项

对性能要求比较苛刻。Elasticsearch采用了不同的数据结构，并非通过倒排索引来完成。而是将Analyze的数据编码成FST和索引一起存放。FST会被ES整个加载进内存，速度很快

FST只能用于前缀查找


# 使用Completion Suggester的一些步骤
+ 定义Mapping，使用“completion“type
+ 索引数据
+ 运行”suggest“查询，得到搜索建议

# 实现Context Suggester
可以定义两种类型的Context
+ Category任意的字符串
+ Geo地理位置信息

实现Context Suggester的具体路径
+ 定制一个Mapping
+ 索引数据，并且为每个文档加入Context信息
+ 结合Context进行Suggestion查询

# 精准度和召回率
+ 精准度：Completion>Phrase>Term
+ 召回率：Term>Phrase>Completion
+ 性能：Completion>Phrase>Term

```json
PUT articles
{
  "mappings": {
    "properties": {
      "title_completion":{
        //设置为completion才能被suggest捕获
        "type": "completion"
      }
    }
  }
}
```

```json
DELETE articles
PUT articles
{
  "mappings": {
    "properties": {
      "title_completion":{
        "type": "completion"
      }
    }
  }
}

POST articles/_bulk
{ "index" : { } }
{ "title_completion": "lucene is very cool"}
{ "index" : { } }
{ "title_completion": "Elasticsearch builds on top of lucene"}
{ "index" : { } }
{ "title_completion": "Elasticsearch rocks"}
{ "index" : { } }
{ "title_completion": "elastic is the company behind ELK stack"}
{ "index" : { } }
{ "title_completion": "Elk stack rocks"}
{ "index" : {} }


POST articles/_search?pretty
{
  "size": 0,
  "suggest": {
    "article-suggester": {
      "prefix": "elk ",
      "completion": {
        "field": "title_completion"
      }
    }
  }
}


DELETE comments
PUT comments
PUT comments/_mapping
{
  "properties": {
    "comment_autocomplete":{
      "type": "completion",
      "contexts":[{
        "type":"category",
        "name":"comment_category"
      }]
    }
  }
}

POST comments/_doc
{
  "comment":"I love the star war movies",
  "comment_autocomplete":{
    "input":["star wars"],
    "contexts":{
      "comment_category":"movies"
    }
  }
}

POST comments/_doc
{
  "comment":"Where can I find a Starbucks",
  "comment_autocomplete":{
    "input":["starbucks"],
    "contexts":{
      "comment_category":"coffee"
    }
  }
}


POST comments/_search
{
  "suggest": {
    "MY_SUGGESTION": {
      "prefix": "sta",
      "completion":{
        "field":"comment_autocomplete",
        "contexts":{
          "comment_category":"coffee"
        }
      }
    }
  }
}

```