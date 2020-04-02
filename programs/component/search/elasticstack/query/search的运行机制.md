[TOC]

# Search的运行机制
Search执行的时候实际分为两个步骤运作的
+ Query阶段
+ Fetch阶段

# Query阶段
假设有3个node
1. node3在接收到用户的search请求后，会先进行Query阶段（此时是Coordinating Node角色）
2. node3在6个主副分片中随机选择3个分片，发送search request
4. 被选中的3个分片会分别执行查询并排序，返回from+size个文档id和排序值
5. node3整合3个分片返回的from+size个文档id，根据排序值排序后选取from到from+size的文档id

# Fetch阶段



# Search的运行机制——相关性算分问题
+ 相关性算分在shard和shard间是相互独立的，也就意味着同一个term的IDF等值在不同的shard上是不同的。文档的相关性算分和它所处的shard相关
+ 在文档数量不多时，会导致相关性算分严重不准的情况发生

# Search的运行机制——相关性算分问题
解决思路有两个：
+ 一是设置分片数为1个，从根本上排除问题，在文档数量不多的时候可以考虑该方案，比如百万到千万级别的文档数量
+ 二是使用DFS Query-then-Fetch查询方式

DFS Query-then-Fetch是在拿到所有文档后再重新完整的计算一次相关性算法，消费更多的cpu和内存，执行性能也比较低下，一般不建议使用。使用方式如下：
```json
GET index/_search?search_type=dfs_query_then_fetch
{
    "query":
    {
        "match":
        {
            "name":"hello"
        }
    }
}
```
缺点：
不建议使用，内存撑爆

# 排序
es默认会采用相关性算分排序，用户可以通过设定sorting参数来自行设定排序规则
```json
GET index/_search
{
    "sort":
    {
        "birth":"desc"
    }
}
```
按多个字段排序
```json
GET index/_search
{
    "sort":[
        {
            "birth":"desc"
        },
        {
            "_score":"desc"
        },
        {
            "_doc":"desc"
        }
    ]
}
```
# text文档不可直接排序

text类型与keyword类型排序
fields

```json
GET index/_search
{
    "sort":[
        {
            "username.keyword":"desc"
        }
    ]
}
```
# 排序
+ 排序的过程实质是对字段原始内容排序的过程，这个过程中倒排索引无法发挥作用，需要用到正排索引，也就是通过文档id和字段可以快速得到字段原始内容
+ es对此提供了2种实现方式：
    fielddata默认禁用
    doc values默认启用，除了text类型

# Fielddata VS DocValues
|  对比   |                 FIeldData                 |          DocValues           |
| ------ | ----------------------------------------- | --------------------------- |
| 创建时机 | 搜索时即时创建                              | 索引时创建，与倒排索引创建时机一致 |
| 创建位置 | JVM Heap                                  | 磁盘                         |
| 优点    | 不会占用额外的磁盘资源                        | 不会占用Heap内存               |
| 缺点    | 文档过多时，即时创建会花过多时间，占用过多Heap内存 | 减慢索引的速度，占用额外的磁盘资源 |

# Fielddata
Fielddata默认是关闭的，可以通过如下api开启：
+ 此时字符串是按照分词后term排序，往往结果很难符合预期
+ 一般是在对分词在聚合分析的时候开启



# 分页与遍历
es提供了3种方式来解决分页与遍历的问题：
+ from/size
+ scroll
+ search_after

## from/size
最常用的分册方案：
+ from指明开始位置
+ size指明获取总数

```json
GET index/_search
{
    "from":1,
    "size":2
}
```

深度分页是一个经典的问题：在数据分片存储的情况下如何获取前1000个文档？
+ 获取从990~1000的文档时，会在每个分片上都先获取1000个文档，然后再由Coordinating Node聚合所有分片的结果后排序选取前1000个文档
+ 页数越深，处理文档越多，占用内存越多，耗时越长。尽量避免深度分页，es通过`index.max_result_window`限定最多到10000条数据

Google使用了避免深排序

# Scroll
遍历文档集api，以快照的方式来避免深度分页的问题：
+ 不能用来做实时搜索，因为数据不是实时的
+ 尽量不要使用复杂的sort条件，使用_doc最高效
+ 使用稍显复杂

## 第一步需要发起1个scroll search，如下所示
es在收到该请求后根据查询条件创建文档id合集的快照
```json
# 指定快照有效时间
GET index/_search?scroll=5m
{
    "size":1
}

{
"_scroll_id":xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
}
```
## 第二步调用scroll search的api，获取文档集合
 不断迭代调用直到返回hits.hits数组为空时停止

# 过多的scroll调用会占用大量内存，可以通过clear api删除过多的scroll快照
```json
DELETE /_search/scroll/_all
```

# search after
避免深度分页的性能问题，提供实时的下一页文档获取功能
+ 缺点是不能使用from参数，既不能指定页数
+ 只能下一页，不能上一页
+ 使用简单

## 第一步为正常的搜索，但要指定sort值，并保证值唯一
## 第二步为使用上一步最后一个文档的sort值进行查询


# 应用场景
|     类型     |              场景               |
| ------------ | ------------------------------ |
| from/size    | 需要实时获取顶部的文档，且需要自由翻页 |
| scroll       | 需要全部文档，如导出所有数据的功能    |
| search_after |         需要全部文档，不需要自由翻页                       |


