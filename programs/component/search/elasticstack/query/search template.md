[TOC]

参数化搜索：
```json
GET _search/template
{
    "source" : {
      "query": { "match" : { "{{my_field}}" : "{{my_value}}" } },
      "size" : "{{my_size}}"
    },
    "params" : {
        "my_field" : "message",
        "my_value" : "some message",
        "my_size" : 5
    }
}
```

模板化查询
```json
POST _scripts/mytp
{
  "script":{
    "lang":"mustache",
    "source": {
      "_source":["title","overview"],
      "size":20,
      "query":{
        "multi_match":{
          "query":"{{q}}",
          "fields":["title","overview"]
        }
      }
    }
  }
}

DELETE _scripts/mytp

POST movie/_search/template
{
  "id": "mytp",
  "params": {
    "q":"basketball"
  }
}
```