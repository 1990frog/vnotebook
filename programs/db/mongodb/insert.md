[TOC]

MongoDB提供了保存数据的方法一共有三个：
1. db.collection.insertOne()
2. db.collection.insertMany()
3. db.collection.insert()

# insertOne
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
# insertMany
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
{ "_id" : ObjectId("5ea30d9006298951fd1fcd44"), "name" : "hui", "age" : 18 }
```
ordered参数用来决定是否要按顺序写入这些文档，默认true
# insert
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
# save
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