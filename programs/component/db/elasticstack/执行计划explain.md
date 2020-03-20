# 执行计划
## 单字段
```json
# 执行计划
GET /movie/_search
{
  "explain": true,
  "query": {
    "match": {
      "title": "Basketball"
    }
  }
}
```
结果分析
```json
"_explanation":
{
  "value" : 8.280251,
  "description" : "weight(title:basketbal in 2390) [PerFieldSimilarity], result of:",
  "details" : [
    {
      "value" : 8.280251,
      "description" : "score(freq=1.0), computed as boost * idf * tf from:",
      "details" : [
        {
          "value" : 2.2,
          "description" : "boost",
          "details" : [ ]
        },
        {
          "value" : 8.00659,
          "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
          "details" : [
            {
              "value" : 1,
              "description" : "n, number of documents containing term",
              "details" : [ ]
            },
            {
              "value" : 4500,
              "description" : "N, total number of documents with field",
              "details" : [ ]
            }
          ]
        },
        {
          "value" : 0.47008157,
          "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
          "details" : [
            {
              "value" : 1.0,
              "description" : "freq, occurrences of term within document",
              "details" : [ ]
            },
            {
              "value" : 1.2,
              "description" : "k1, term saturation parameter",
              "details" : [ ]
            },
            {
              "value" : 0.75,
              "description" : "b, length normalization parameter",
              "details" : [ ]
            },
            {
              "value" : 2.0,
              "description" : "dl, length of field",
              "details" : [ ]
            },
            {
              "value" : 2.1757777,
              "description" : "avgdl, average length of field",
              "details" : [ ]
            }
          ]
        }
      ]
    }
  ]
}
```
## 多字段
```json
GET /movie/_search
{
  "explain": true,
  "query": {
    "multi_match": {
      "query": "basketball with cartoom aliens",
      "fields": ["title","overview"]
    }
  }
}
```