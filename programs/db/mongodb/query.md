[TOC]

# 语法
`db.<collection>.find(<query>,<projection>)`
+ <query>文档定义了读取操作时筛选文档的条件
+ <projection>文档定义了对读取结果进行的投射

读取全部：
`db.test.find()`
美化：
`db.test.find().pretty()`

# 匹配查询
查询条件，json格式
```json
>db.test.find({name:"xxx",...})
```

# 比较操作符
`db.<collection>.find({主语(field):{谓语(比较运算符):宾语}})`
+ $eq 匹配字段值相等的文档
+ $ne 匹配字段值不等的文档
+ $gt 匹配字段值大于查询值的文档
+ $gte 匹配字段值大于或等于查询值的文档
+ $lt 匹配字段值小于查询值的文档
+ $lte 匹配字段值小于或等于查询值的文档
+ $in 匹配字段值与任一查询值相等的文档
+ $nin 匹配字段值与任何查询值都不等的文档

## $eq,\$ne 相等，不等
```json
> db.test.find({age:{$eq:16}})
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ], "age" : 16 }
```
## $gt,\$gte,\$lt,\$lte 边界
```json
> db.test.find({age:{$gte:15,$lte:20}})
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ], "age" : 16 }
```
## $in,\$nin 包含，非包含
```json
> db.test.find({list:{$in:[1,2,3]}})
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ] }
> db.test.find()
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ] }
> db.test.find({list:{$in:[1,2,3]}})
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ] }
```
$in 匹配一个就返回true
$nin 必须都满足才会返回true

# 逻辑运算符
+ $not 匹配筛选条件不成立的文档，`主语+谓词`
+ $and 匹配多个筛选条件全部成立的文档，`连接词+[...]`
+ $or 匹配至少一个筛选条件成立的文档，`连接词+[...]`
+ $nor 匹配多个筛选条件全部不成立的文档，`连接词+[...]`

## $not 匹配筛选条件不成立的文档
主+谓
要搭配比较运算符一起使用
```json
> db.test.find({age:{$not:{$eq:20}}})
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ], "age" : 16 }
```
## $and 匹配多个筛选条件全部成立的文档
```json
> db.test.find({$and:[{age:16},{name:"cai"}]})
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ], "age" : 16 }
```
## $or 匹配至少一个筛选条件成立的文档
```json
> db.test.find({$or:[{age:16},{name:"cai_1"}]})
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ], "age" : 16 }
```
## $nor 匹配多个筛选条件全部不成立的文档
```json
> db.test.find({$nor:[{age:19},{name:"cai_1"}]})
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ], "age" : 16 }
```

# 字段操作符
+ $exists 匹配包含查询字段的文档
+ $type 匹配字段类型复合查询值的文档

## $exists 匹配包含查询字段的文档
`主语（field）+谓词（关键字）`结构
0：不存在
1：存在
```json
> db.test.find({age:{$exists:0}})
> db.test.find({age:{$exists:1}})
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ], "age" : 16 }
```

## $type 匹配字段类型复合查询值的文档
type常用类型

|    类型     | 数字 |
| ----------- | --- |
| Double      | 1   |
| String      | 2   |
| Object      | 3   |
| Array       | 4    |
| Binary data | 5    |
| Object id   | 7    |
| Boolean     | 8    |
| Date        | 9    |
| Null        | 10  |

# 数组操作符
+ $all 匹配数组字段中包含所有查询值的文档
+ $elemMatch 匹配数组字段中至少存在一个值满足筛选条件的文档

## $all 匹配数组字段中包含所有查询值的文档
```json
> db.test.find({},{_id:0,list:1})
{ "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ] }
> db.test.find({list:{$all:[1,2,3]}})
{ "_id" : ObjectId("5ea6633869db2a1d09c40ba4"), "name" : "cai", "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ], "age" : 16 }
> db.test.find({list:{$all:[1,2,3,99]}})
null
```
## $elemMatch 匹配数组字段中至少存在一个值满足筛选条件的文档
如果只有一个查询条件就没必要使用 $elemMatch
```json
> db.test.find({},{_id:0,list:1})
{ "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ] }
> db.test.find({list:{$elemMatch:{$eq:1,$eq:9}}},{_id:0,list:1})
{ "list" : [ 1, 2, 3, 5, 6, 7, 8, 9 ] }
```

# 游标
```js
var cursor = db.test.find();
cursor[1]
```
游标遍历完所有的文档之后，或者在10分钟之后，游标会自动关闭
可以用noCursorTimeout()函数来保持游标一直有效
```js
var cursor = db.test.find().noCursorTimeout()
```
在这之后，在不遍历游标的情况下，需要主动关闭游标
```js
cursor.close()
```

# 游标函数
+ cursor.hasNext()
+ cursor.next()
+ cursor.forEach()
+ cursor.limit()
+ cursor.skip()
+ cursor.count()
+ cursor.sort()

## next,hasNext,forEach 遍历

## sort,skip,count,limit 数量操作

## 函数存在优先级

```json
> db.test.find({age:16}).count()
7
> db.test.find({age:16}).limit(1).count()
7
> db.test.find({age:16}).limit(1).count(true)
1
```
count(<applySkipLimit>默认false)

当数据库分布式结构较为复杂时，元数据中的文档数量可能不准确
在这种情况下，应该避免应用不提供赛选条件的cursor.count()函数，而使用聚合管道来计算文档数量

```json
> db.test.find().sort({age:1})
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16 }
{ "_id" : ObjectId("5ea30d9006298951fd1fcd44"), "name" : "hui", "age" : 18 }
> db.test.find().sort({age:-1})
{ "_id" : ObjectId("5ea30d9006298951fd1fcd44"), "name" : "hui", "age" : 18 }
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16 }
```
1表示由小到大正向排序，-1表示逆向排序

db.collection.limit().skip()
db.collection.skip().limit()
skip永远先于limit执行

sort永远先于skip与limit

# 投影
db.collection.find(<query>,<projection>)
不使用投影时，db.collection.find()返回符合条件的完整文档
而使用投影可以有选择性的返回文档中的部分字段
{field: inclusion}
1表示返回字段，0表示不返回字段
```json
> db.test.find()
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai", "age" : 16 }
> db.test.find({},{name:1})
{ "_id" : ObjectId("5ea30ce206298951fd1fcd41"), "name" : "cai" }
> db.test.find({},{name:1,age:0})
Error: error: {
        "ok" : 0,
        "errmsg" : "Projection cannot have a mix of inclusion and exclusion.",
        "code" : 2,
        "codeName" : "BadValue"
}
```
除了文档主键之外，我们不可用在投影文档中混合使用包含和不包含这两种投影操作
要么在投影文档中列出所有应该包含的字段，要么列出所有不应该包含的字段

## 投影操作符
$slice 操作符可以返回数组字段中的部分元素,1：返回第一个元素，-1：返回倒数第一个元素，-2：返回后面2个元素
$elemMatch和\$可以返回数组字段中满足筛选条件的第一个元素

```json
db.accounts.find({contact:{$gt:"Alabama"}},{_id:0,name:1,"contact.$":1})
```

`select 投影 from db.collection`