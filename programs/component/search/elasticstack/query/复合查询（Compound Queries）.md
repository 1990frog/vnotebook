[TOC]

# Boolean多条件查询
<font color="red">可以将bool看成where</font>
<font color="red">bool可以嵌套</font>

布尔(bool)过滤器--AND、OR、NOT查询：
|        关键字        |                                     解释                                      |
| ------------------- | ---------------------------------------------------------------------------- |
| must                |                  必须匹配，贡献算分，与`AND`等价，并且参与计算分值                                                            |
| must_not             | 查询字段，必须不能匹配（Filter Context），与`NOT`等价                               |
| filter              | 必须匹配，但是不能贡献算分（Filter Context），<font color="red">不想算分就用这个</font> |
| should              | 选择性匹配，贡献算分，与`OR`等价。<font color="red">可以分为多级，不同级别分值不同</font> |
| minimum_should_match | 参数定义了至少满足几个子句                                                        |
+ 子查询可以任意顺序出现
+ 可以嵌套多个查询
+ 如果bool查询中，没有must条件，should中必须满足一条查询（未指定`minimum_should_match`的情况）

## 层级
1. query
2. bool【bool可以嵌套】
3. must|must_not|filter|should
4. term|match|range|[bool]
5. boost

## 基础用法
```json
POST _search //不加index查询全部索引
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
## 多个bool嵌套
```json
POST _search
{
    "query":{
        "bool":{
            "must":{
                "term":{"price":"30"}
            },
            "should":[
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
## should层级
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
<font color="red">同一层级下的竞争字段，具有相同的权重</font>
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
          "bool": { // 最外层的分数比较高
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
## 设定权重
boost参数被用来增加一个子句的相对权重(当boost大于1时)，或者减小相对权重(当boost介于0到1时)，但是增加或者减小不是线性的。
换言之，boost设为2并不会让最终的_score加倍。
相反，新的_score会在适用了boost后被归一化(Normalized)。
每种查询都有自己的归一化算法(Normalization Algorithm)，算法的细节超出了本书的讨论范围。
但是能够说一个高的boost值会产生一个高的_score。
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
## 不完全的不(Not Quite Not)
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
# Boosting调整相关度
Boosting是控制相关度的一种手段，它接受一个`positive`查询和一个`negative`查询。
+ positive：只有匹配了`positive`查询的文档才会被包含到结果集中（命中文档）
+ negative：匹配了`negative`查询的文档会被降低其相关度（debuff）
+ 得分：通过将文档原本的`_score`和`negative_boost`参数进行相乘来得到新的`_score`（因此，negative_boost参数必须小于1.0，反过来可以加分哈）

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
# Constant score常数分值
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
# Disjunction max分离最大化查询
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

# Fuction score函数打分

3.match分词后的and和or
GET /movie/_search
{
  "query":{
    "match":{"title":"basketball with cartoom aliens"},
  }
}
使用的是or
GET /movie/_search
{
  "query":{
    "match": {
      "title": {
        "query": "basketball with cartoom aliens",
        "operator": "and" 
      }
    }
  } 
}
使用and

4.最小词项匹配
GET /movie/_search
{
  "query":{
￼    "match": {
      "title": {
        "query": "basketball with cartoom aliens",
        "operator": "or" ,
        "minimum_should_match": 2
      }
    }
  }
}

5.短语查询
GET /movie/_search
{
  "query":{
    "match_phrase":{"title":"steve zissou"}
  }
}
短语前缀查询
GET /movie/_search
{
  "query":{
    "match_phrase_prefix":{"title":"steve zis"}
  }
}

6.多字段查询
GET /movie/_search
{
  "query":{
    "multi_match":{
      "query":"basketball with cartoom aliens",
      "field":["title","overview"]
    }
  }
}


￼操作不管是字符与还是或，按照逻辑关系命中后相加得分

GET /movie/_search
{
  "explain": true, 
  "query":{
    "match":{"title":"steve"}
  }
}
查看数值，tfidf多少分，tfnorm归一化后多少分

多字段查询索引内有query分词后的结果，因为title比overview命中更重要，因此需要加权重
GET /movie/_search
{
  "query":{
    "multi_match":{
      "query":"basketball with cartoom aliens",
      "fields":["title^10","overview"],
      "tie_break":0.3
    }
  }
}


继续深入查询：
Bool查询

must：必须都是true
must not： 必须都是false
should：其中有一个为true即可，但true的越多得分越高
GET /movie/_search
{
  "query":{
    "bool": { 
      "should": [
        { "match": { "title":"basketball with cartoom aliens"}}, 
        { "match": { "overview":"basketball with cartoom aliens"}}  
￼      ]
    }
  }
}


2.不同的multi_query的type
和multi_match得分不一样
因为multi_match有很多种type
best_fields:默认，取得分最高的作为对应的分数，最匹配模式,等同于dismax模式
GET /movie/_search
{
  "query":{
    "dis_max": { 
      "queries": [
        { "match": { "title":"basketball with cartoom aliens"}}, 
        { "match": { "overview":"basketball with cartoom aliens"}}  
      ]
    }
  }
}

然后使用explan看下 ((title:steve title:job) | (overview:steve overview:job))，打分规则
GET /movie/_validate/query?explain
{
  //"explain": true, 
  "query":{
    "multi_match":{
      "query":"steve job",
      "fields":["title","overview"],
      "operator": "or",
      "type":"best_fields"
    }
  }
}
以字段为单位分别计算分词的分数，然后取最好的一个,适用于最优字段匹配。


将其他因素以0.3的倍数考虑进去
GET /movie/_search
{
  "query":{
    "dis_max": { 
      "queries": [
        { "match": { "title":"basketball with cartoom aliens"}}, 
        { "match": { "overview":"basketball with cartoom aliens"}}  
      ],
      "tie_breaker": 0.3
    }
  }
}




most_fields:取命中的分值相加作为分数，同should match模式，加权共同影响模式

然后使用explain看下 ((title:steve title:job) | (overview:steve overview:job))~1.0，打分规则
GET /movie/_validate/query?explain
{
  //"explain": true, 
  "query":{
    "multi_match":{
      "query":"steve job",
      "fields":["title","overview"],
      "operator": "or",
      "type":"most_fields"
    }
  }
}
以字段为单位分别计算分词的分数，然后加在一起，适用于都有影响的匹配



cross_fields:以分词为单位计算栏位总分
然后使用explain看下 blended(terms:[title:steve, overview:steve]) blended(terms:[title:job, overview:job])，打分规则
GET /movie/_validate/query?explain
{
  //"explain": true, 
  "query":{
    "multi_match":{
      "query":"steve job",
      "fields":["title","overview"],
      "operator": "or",
      "type":"most_fields"
    }
  }
}
以词为单位，分别用词去不同的字段内取内容，拿高的分数后与其他词的分数相加，适用于词导向的匹配



GET /forum/article/_search
{
  "query": {
    "multi_match": {
      "query": "Peter Smith",
      "type": "cross_fields", 
      "operator": "or",
      "fields": ["author_first_name", "author_last_name"]
    }
  }
}
//要求Peter必须在author_first_name或author_last_name中出现
//要求Smith必须在author_first_name或author_last_name中出现

//原来most_fiels，可能像Smith //Williams也可能会出现，因为most_fields要求只是任何一个field匹配了就可以，匹配的field越多，分数越高

GET /movie/_search
{
  "explain": true, 
  "query":{
    "multi_match":{
      "query":"steve job",
      "fields":["title","overview"],
      "operator": "or",
      "type":"cross_fields"
    }
  }
}
看一下不同的评分规则



3.query string
方便的利用AND(+) OR(|) NOT(-)
GET /movie/_search
{
  "query":{
    "query_string":{
      "fields":["title"],
      "query":"steve AND jobs"
      
    }
  }
}



过滤查询

filter过滤查询

单条件过滤
￼GET /movie/_search
{
  "query":{
    "bool":{
      "filter":{
          "term":{"title":"steve"}
      }
    }
  }
}
多条件过滤
GET /movie/_search
{
  "query":{
    "bool":{
      "filter":[
        {"term":{"title":"steve"}},
        {"term":{"cast.name":"gaspard"}},
        {"range": { "release_date": { "lte": "2015/01/01" }}},
        {"range": { "popularity": { "gte": "25" }}}
        ]
    }
  },
  "sort":[
    {"popularity":{"order":"desc"}}
  ]
}

带match打分的的filter
GET /movie/_search
{
  "query":{
    "bool":{
      "must": [
        { "match": { "title":   "Search"        }}, 
        { "match": { "tagline": "Elasticsearch" }}  
      ],
      "filter":[
￼        {"term":{"title":"steve"}},
        {"term":{"cast.name":"gaspard"}},
        {"range": { "release_date": { "lte": "2015/01/01" }}},
        {"range": { "popularity": { "gte": "25" }}}
        ]
    }
  }
}

返回0结果

GET /movie/_search
{
  "query":{
    "bool":{
      "should": [
        { "match": { "title":   "Search"        }}, 
        { "match": { "tagline": "Elasticsearch" }}  
      ],
      "filter":[
        {"term":{"title":"steve"}},
        {"term":{"cast.name":"gaspard"}},
        {"range": { "release_date": { "lte": "2015/01/01" }}},
        {"range": { "popularity": { "gte": "25" }}}
        ]
    }
  }
}

有结果，但是返回score为0，因为bool中若有filter的话，即便should都不满足，只是返回为0分而已
修改为
GET /movie/_search
{
  "query":{
    "bool":{
      "should": [
        { "match": { "title":   "life"        }}, 
￼        { "match": { "tagline": "Elasticsearch" }}  
      ],
      "filter":[
        {"term":{"title":"steve"}},
        {"term":{"cast.name":"gaspard"}},
        {"range": { "release_date": { "lte": "2015/01/01" }}},
        {"range": { "popularity": { "gte": "25" }}}
        ]
    }
  }
}
可以看到分数



function score自定义打分

```json
GET /movie/_search
{
  "query":{
    "function_score": {
      //原始查询得到oldscore
      "query": {      
        "multi_match":{
        "query":"steve job",
        "fields":["title","overview"],
        "operator": "or",
        "type":"most_fields"
      }
    },
    "functions": [
      {"field_value_factor": {
          "field": "popularity",   //对应要处理的字段
          "modifier": "log2p",    //将字段值+2后，计算对数
          "factor": 10    //字段预处理*10
        }
      }
    ], 

    "score_mode": "sum",   //不同的field value之间的得分相加
    "boost_mode": "sum"    //最后在与old value相加
  }
}
}
```