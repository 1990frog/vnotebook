[TOC]

# 语法
```
GET /_search
{
  "query": {
    # 关键字
    "multi_match" : {
      # 检索字符串
      "query": "this is a test",
      # 检索字段数组
      # 匹配符*，匹配字段
      # ^num，评分加权
      "fields": [ "subject", "message","*name","value^3" ]
      # 评分策略
      "type":best_fields|most_fields|cross_fields
    }
  }
}
```

# type
|     策略      |                      介绍                      |
| ------------ | --------------------------------------------- |
| best_fields  | （默认）通过多个字段检索文档，但是使用评分最高字段的评分 |
| most_fields  | 最终评分使用所有字段评分加和                         |
| cross_fields | 以分词为单位计算栏位的总分，适用于词导向的匹配         |
| phrase       |                                               |
| phrase_prefix |                                               |
| bool_prefix   |                                               |



