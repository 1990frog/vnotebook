[TOC]

如果自己亲手去试一下，可以看到Google在用户刚开始输入的时候是自动补全的，而当输入到一定长度，如果因为单词拼写错误无法补全，就开始尝试提示相似的词。

Elasticsearch里设计了4种类别的Suggester:
Term Suggester
Phrase Suggester
Completion Suggester
Context Suggester


# 设置
通常Phrase suggester需要设置特别的mapping才能使用
```json

```
# 语法
建议现在query中查询，如果查询未匹配到，则返回建议
```json
POST twitter/_search
{
  "query" : {
    "match": {
      "message": "tring out Elasticsearch"
    }
  },
  "suggest" : {
    "my-suggestion" : {
      "text" : "tring out Elasticsearch",
      "term" : {
        "field" : "message"
      }
    }
  }
}
```
可以设置多个suggest
```json
POST _search
{
  "suggest": {
    "my-suggest-1" : {
      "text" : "tring out Elasticsearch",
      "term" : {
        "field" : "message"
      }
    },
    "my-suggest-2" : {
      "text" : "kmichy",
      "term" : {
        "field" : "user"
      }
    }
  }
}
```

# Term Suggester
```json
PUT /blogs
{
  "mappings": {
    "properties": {
        "body": {
          "type": "text"
        }
    }
  }
}

POST /blogs/_bulk
{ "index" : { } }
{ "body": "Lucene is cool"}
{ "index" : { } }
{ "body": "Elasticsearch builds on top of lucene"}
{ "index" : { } }
{ "body": "Elasticsearch rocks"}
{ "index" : { } }
{ "body": "Elastic is the company behind ELK stack"}
{ "index" : { } }
{ "body": "elk rocks"}
{ "index" : { } }
{  "body": "elasticsearch is rock solid"}

这些分出来的token都会成为词典里一个term，注意有些token会出现多次，因此在倒排索引里记录的词频会比较高，同时记录的还有这些token在原文档里的偏移量和相对位置信息。
POST _analyze
{
  "text": [
    "Lucene is cool",
    "Elasticsearch builds on top of lucene",
    "Elasticsearch rocks",
    "Elastic is the company behind ELK stack",
    "elk rocks",
    "elasticsearch is rock solid"
  ]
}

POST /blogs/_search
{
  "suggest": {
    "my-suggestion": {
      "text": "lucne rock",
      "term": {
        "suggest_mode": "missing",
        "field": "body"
      }
    }
  }
}
```
suggest就是一种特殊类型的搜索，DSL内部的"text"指的是api调用方提供的文本，也就是通常用户界面上用户输入的内容。这里的lucne是错误的拼写，模拟用户输入错误。 "term"表示这是一个term suggester。 "field"指定suggester针对的字段，另外有一个可选的"suggest_mode"。 范例里的"missing"实际上就是缺省值，它是什么意思？有点挠头... 还是先看看返回结果吧:


在返回结果里"suggest" -> "my-suggestion"部分包含了一个数组，每个数组项对应从输入文本分解出来的token（存放在"text"这个key里）以及为该token提供的建议词项（存放在options数组里)。  示例里返回了"lucne"，"rock"这2个词的建议项(options)，其中"rock"的options是空的，表示没有可以建议的选项，为什么？ 上面提到了，我们为查询提供的suggest mode是"missing",由于"rock"在索引的词典里已经存在了，够精准，就不建议啦。 只有词典里找不到词，才会为其提供相似的选项。

如果将"suggest_mode"换成"popular"会是什么效果？
尝试一下，重新执行查询，返回结果里"rock"这个词的option不再是空的，而是建议为rocks。


# phrase suggester
Phrase suggester在Term suggester的基础上，会考量多个term之间的关系，比如是否同时出现在索引的原文里，相邻程度，以及词频等等
```json
POST /blogs/_search
{
  "suggest": {
    "my-suggestion": {
      "text": "lucne and elasticsear rock",
      "phrase": {
        "field": "body",
        "highlight": {
          "pre_tag": "<em>",
          "post_tag": "</em>"
        }
      }
    }
  }
}
```
options直接返回一个phrase列表，由于加了highlight选项，被替换的term会被高亮。因为lucene和elasticsearch曾经在同一条原文里出现过，同时替换2个term的可信度更高，所以打分较高，排在第一位返回。Phrase suggester有相当多的参数用于控制匹配的模糊程度，需要根据实际应用情况去挑选和调试。

# Completion Suggester
最后来谈一下Completion Suggester，它主要针对的应用场景就是"Auto Completion"。 此场景下用户每输入一个字符的时候，就需要即时发送一次查询请求到后端查找匹配项，在用户输入速度较高的情况下对后端响应速度要求比较苛刻。因此实现上它和前面两个Suggester采用了不同的数据结构，索引并非通过倒排来完成，而是将analyze过的数据编码成FST和索引一起存放。对于一个open状态的索引，FST会被ES整个装载到内存里的，进行前缀查找速度极快。但是FST只能用于前缀查找，这也是Completion Suggester的局限所在。


