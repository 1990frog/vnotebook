# 查询选择器与投影选择器（Query and Projection Operators）

查询选择器（Query Selectors）

Comparison：比较运算符
$eq (equal)
含义：
匹配指定值
Matchs values that are equal to a specified value
语法： 
{<field>: { $eq: <value> } }
示例：
1.匹配单个值
db.inventory.find( { qty: { $eq: 20 } } )
2.匹配数组
db.collection.find({filed:{$eq:[1,2]}})
3.匹配子对象
db.collection.find({obj.filed:{$eq:1}})

$ne (not equal)
含义：
匹配所有不等于指定值的值
Matches all values that are not equal to a specified value
语法: 
{<field> : {$ne: value} }
示例：
1.单个字段
db.inventory.find( { qty: { $ne: 20 } } )
2.数组
db.collection.find({array:{$ne:[1,2]}})
3.子元素
db.inventory.update( { "carrier.state": { $ne: "NY" } }, { $set: { qty: 20 } } )

$gt (greater)
含义：
匹配比指定值大的值
Matchs values that are greater than a specified value
语法：
{<field> : {$gt: value} }
示例：
1.只能比较单个值
db.inventory.find( { qty: { $gt: 20 } } )
db.inventory.update( { "carrier.fee": { $gt: 2 } }, { $set: { price: 9.99 } } )

$gte (greater equal)
含义：
匹配大于或等于指定值的值
Matchs values that are greater than or equal to a specified value
语法: 
{<field> : {$gte: value} }
示例：
1.比较单个值
db.inventory.find( { qty: { $gte: 20 } } )
db.inventory.update( { "carrier.fee": { $gte: 2 } }, { $set: { price: 9.99 } } )

$lt 
含义：
匹配小于指定值的值
Matches values that less than a specified value
语法 :
{field: {$lt: value} }
示例：
1.比较单个值
db.inventory.find( { qty: { $lt: 20 } } )
db.inventory.update( { "carrier.fee": { $lt: 20 } }, { $set: { price: 9.99 } } )

$lte
含义：
匹配小于或等于指定值的值
Matches values that less than and equal to a specified value
语法 : 
{<field> : { $lte: value} }
示例：
1.比较单个值
db.inventory.find( { qty: { $lte: 20 } } )
db.inventory.update( { "carrier.fee": { $lte: 5 } }, { $set: { price: 9.99 } } )

$in (in)
含义：
匹配数组中指定的任何值（）
Matches any of the values specified in an array
语法 : 
{<field>: { $in: [<value1>, <value2>, ... <valueN> ] } }
注意：
1.$in与$or差别：$in只能用于单字段，$or可用于多字段
2.$in与$all在数组中的差别：使用$in时，指定的数组中满足一个元素（子集）即返回true，而$all（全集）需要全部满足
示例：
1.在字段中使用
db.inventory.find( { qty: { $in: [ 5, 15 ] } } )
2.在数组中使用
db.inventory.update( { tags: { $in: ["appliances", "school"] } },{ $set: { sale:true } })
3.在正则中使用
db.inventory.find( { tags: { $in: [ /^be/, /^st/ ] } } )


$nin (not in)
含义：
不匹配数组中指定的值
Matches none of the values specified in an array
语法:
{<field> : { $nin: [ <value1>, <value2> ... <valueN> ]} }
示例：
1.在字段中使用
db.inventory.find( { qty: { $nin: [ 5, 15 ] } } )
db.inventory.update( { tags: { $nin: [ "appliances", "school" ] } }, { $set: { sale: false } } )
2.在数组中使用
db.collections.find({array:{$nin:[1,2]}}) 取差集

总结：都是直接接值操作：{filed:{$comparison:[]|value}}
$eq对应$ne
$lt,lte对应$gt,$gte
$in对应$nin
Logical：逻辑运算符
$or
含义：
使用逻辑运算符or连接查询子句返回符合任一子句条件的所有文档。
Joins query clauses with a logical OR returns all documents that match the conditions of either clause.
语法：
{ $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }
示例：
db.inventory.find( { $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] } )

$nor
含义：
返回不符合全部条件的数据
语法：
{ $nor: [ { <expression1> }, { <expression2> }, ... { <expressionN> } ] }
示例：
db.inventory.find( { $nor: [ { price: 1.99 }, { sale: true } ] } )
db.inventory.find( { $nor: [ { price: 1.99 }, { qty: { $lt: 20 } }, { sale: true } ] } )
db.inventory.find( { $nor: [ { price: 1.99 }, { price: { $exists: false } },{ sale: true }, { sale: { $exists: false } } ] } )

$and
含义：
使用逻辑连接符$and将返回符合多个子句条件的所有文档
Joins query clauses with a logical AND returns all Documents that match the conditions of both clauses
语法：
{ $and: [ { <expression1> }, { <expression2> } , ... , {<expressionN> } ] }
示例：
1.$and多个表达式查询指定相同的字段
db.inventory.find( { $and: [ { price: { $ne: 1.99 } }, { price: { $exists: true } } ] } )
等同于
db.inventory.find( { price: { $ne: 1.99, $exists: true } } )
2.$and多个表达式查询指定相同的运算符
db.inventory.find( {$and : [ { $or : [ { price : 0.99 }, { price : 1.99 } ] },{ $or : [ { sale : true }, { qty : { $lt : 20 } } ] } ]} )

$not
含义：
$not performs a logical NOT operation on the specified <operator-expression> and selects the documents that do not match the <operator-expression>.This includes documents that do not contain the field.
语法：
{ field: { $not: { <operator-expression> } } }
注意：
$not操作符只影响其他操作，不能单独用于字段和文档
示例：
1.不可直接作用于字面量
db.inventory.find( { price: { $not: { $gt: 1.99 } } } )
db.inventory.find( { item: { $not: /^p.*/ } } )

$or对应$nor
$and对应$not
Element：元素运算符
$exists
含义：
匹配具有指定字段的文档。
Matches documents that have the specified field.
语法：
{ field: { $exists: <boolean> } }
注意：$exists用于匹配字段本身是否存在true|fasle不能应用于字段值
示例：
db.inventory.find( { qty: { $exists: true, $nin: [ 5, 15 ] } } )
db.records.find( { a: { $exists: true } } )
db.records.find( { b: { $exists: false } } 

$type
含义：
如果字段是指定的类型，则选择文档。
Selects documents if a field is of the specified type.
语法：
{field:{$type:<BSON type>}}
Double 1,String 2…..

$exists判断字段是否存在，$type判断字段类型。
都是作用于字段本身
Evaluation ：评估选择器
$expr
含义：
允许在查询语言中使用聚合表达式。
Allows use of aggregation expressions within the query language.
语法：
{ $expr: { <expression> } }
注意：
$expr可以构建查询表达式，用于在$match阶段比较同一文档字段。
$expr can build query expressions that compare fields from the same document in a $match stage.
如果$match阶段是$lookup阶段的一部分，$expr可以使用let变量比较字段。
If the $match stage is part of a $lookup stage,$expr can compare fields using let variables.See Specify Multiple Join Conditions with $lookup for an example.
示例：
1.比较同一个文档的两个字段
db.monthlyBudget.insertMany([
{ "_id" : 1, "category" : "food", "budget": 400, "spent": 450 },
{ "_id" : 2, "category" : "drinks", "budget": 100, "spent": 150 },
{ "_id" : 3, "category" : "clothes", "budget": 100, "spent": 50 },
{ "_id" : 4, "category" : "misc", "budget": 500, "spent": 300 },
{ "_id" : 5, "category" : "travel", "budget": 200, "spent": 650 }])
db.monthlyBudget.find( { $expr: { $gt: [ "$spent" , "$budget" ] } } )
2.使用$expr和条件语句
db.supplies.insertMany([
{ "_id" : 1, "item" : "binder", "qty": 100 , "price": 12 },
{ "_id" : 2, "item" : "notebook", "qty": 200 , "price": 8 },
{ "_id" : 3, "item" : "pencil", "qty": 50 , "price": 6 },
{ "_id" : 4, "item" : "eraser", "qty": 150 , "price": 3 }
])
db.supplies.find( {
    $expr: {
       $lt:[ {
          $cond: {
             if: { $gte: ["$qty", 100] },
             then: { $divide: ["$price", 2] },
             else: { $divide: ["$price", 4] }
           }
       },
       5 ] }
} )
db.collection.find与aggregate阶段match差异？

$mod
含义：
选择满足取余条件的文档
语法：
{ field: { $mod: [ divisor, remainder ] } }
divisor:除数
remainder:余数
示例：
db.inventory.find( { qty: { $mod: [ 4, 2 ] } } )

$text
含义：
$text在一个存在索引的索引字段上执行文本搜索
$text performs a text search on the content of the fields indexed with a text index.
语法：
{
  $text:
    {
      $search: <string>,
      $language: <string>,不支持中文？
      $caseSensitive: <boolean>,
      $diacriticSensitive: <boolean>
    }
}
注意：
1.只能使用于创建索引的字段上
2.-为排除操作符,'\'包围短语
3.language：语言，不支持中文
4.caseSensitive：区分大小写
5.diacriticSensitive：是否区分读音
示例：
1.查询一个简单的单词
db.articles.find( { $text: { $search: "coffee" } } )
2.查询多个单词
db.articles.find({$text:{$search:”bake coffee cake”}})
等同于查询”bake” or “coffee” or “cake”
3.查询一个短语
db.articles.find({$text:{$search: “\”coffee shop\””}})
4.排除一项
db.articles.find({$text:{$search:”coffee -shop”}})

$where
含义：
使用$where操作符将包含JAVASCRIPT表达式的字符串或完整的JavaScript函数传递给查询系统。$where提供了更大的灵活性，但要求数据库处理集合中每个文档的JavaScript表达式或函数。使用this或obj在JavaScript表达式或函数中引用文档。
Use the $where operator to pass either a string containing a JavaScript expression or a full JavaScript function to the query system.The $where provides greater flexibility, but requires that the database processes the JavaScript expression or function for each document in the collection.Reference the document in the JavaScript expression or function using either this or obj.
示例：
db.users.insertMany([{
   _id: 12378,
   name: "Steve",
   username: "steveisawesome",
   first_login: "2017-01-01"
},
{
   _id: 2,
   name: "Anya",
   username: "anya",
   first_login: "2001-02-02"
}
])
db.foo.find( { $where: function() {
   return (hex_md5(this.name) == "9b53e667f30cd329dca1ec9e6a83e994")
} } );


$jsonSchema
$regex
$expr执行aggregate阶段match操作
$text检索文本字段
$where使用内嵌js表达式
Geospatial
运算符： $geoIntersects,$geoWithin,$near,$nearSphere
Array：数组选择器
$all
含义：
匹配包含指定元素的数组。
Matches arrays that contain all elements specified in the query.
语法：
{ <field>: { $all: [ <value1> , <value2> ... ] } }
注意：
$all与$and的差异在于作用于数组
{ tags: { $all: [ "ssl" , "security" ] } }
{ $and: [ { tags: "ssl" }, { tags: "security" } ] }
示例：
1.使用$all
db.inventory.insertMany([{
   _id: ObjectId("5234cc89687ea597eabee675"),
   code: "xyz",
   tags: [ "school", "book", "bag", "headphone", "appliance" ],
   qty: [
          { size: "S", num: 10, color: "blue" },
          { size: "M", num: 45, color: "blue" },
          { size: "L", num: 100, color: "green" }
        ]
},
{
   _id: ObjectId("5234cc8a687ea597eabee676"),
   code: "abc",
   tags: [ "appliance", "school", "book" ],
   qty: [
          { size: "6", num: 100, color: "green" },
          { size: "6", num: 50, color: "blue" },
          { size: "8", num: 100, color: "brown" }
        ]
},
{
   _id: ObjectId("5234ccb7687ea597eabee677"),
   code: "efg",
   tags: [ "school", "book" ],
   qty: [
          { size: "S", num: 10, color: "blue" },
          { size: "M", num: 100, color: "blue" },
          { size: "L", num: 100, color: "green" }
        ]
},
{
   _id: ObjectId("52350353b2eff1353b349de9"),
   code: "ijk",
   tags: [ "electronics", "school" ],
   qty: [
          { size: "M", num: 100, color: "green" }
        ]
}])
db.inventory.find( { tags: { $all: [ "appliance", "school", "book" ] } } )

2.$all与$elemMatch同时使用
db.inventory.find( {
                     qty: { $all: [
                                    { "$elemMatch" : { size: "M", num: { $gt: 50} } },
                                    { "$elemMatch" : { num : 100, color: "green" } }
                                  ] }
                   } )

$elemMatch(query)
含义：
$elemMatch操作符匹配文档，其包含一个至少有一个元素的数组字段，其匹配所有的查询标准。
The $elemMatch operator matches documents that contain an array field with at least one element that matches all the specified query criteria.
语法：
{ <field>: { $elemMatch: { <query1>, <query2>, ... } } }
注意：
如果查询条件只有一个，则没必要使用$elemMatch
db.survey.find({results:{$elemMatch:{product:”xyz”}}})
等同于
db.survey.find({results.product:”xyz”})
示例：
1.元素匹配
db.scores.drop()
db.scores.insertMany([
{ _id: 1, results: [ 82, 85, 88 ] },
{ _id: 2, results: [ 75, 88, 89 ] }
])
db.scores.find(
   { results: { $elemMatch: { $gte: 80, $lt: 85 } } }
)

2.内嵌文档数组
db.survey.drop()
db.survey.insertMany([
{ _id: 1, results: [ { product: "abc", score: 10 }, { product: "xyz", score: 5 } ] },
{ _id: 2, results: [ { product: "abc", score: 8 }, { product: "xyz", score: 7 } ] },
{ _id: 3, results: [ { product: "abc", score: 7 }, { product: "xyz", score: 8 } ] }
])
db.survey.find(
   { results: { $elemMatch: { product: "xyz", score: { $gte: 8 } } } }
)

3.单个查询条件
db.survey.find(
   { results: { $elemMatch: { product: "xyz" } } }
)
db.survey.find(
   { "results.product": "xyz" }
)

$size
含义：
如果数组字段是指定的大小，则选择文档。
Selects documents if the array field is a specified size.
示例：
db.collection.find( { field: { $size: 2 } } )
位运算符（Bitwise）
运算符： $bitsAllClear,$bitsAllSet,$bitsAnyClear,$bitsAnySet

Comments
$comment
示例：
db.collection.find( { <query>, $comment: <comment> } )
db.records.find({x: { $mod: [ 2, 0 ] },$comment: "Find even values."})
db.records.aggregate( [{ $match: { x: { $gt: 0 }, $comment: "Don't allow negative inputs." } },{ $group : { _id: { $mod: [ "$x", 2 ] }, total: { $sum: "$x" } } }] )
投影选择器（Projection Operators）
$(projection)
含义：
位置操作符$将查询结果中<array>的内容限制为仅包含匹配查询文档的第一个元素。
The positional $ operator limits the contents of an <array> from the query results to contain only the first element matching the query document.
注意：
1.$操作符与$elemMatch操作符都根据条件投射数组中的第一个匹配元素。
Both the $ operator and the $elemMatch operator project the first matching element from an array based on a condition.
2.$操作符根据查询语句的某些条件从集合中的每个文档中投影第一个匹配的数组元素。
The $ operator projects the first matching array element from each document in a collection based on some condition from the query statement.
3.The $elemMatch projection operator takes an explicit condition argument. This allows you to project based on a condition not in the query, or if you need to project based on multiple fields in the array’s embedded documents. See Array Field Limitations for an example.
4.在投影中db.collection.find()不支持$投影操作
db.collection.find() operations on views do not support $ projection operator.
5.当find()方法包含sort()操作时，find()方法在应用$projection操作符之前应用sort命令匹配文档。$只匹配第一个符号条件的元素。
When the find() method includes a sort(),the find() method applies the sort() to order the matching documents before it applies the positional $ projection operator.
示例：
1.数组值投影
db.students.insertMany([
{ "_id" : 1, "semester" : 1, "grades" : [ 70, 87, 90 ] },
{ "_id" : 2, "semester" : 1, "grades" : [ 90, 88, 92 ] },
{ "_id" : 3, "semester" : 1, "grades" : [ 85, 100, 90 ] },
{ "_id" : 4, "semester" : 2, "grades" : [ 79, 85, 80 ] },
{ "_id" : 5, "semester" : 2, "grades" : [ 88, 88, 92 ] },
{ "_id" : 6, "semester" : 2, "grades" : [ 95, 90, 96 ] }])
db.students.find( { semester: 1, grades: { $gte: 85 } },
                  { "grades.$": 1 } )
2.数组文档投影
db.students.insertMany([{ "_id" : 7, semester: 3, "grades" : [ { grade: 80, mean: 75, std: 8 },
                                       { grade: 85, mean: 90, std: 5 },
                                       { grade: 90, mean: 85, std: 3 } ] },
{ "_id" : 8, semester: 3, "grades" : [ { grade: 92, mean: 88, std: 8 },
                                       { grade: 78, mean: 90, std: 5 },
                                       { grade: 88, mean: 85, std: 3 } ] }
])
db.students.find(
   { "grades.mean": { $gt: 70 } },
   { "grades.$": 1 }
)

$elemMatch    
含义：
$elemMatch操作符限定符合的数组字段返回满足匹配条件的第一个元素，如果没有满足匹配规则，则不显示投影。
The $elemMatch operator limits the contents of an <array> field from the query results to contain only the first element matching the '$elemMatch' condition.
注意：
如存在排序操作（大小比较等）则第一个元素为符合操作的那个元素。
示例：
db.school.insertMany([{
_id: 1,
zipcode: "63109",
students: [
              { name: "john", school: 102, age: 10 },
              { name: "jess", school: 102, age: 11 },
              { name: "jeff", school: 108, age: 15 }
           ]
},
{
_id: 2,
zipcode: "63110",
students: [
              { name: "ajax", school: 100, age: 7 },
              { name: "achilles", school: 100, age: 8 },
           ]
},
{
_id: 3,
zipcode: "63109",
students: [
              { name: "ajax", school: 100, age: 7 },
              { name: "achilles", school: 100, age: 8 },
           ]
},
{
_id: 4,
zipcode: "63109",
students: [
              { name: "barney", school: 102, age: 7 },
              { name: "ruth", school: 102, age: 16 },
           ]
}
])
db.schools.find( { zipcode: "63109" },
                 { students: { $elemMatch: { school: 102 } } } )
db.schools.find( { zipcode: "63109" },
                 { students: { $elemMatch: { school: 102, age: { $gt: 10} } } } )


$slice(projection)
含义：
$slice操作符控制查询返回的数组的项目数。1为第一个元素，-1为最后一个元素。
The $slice operator controls the number of items of an array that a query returns.
示例：
db.posts.find( {}, { comments: { $slice: 1} } )
db.posts.find( {}, { comments: { $slice: -1} } )
db.posts.find( {}, { comments: { $slice: [ 1, 5] } } )

$meta
含义：
$meta表达式指定将字段包含到结果集中，而不指定排除其他字段。
The $meta expression specifies the inclusion of the field to the result set and does not specify the exclusion of the other fields.
{ $meta: <metaDataKeyword> }
{ <projectedFieldName>: { $meta: "textScore" } }
示例：
db.collection.find(<query>,{ score: { $meta: "textScore" } })
db.collection.find(<query>,{ score: { $meta: "textScore" } }).sort( { score: { $meta: "textScore" } } )

