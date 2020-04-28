vnote_backup_file_826537664 /home/cai/Documents/vnotebook/programs/component/db/mongodb/CRUB.md
[TOC]

# 切换数据库
use [dbname]

# 查看数据库中的集合
show collections

# 创建
MongoDB提供了保存数据的方法一共有三个：
1. db.collection.insertOne()
2. db.collection.insertMany()
3. db.collection.insert()

`"acknowledged" : true`

差异：
+ insertOne,insertMany不支持explain命令
+ insert命令支持explain
## insertOne
将单个文档插入到一个集合
```json
> db.test.insertOne({name:"cai",age:16});
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5ea30ce206298951fd1fcd41")
}
> db.test.find({name:"cai"})
{ "_id" : ObjectId("5ea30cf606298951fd1fcd42"), "name" : "cai", "age" : 16 }
```
## insertMany
将文档数组插入到一个集合
```json
> db.test.insertMany([{name:"cai",age:16},{name:"hui",age:18}]);
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("5ea30d9006298951fd1fcd43"),
                ObjectId("5ea30d9006298951fd1fcd44")
        ]
}
> db.test.find()
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16 }
{ "_id" : ObjectId("5ea30cf606298951fd1fcd42"), "name" : "cai", "age" : 16 }
{ "_id" : ObjectId("5ea30d9006298951fd1fcd43"), "name" : "cai", "age" : 16 }
{ "_id" : ObjectId("5ea30d9006298951fd1fcd44"), "name" : "hui", "age" : 18 }
```
ordered参数用来决定是否要按顺序写入这些文档，默认true
## insert
将单个或多个文档到一个集合。要插入一个单一的文件，传递文档;插入多个文件，传递文档数组
```json
> db.test.insert([{name:"cai1",age:16},{name:"hui1",age:18}]);
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 2,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})
> db.test.find()
{ "_id" : ObjectId("5ea30df906298951fd1fcd45"), "name" : "cai1", "age" : 16 }
{ "_id" : ObjectId("5ea30df906298951fd1fcd46"), "name" : "hui1", "age" : 18 }
```

## save
db.collection.save命令处理一个新文档的时候，它会调用db.collection.insert()命令
```json
> db.test.save([{name:"cai2",age:16},{name:"hui2",age:18}])
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 2,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})
> db.test.find()
{ "_id" : ObjectId("5ea30ec106298951fd1fcd4c"), "name" : "cai2", "age" : 16 }
{ "_id" : ObjectId("5ea30ec106298951fd1fcd4d"), "name" : "hui2", "age" : 18 }
```
# 读取
## find
语法：
```
db.<collection>.find(<query>,<projection>)
<query>文档定义了读取操作时筛选文档的条件
<projection>文档定义了对读取结果进行的投射
```

### 读取全部
```json
> db.test.find()
```
更清楚的显示文档
```json
> db.test.find().pretty()
```

### 匹配查询
db.test.find({name:"xxx",...})

### 比较操作符
+ $eq 匹配字段值相等的文档
+ $ne 匹配字段值不等的文档
+ $gt 匹配字段值大于查询值的文档
+ $gte 匹配字段值大于或等于查询值的文档
+ $lt 匹配字段值小于查询值的文档
+ $lte 匹配字段值小于或等于查询值的文档

```json
> db.test.find({age:{$gt:16}})
{ "_id" : ObjectId("5ea30d9006298951fd1fcd44"), "name" : "hui", "age" : 18 }
{ "_id" : ObjectId("5ea30df906298951fd1fcd46"), "name" : "hui1", "age" : 18 }
{ "_id" : ObjectId("5ea30e5106298951fd1fcd48"), "name" : "hui1", "age" : 18 }
{ "_id" : ObjectId("5ea30e5406298951fd1fcd4a"), "name" : "hui1", "age" : 18 }
{ "_id" : ObjectId("5ea30ec106298951fd1fcd4d"), "name" : "hui2", "age" : 18 }
```

+ $in 匹配字段值与任一查询值相等的文档
+ $nin 匹配字段值与任何查询值都不等的文档

```json
> db.test.find({name:{$in:["cai","hui"]}})
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16 }
{ "_id" : ObjectId("5ea30cf606298951fd1fcd42"), "name" : "cai", "age" : 16 }
{ "_id" : ObjectId("5ea30d9006298951fd1fcd43"), "name" : "cai", "age" : 16 }
{ "_id" : ObjectId("5ea30d9006298951fd1fcd44"), "name" : "hui", "age" : 18 }
```

### 逻辑运算符
+ $not 匹配筛选条件不成立的文档
+ $and 匹配多个筛选条件全部成立的文档
+ $or 匹配至少一个筛选条件成立的文档
+ $nor 匹配多个筛选条件全部不成立的文档

```json
> db.test.find({age:{$not:{$gt:20}}})
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16 }

> db.test.find({$and:[{age:{$gt:5}},{age:{$lt:20}}]})
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16 }

> db.test.find({age:{$gt:5,$lt:20}})
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16 }
```

### 字段操作符
+ $exists 匹配包含查询字段的文档
+ $type 匹配字段类型复合查询值的文档

### 数组操作符
+ $all 匹配数组字段中包含所有查询值的文档
+ $elemMatch 匹配数组字段中至少存在一个值满足筛选条件的文档

### 游标
var myCursor = db.test.find();
myCursor[1]
游标遍历完所有的文档之后，或者在10分钟之后，游标会自动关闭
可以用noCursorTimeout()函数来保持游标一直有效
var myCursor = db.test.find().noCursorTimeout()
在这之后，在不遍历游标的情况下，需要主动关闭游标
myCursor.close()

游标函数：
cursor.hasNext()
cursor.next()
cursor.forEach()
cursor.limit()
cursor.skip()
cursor.count()
cursor.sort()