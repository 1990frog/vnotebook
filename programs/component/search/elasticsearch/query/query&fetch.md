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


