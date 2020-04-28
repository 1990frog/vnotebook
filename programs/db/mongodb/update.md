[TOC]

+ db.collection.update()
+ db.collection.findAndModify()
+ db.collection.save()

# update
`db.<collection>.update(<query>,<update>,<options>)`
+ <query>文档定义了更新操作时筛选文档的条件
+ <update>文档提供了更新内容
+ <option>文档声明了一些更新操作的参数

如果<update>文档不包含任何更新操作符，db.collection.update()将会使用<update>文档直接替换集合中符合<query>文档筛选条件的文档

```json
> db.test.update({age:22},{name:"lilei",age:22})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

注意：
文档主键_id是不可以更改的
在使用<update>文档替换整篇被更新文档时，只有第一篇符合<query>的文档筛选条件的文档会被更新
不使用更新操作符会替换掉全部文档

# 文档操作符
+ $set 更新或新增字段
+ $unset 删除字段
+ $rename 重命名字段
+ $inc 加减字端值
+ $mul 相乘字段值
+ $min 比较减小字段值
+ $max 比较增大字段值

```json
> db.test.update({name:"cai"},{$set:{values:[1,2,3]}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16, "values" : [ 1, 2, 3 ] }
```
set更新内嵌文档
```json
> db.test.update({name:"cai"},{$set:{"a.b":2}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16, "values" : [ 1, 2, 3 ], "a" : { "b" : 2 } }
```
使用dot操作符嵌套，要用引号将整个字段括起来

更新或新增数组内的元素
```json
> db.test.update({name:"cai"},{$set:{"values.0":9}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16, "values" : [ 9, 2, 3 ], "a" : { "b" : 2 } }
```

$unset 如果命令中的字段根本不存在，那么文档内容将保持不变
使用$unset命令删除数组字段中的某一个元素时，这个元素不会被删除，只会被赋值以null值，而数组的长度不会改变

如果$rename命令要重命名的字段并不存在，那么文档内容不会被改变
```json
> db.test.update({values:{$exists:1}},{$rename:{"name":"myname"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "age" : 16, "values" : [ 9, 2, 3 ], "a" : { "b" : 2 }, "myname" : "cai" }
```
当$rename命令中的新字段存在的时候，\$rename命令会先\$unset新旧字段，然后再$set新字段

更改字段在文档中的位置
```json
db.accounts.update(
    {name:"karen"},
    {$rename:
        {
            "info.branch":"branch",
            "balance":"info.balance"
        }
    }
)
```

数组内嵌元素不能使用$rename移到外面，已不能移到数组里面

$rename命令中的就字段和新字段都不可以指向数组元素
这一点和之前介绍过的$set和\$unset命令不同

{$inc:{<field1>:<amount1>,...}}
{$mul:{<field1>:<amount1>,...}}
更新字段值

```json
> db.test.update({values:{$exists:1}},{$inc:{age:1}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "age" : 17, "values" : [ 9, 2, 3 ], "a" : { "b" : 2 }, "myname" : "cai" }
```

$inc和\$mul命令只能应用在数字字段上

如果被更新的字段不存在，$inc会创建字段，并且将字段值设为命令中的增减值，而\$mul会创建字段，但是字段值设为0

{$min:{<field1>:<amount1>,...}}
{$max:{<field1>:<amount1>,...}}

```json
> db.test.update({values:{$exists:1}},{$min:{age:1}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "age" : 1, "values" : [ 9, 2, 3 ], "a" : { "b" : 2 }, "myname" : "cai" }
```

如果被更新的字段不存在，$min和\$max命令会创建字段，并且将字段值设为命令中的更新值

![](_v_images/20200426233619956_930530089.png)

更新操作符：
数组更新操作符
$addToSet 向数组中增添元素
$pop 从数组中移除元素
$pull 从数组中有选择性地移除元素
$pullAll 从数组中有选择性地移除元素
$push 向数组中添加元素

```json
> db.test.find()
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "age" : 16 }
> db.test.update({name:"cai"},{$set:{list:[1,2,3]}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "age" : 16, "list" : [ 1, 2, 3 ] }
> db.test.update({name:"cai"},{$addToSet:{list:4}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "age" : 16, "list" : [ 1, 2, 3, 4 ] }
```
如果要插入的值已经存在数组字段中，则$addToSet不会再添加重复值
注意一下，使用$addToSet插入数组和文档时，插入值中的字段顺序也和已有值重复的时候，才被算作重复值被忽略

```json
> db.test.update({name:"cai"},{$addToSet:{contact:["contact1","contact2"]}})
```
更新之后，以内嵌数组添加

$addToSet会将数组插入被更新的数组字段中，称为内嵌数组
如果想要将多个元素直接添加到数组字段中，则需要使用$each操作符

```json
> db.test.update({name:"cai"},{$addToSet:{list:{$each:[5,6]}}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "age" : 16, "list" : [ 1, 2, 3, 4, 5, 6 ] }
```

$pop:1,-1
只能删除第一个元素或最后一个元素
第一个元素：-1

将karen的账号文档复制为lawrence的账号文档
```json
> db.accounts.find(
    {name:"karen"},
    {_id:0}
).forEach(
    function(doc){
        var newDoc = doc;
        newDoc.name = "lawrence";
        db.accounts.insert(newDoc);
    }
)
```

```json
> db.accounts.update({name:"karen"},{$pull:{contact:{$regex:/hi/}}})

> db.test.update({name:"cai"},{$pull:{list:4}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "age" : 16, "list" : [ 1, 2, 3, 5, 6 ] }
```

{$pull:{<field1>:<value|condition>,...}}
{$pullAll:{<field1>:[<value1>,<value2>,...],...}}

$pullAll命令只会删除字段和字段排列顺序都完全匹配的文档元素
$pull命令会删去包含指定的文档和字段值的文档元素，字段排列顺序不需要完全匹配

更新操作符
{$push:{<field1>:<value1>,...}}
向数组字段中添加元素
$push和\$addToSet命令相似，但是$push命令的功能更强大
和$addToSet相似，\$push操作符也可以和$each搭配使用

使用$position操作符将元素插入到数组的指定位置
```json
> db.test.update({name:"cai"},{$push:{list:{$each:[7,8],$position:0}}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find()
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "age" : 16, "list" : [ 7, 8, 1, 2, 3, 5, 6 ] }
```

使用$sort对数组进行排序操作
```json
> db.test.update({name:"cai"},{$push:{list:{$each:[9],$sort:1}}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test.find().pretty()
{
        "_id" : ObjectId("5ea6633869db2a1d09c40ba4"),
        "name" : "cai",
        "age" : 16,
        "list" : [
                1,
                2,
                3,
                5,
                6,
                7,
                8,
                9
        ]
}
```
$sort操作符必须搭配\$each操作符与\$push操作符

$slice操作符截取数组的一部分
+从头开始数n个
-从尾开始数n个

$positopn,\$sort,\$slice可以一起使用
这三个操作符的执行顺序是：
$position
$sort
$slice

```json
db.collection.update(
    {<array>:<query selector>},
    {<update operator>:{"<array>.$":value}}
)
```
更新数组中的特定元素
$是数组中第一个符合筛选条件的数组元素的占位符

{<update operator>:{"<array>.$[]}":value}

db.<collection>.update(<query>,<update>,<options>)
<option>文档提供了update命令的更多选项

{multi:<boolean>}
更新多个文档
在默认情况下，即使筛选条件对应了多篇文档，update命令仍然只会更新一篇文档
注意：mongodb只能保证单个文档操作的原子性，不能保证多个文档操作的原子性
更新多个文档的操作虽然在单一线程中执行，但是线程在执行过程中可能被挂起，以便其他线程也有机会对数据进行操作

如果需要保证多个文档操作时的原子性，就需要使用mongodb4.0版本引入的事务功能进行操作

{upsert:<boolean>}
更新或者创建文档
在默认情况下，如果update命令中的筛选条件没有匹配任何文档，则不会进行任何操作
将upsert选项设置为true，如果update命令中的赛选条件没有匹配任何文档，则会创建新文档
不过，如果无法从筛选条件中推断出确定的字段值，新创建的文件将不会包含筛选条件涉及的字段

![](_v_images/20200428005521591_557980795.png)
