[TOC]

主要分两类：
+ 单值分析，只输出一个分析结果
    1. min,max,avg,sum
    2. cardinality
+ 多值分析，输出多个分析结果
    1. stats，extended stats
    2. percentile，percentile rank
    3. top hits

# min
返回数值类字段的最小值
```json
GET index/_search
{
    # 不需要返回文档列表
    "size":0,
    "aggs":{
        "min_age":{
            # 关键词
            "min":{
                "field":"age"
            }
        }
    }
}
```
# max
返回数值类字段的最大值
```json
GET index/_search
{
    # 不需要返回文档列表
    "size":0,
    "aggs":{
        "max_age":{
            # 关键词
            "max":{
                "field":"age"
            }
        }
    }
}
```
# avg
返回数值类字段的平均值
```json
GET index/_search
{
    # 不需要返回文档列表
    "size":0,
    "aggs":{
        "avg_age":{
            # 关键词
            "avg":{
                "field":"age"
            }
        }
    }
}
```
# sum
返回数值类字段的总和
```json
GET index/_search
{
    # 不需要返回文档列表
    "size":0,
    "aggs":{
        "sum_age":{
            # 关键词
            "sum":{
                "field":"age"
            }
        }
    }
}
```
# 复合使用
```json
GET index/_search
{
    # 不需要返回文档列表
    "size":0,
    "aggs":{
        "sum_age":{
            # 关键词
            "sum":{
                "field":"age"
            }
        },"avg_age":{
            # 关键词
            "avg":{
                "field":"age"
            }
        },"max_age":{
            # 关键词
            "max":{
                "field":"age"
            }
        },"min_age":{
            # 关键词
            "min":{
                "field":"age"
            }
        }
    }
}
```
# Cardinality
Cardinality，意为集合的势，或者基数，是指不同数值的个数，类似sql中的distrinct count概念
```json
GET index/_search
{
    "size":0,
    "aggs":{
        "count_of_job":{
            "cardinality":{
                "field":"job.keyword"
            }
        }
    }
}
```
# stats
返回一系列数值类型的统计值，包含min、max、avg、sum和count
```json
GET index/_search
{
    "size":0,
    "aggs":{
        "stats_age":{
            "stats":{
                "field":"age"
            }
        }
    }
}
```
# extended stats
对stats的扩展，包含了更多的统计数据，如方差、标准差等
```json
GET index/_search
{
    "size":0,
    "aggs":{
        "stats_age":{
            "extended_stats":{
                "field":"age"
            }
        }
    }
}
```
# percentile
百分位数统计
```json
GET index/_search
{
    "size":0,
    "aggs":{
        "per_age":{
            "percentiles":{
                "field":"salary"
            }
        }
    }
}

GET index/_search
{
    "size":0,
    "aggs":{
        "per_age":{
            "percentiles":{
                "field":"salary",
                "percents":[
                    95,
                    99,
                    99.9
                ]
            }
        }
    }
}
```
# top hits
一般用于分桶后获取该通内最匹配的顶部文档列表，即详情数据
