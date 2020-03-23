[TOC]

+ Term Suggester
+ Phrase Suggester

现代的搜索引擎，一般都会提供Suggest as you type的功能
帮助用户再输入搜索的过程中，进行自动补全或者纠错。通过协助用户输入更加精准的关键词，提高后续搜索阶段文档匹配的程度
在google上搜索，一开始会自动补全。当输入到一定长度，如因为单词拼写错误无法补全，就会卡死提示相似的词或者句子

搜索引擎中类似的功能，在Elasticsearch中是通过suggest api实现的
原理：将输入的文本分解为token，然后在索引的字典里查找相同的term并返回
根据不同的使用场景，Elasticsearch设计了4种类别的Suggesters
+ term & phrase suggester
+ complete & context suggester


每个建议都包含了一个算法，相似性是通过Levenshtein Edit Distance的算法实现的。核心思想就是一个词改动多少字符就可以和另一个词一致。提供了很多可选参数来控制相似性的模糊程度。例如“max_edits”

# Term Suggester
几种Suggestion Mode
+ Missing如索引中已经存在，就不提供建议
+ Popular推荐出现频率更加高的词
+ Always无论是否存在，都提供建议
# Phrase Suggester
Phrase Suggester在Term Suggester上增加了一些额外的逻辑
一些参数：
+ Suggest Mode：missing，popular，always
+ Max Errors：最多可以拼错的terms数
+ confidence：限制返回结果数，默认为1

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
```

```json

DELETE articles

POST articles/_bulk
{ "index" : { } }
{ "body": "lucene is very cool"}
{ "index" : { } }
{ "body": "Elasticsearch builds on top of lucene"}
{ "index" : { } }
{ "body": "Elasticsearch rocks"}
{ "index" : { } }
{ "body": "elastic is the company behind ELK stack"}
{ "index" : { } }
{ "body": "Elk stack rocks"}
{ "index" : {} }
{  "body": "elasticsearch is rock solid"}


POST _analyze
{
  "analyzer": "standard",
  "text": ["Elk stack  rocks rock"]
}

POST /articles/_search
{
  "size": 1,
  "query": {
    "match": {
      "body": "lucen rock"
    }
  },
  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "missing",
        "field": "body"
      }
    }
  }
}


POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "popular",
        "field": "body"
      }
    }
  }
}


POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "always",
        "field": "body",
      }
    }
  }
}


POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen hocks",
      "term": {
        "suggest_mode": "always",
        "field": "body",
        "prefix_length":0,
        "sort": "frequency"
      }
    }
  }
}


POST /articles/_search
{
  "suggest": {
    "my-suggestion": {
      "text": "lucne and elasticsear rock hello world ",
      "phrase": {
        "field": "body",
        "max_errors":2,
        "confidence":0,
        "direct_generator":[{
          "field":"body",
          "suggest_mode":"always"
        }],
        "highlight": {
          "pre_tag": "<em>",
          "post_tag": "</em>"
        }
      }
    }
  }
}
```