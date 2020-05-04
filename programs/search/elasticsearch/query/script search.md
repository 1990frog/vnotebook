# 脚本字段
用例：订单中有不同的汇率，需要结合汇率对订单价格进行
```json
GET index/_search
{
    "script_fields":{
        "new_field":{
            "script":{
                "lang":"painless",
                "source":"doc['order_date'].value+'hello'"
            }
        }
    },
    "from":10,
    "size":5,
    "query":{
        "match_all":{}
    }
}
```