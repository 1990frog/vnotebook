# db.collection.method

find
db.collection.find(query:查询条件,projection:投影).skip(0).limit(0)
其中skip方法与limit方法无顺序要求
db.collection.findOne(query, projection)虽然类似于find（）方法，但findOne（）方法返回的是文档而不是游标。
Oracle游标会加锁，mongodb的不加
update
db.collection.update(
<query>,
<update>,
{
upsert: <boolean>,
multi: <boolean><!--true:updateMany,false:updateOne-->,
writeConcern: <document>,
collation: <document>,
arrayFilters: [ <filterdocument1>, ... ]
})

db.collection.updateMany(
<filter>,
<update>,
{
upsert: <boolean>,
writeConcern: <document>,
collation: <document>,
arrayFilters: [ <filterdocument1>, ... ]
})


db.collection.updateOne(
<filter>,
<update>,
{
upsert: <boolean>,
writeConcern: <document>,
collation: <document>,
arrayFilters: [ <filterdocument1>, ... ]
})
