[TOC]

# redisObject
![270476457-5d4838a27babd_articlex](_v_images/20191120143355846_1287757325.png)
```c
struct redisObject
{
    数据类型（type）[string,hash,list,set,sorted set];
    编码方式（encoding）[raw,int,ziplist,linkedlist,hashmap,intset];
    数据指针（ptr）;
    虚拟内存（vm）;
    其他信息;
};
```

# hash
哈希键值结构：key->[field,value]
特点
map的map
small的redis
field不能相同，value可以相同
hash分为两种：hashtable、ziplist，如果量达到一定就会使用ziplist压缩节省内存
hash缺点：不能对key设置过期时间

# list
RPUSH+RPOP=Stack
LPUSH+RPOP=Queue
LPUSH+LTRIM=Capped Collection
LPUSH+BRPOP=Message Queue
LPUSH+BRPOP=MQ

# set
SADD =Tagging
SPOP/SRANDMEMBER=Random item
SADD+SINTER=Social Graph
特点：
无序、无重复、集合间操作（交集、并集、差集）

注意：
集合实战：
给用户添加标签
```
sadd user:1:tags tag1 tag2 tag3
```
给用户添加标签
```
sadd tag1:users user1 user2
```

# zset
集合vs有序集合
集合：无重复元素、无序、element
有序集合：无重复元素、有序、element+score

基本操作：zadd,zrem,zcard,zincrby,zscore
范围操作：zrange,zrangebyscore,zcount,zremrangebyrank
集合操作：zunionstore,zinterstore
