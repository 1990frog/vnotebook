# 更新操作符（Update Operators）

一.字段更新操作符（Field Update Operators）

$currentDate
含义：
将字段的值设置为当前日期，可以是日期或时间戳。默认为日期。
Sets the value of a field to current date, either as a Date or a Timestamp,The default type is Date.
语法：
{ $currentDate: { <field1>: <typeSpecification1>, ... } }
<typeSpecification> can be either:
    a boolean true to set the field value to the current date as a Date,or
    a document {$type:"timestamp"} or {$type:"data"} which explicitly specifies the type.The operator is case-sensitive and accepts only the lowercase "timestamp" or thelowercase "date".
注意：
If the field does not exist,$currentDate adds the field to a document.
示例：
db.users.insert({ _id: 1, status: "a", lastModified: ISODate("2013-10-02T01:11:18.965Z") })
db.users.update(
   { _id: 1 },
   {
     $currentDate: {
        lastModified: true,
        "cancellation.date": { $type: "timestamp" }
     },
     $set: {
        status: "D",
        "cancellation.reason": "user request"
     }
   }
)
{
   "_id" : 1,
   "status" : "D",
   "lastModified" : ISODate("2014-09-17T23:25:56.314Z"),
   "cancellation" : {
      "date" : Timestamp(1410996356, 1),
      "reason" : "user request"
   }
}

$inc
含义：
按指定的量增加字段的值。
Increments the value of the field by the specified amount
语法：
{ $inc: { <field1>: <amount1>, <field2>: <amount2>, ... } }
注意：
1.$inc操作符接受正值和负值。
The $inc operator accepts positive and negative values.
2.如果该字段不错在，$inc将创建该字段并将该字段设置为指定的值。
If the field does not exist,$inc creates the field and sets the field to the specified value.
3.$inc操作符在值为null的字段上使用会产生一个错误。
Use of the $inc operator on a field with a null value will 
4.$inc是单个文档中的原子操作。
语法： 
db.products.insertMany([{
  _id: 1,
  sku: "abc123",
  quantity: 10,
  metrics: {
    orders: 2,
    ratings: 3.5
  }
}])
db.products.update(
   { sku: "abc123" },
   { $inc: { quantity: -2, "metrics.orders": 1 } }
)1 } })

$min
含义：
如果指定的值小于现有字段值，则更新字段
Only updates the field if the specified value is less than the existing field value
如果该字段不存在，则$min操作符将该字段设置为指定的值。
if the field does not exists, the $min operator sets the field to the specified value.
语法： 
{ $min: { <field1>: <value1>, ... } }
实例：
db.scores.insert({ _id: 1, highScore: 800, lowScore: 200 })
1.使用$min比较数值
db.scores.update( { _id: 1 }, { $min: { lowScore: 150 } } )
2.使用$min比较时间
db.tags.update({ _id: 1 },{ $min: { dateEntered: new Date("2013-09-25") } }

$max
含义：
如果指定的值大于现有字段值，则更新字段
Only updates the field if the specified value is greater than the existing field value
如果该字段不存在，则$max操作符将该字段设置为指定的值
if the field does not exists, the $max operator sets the field to the specified value.
语法：
{ $max: { <field1>: <value1>, ... } }
实例：
db.scores.insert({ _id: 1, highScore: 800, lowScore: 200 })
1.使用$max比较数值
db.scores.update( { _id: 1 }, { $max: { highScore: 950 } } )
2.使用$max比较时间
db.tags.update({ _id: 1 },{ $max: { dateExpired: new Date("2013-09-30") } })

$mul
含义：
将一个字段的值乘以一个数字
Multiplies the value of a field by a number
如果该字段不存在于文档中，$mul将创建该字段并将该值设置为与乘数相同的数字类型的0.
if the field does not exist in a document,$mul creates the field and sets the value to zero of the same numeric type as the multiplier.
语法： 
{ $mul: { field: <number> } }
实例：
db.products.insert({ _id: 1, item: "ABC", price: 10.99 })
db.products.update({ _id: 1 },{ $mul: { price: 1.25 } })
db.products.update({ _id: 2 },{ $mul: { price: NumberLong(100) } })
db.products.update({ _id: 3 },{ $mul: { price: NumberInt(5) } })

$rename
含义：
1.更改字段名称
Renames a field
2.$rename操作符在逻辑上执行旧名称和新名称的$unset，然后用新名称执行$set操作。因此，操作可能不会保留文档中字段的顺序；即重命名的字段可以在文档内移动。
The $rename operator logically performs an $unset of both the old name and the new name, and then performs a $set operation with the new name. As such, the operation my not preserve the order of the fields in the document;i.e. the renamed field my move within the document.
3，如果文档中已经有一个带有<newName>的字段，则$rename操作符将删除该字段，并将指定的<field>重命名为<newName>.
If the document already has a field with the <newName>,the $rename operator removes that field and renames the specified <field> to <newName>.
4.如果要重命名的字段在文档中不存在，$rename将不执行任何操作
If the field to rename does not exist in a document,$rename does nothing
5.对于嵌入式文档中的字段，$rename操作符可以重命名这些字段，也可以将字段移入货移出嵌入式文档。如果这些字段在数组元素中，$rename不起作用。
For fields in embedded documents, the $rename operator can rename these fields as well as move the fields in and out of embedded documents.$rename does not work if these fields are in array elements.
语法： 
{$rename: { <field1>: <newName1>, <field2>: <newName2>, ... } }
实例：
db.students.insertMany([{
  "_id": 1,
  "alias": [ "The American Cincinnatus", "The American Fabius" ],
  "mobile": "555-555-5555",
  "nmae": { "first" : "george", "last" : "washington" }
},
{
  "_id": 2,
  "alias": [ "My dearest friend" ],
  "mobile": "222-222-2222",
  "nmae": { "first" : "abigail", "last" : "adams" }
},
{
  "_id": 3,
  "alias": [ "Amazing grace" ],
  "mobile": "111-111-1111",
  "nmae": { "first" : "grace", "last" : "hopper" }
}
])
1.重命名一个字段
db.students.update( { _id: 1 }, { $rename: { 'nickname': 'alias', 'cell': 'mobile' } } )
2.重命名一个嵌入式文档字段
db.students.update( { _id: 1 }, { $rename: { "name.first": "name.fname" } } )

$set
含义：
1.$set操作符用指定的值替换字段的值。
The $set operator replaces the value of a field with the specified value.
2.如果该字段不存在，$set将添加一个具有指定值的心字段，前提是新字段不违反类型约束。如果为不存在的字段指定子元素，则$set将根据需要创建嵌入的文档，以实现字段的子元素。
If the field does not exist,$set will add a new field with the specified value,provided that the new field does not violate a type constraint.If you specify a dotted path for a non-existent field,$set will create the embedded documents as needed to fulfill the dotted path to the field.
3.如果你指定了多个字段键值对，则$set将更新货创建每个字段。
If you specify multiple field-value pairs,$set will update or create each field.
语法： 
{ $set: { <field1>: <value1>, ... } }
实例：
db.products.insert({
    _id:100,
    sku:”abc123”,
    quantity:250,
    instock:true,
    reorder:false.
    details:{model:”14q2”,make:”xyzzy”},
    tags:[“apparel”,”clothing”],
    ratings:[{by:”ijk”,rating:4}]
})
1.设置顶层元素
db.products.update( { _id: 100 },{ $set:{quantity: 500」})
2.设置子元素
db.products.update({ _id: 100 },{ $set: { "details.make": "zzz" } })
3.设置数组元素
db.products.update({ _id: 100 },{ $set:{"tags.1": "rain gear","ratings.0.rating": 2}})

$setOnInsert
含义：
如果一个更新操作使用upsert:true新增一条文档，则$setOnInsert为文档新增指定的字段与值。如果更新操作没有完成新增，$setOnInsert无效。
If an update operation with upset:true results in an insert of a document, the $setOnInsert assigns the specified values to the fields in the document.If the update operation does not result in an insert,$setOnInsert does nothing.
如果更新导致插入文档，则设置字段的值。对修改现有文档的更新操作没有影响。
Sets the value of a field if an update results in an insert of a document. Has no effect on update operations that modify existing documents
语法：
db.collection.update(
   <query>,
   { $setOnInsert: { <field1>: <value1>, ... } },
   { upsert: true }
)
实例：
db.products.update(
  { _id: 1 },
  {
     $set: { item: "apple" },
     $setOnInsert: { defaultQty: 100 }
  },
  { upsert: true }
)

$unset
含义：
从文档中删除指定的字段
Removes the specified field from a document
如果字段不存在，则$unset无效
If the field does not exist, then $unset does nothing(i.e. no operation)
当$unset与$一起用来匹配一个数组元素时，$unset用null来替换匹配的元素，而不是从数组中移除匹配的元素。这种行为保持数组的大小和元素位置的一致。
When used with $ to match an array element,$unset replaces the matching element with null rather than removing the matching element from the array.This behavior keeps consistent the array size and element positions.
语法： 
{ $unset: { <field1>: "", ... } }
实例：
db.products.update(
   { sku: "unknown" },
   { $unset: { quantity: "", instock: "" } }
)
二.数组更新操作符（Array Update Operators）

update Operators
$(update)
含义：
充当占位符来更新匹配查询条件的第一个元素
Acts as a placeholder to update the first element that matches the query condition
解疑：
1.位置运算符$标识数组中的元素以更新，而不显示指定数组中元素的位置。
The positional $ operator identifies an element in an array to update without explicity specifying the position of the element in the array.
2.要从读取操作中投影货返回数组元素，请参阅$projection操作符。
To project,or returen,an array element from a read operation,see the $ projection operator instead.
3.要更新数组中的所有元素，请参阅位置运算符$[]。
To update all elements in an array,see the all positional operator $[] instead.
4.要更新与数组过滤器条件匹配的所有元素，请参阅过滤的位置运算符，而不是$[<identifier>]。
To update all elements that match an array filter condition or conditions,see the filtered positional operator instead $[<identifier>].
注意：
1.不要使用位置操作符$来执行upsert操作，因为插入操作将使用$作为插入文档中的字段名称。
Do not use the positional operator $ with upsert operations because inserts will ues the $ as a field name in the inserted document.
2.位移运算符$不能用于遍历多个数组的查询，例如遍历嵌套在其他数组中的查询，因为$占位符替换的是单个值。
The positional $ operator cannot be used for queries which traverse more than one array,such as queries that traverse arrays nested within other arrays,because the replacement for the $ placeholder is a sigle value.
3.当搭配$unset操作符一起使用时，位置操作符$不会从数组中移除所匹配的元素，而是将其设置为null。
When used with the $unset operator,the positional $ operator does not remove the matching element from the array but rather sets it to null.
4.如果查询使用否定运算符（例如$ne,$not,$nin）来匹配数组，则不能使用位置运算符来更新此数组中的值。
If the query matches the array using a negation operator,such as $ne,$not,or $nin,then you cannot use the positional operator the update values from this array.
5.但是，如果查询条件的否定操作位于$elemMatch表达式的内部，则可以使用位置运算符来更新此字段。
However,if the negated portion of the query is inside of an $elemMatch expression,then you can use the positional operator to update this field.
语法：
db.collection.update({ <array>: value ... },{ <update operator>: { "<array>.$" : value } })
db.collection.update({ <query selector>},{ <update operator>: { "array.$.field" : value } })
实例：
1.在数组中更新值
db.students.insert([
   { "_id" : 1, "grades" : [ 85, 80, 80 ] },
   { "_id" : 2, "grades" : [ 88, 90, 92 ] },
   { "_id" : 3, "grades" : [ 85, 100, 90 ] }
])
db.students.insert([
   { "_id" : 1, "grades" : [ 85, 80, 80 ] },
   { "_id" : 2, "grades" : [ 88, 90, 92 ] },
   { "_id" : 3, "grades" : [ 85, 100, 90 ] }
])
db.students.updateOne(
   { _id: 1, grades: 80 },
   { $set: { "grades.$" : 82 } }
)
2.更新数组中的文档
db.collection.insertMany([
{ "_id" : 1, "grades" : [ 85, 82, 80 ] },
{ "_id" : 2, "grades" : [ 88, 90, 92 ] },
{ "_id" : 3, "grades" : [ 85, 100, 90 ] }
])
db.collection.update(
   { <query selector> },
   { <update operator>: { "array.$.field" : value } }
)
db.students.updateOne(
   { _id: 4, "grades.grade": 85 },
   { $set: { "grades.$.std" : 6 } }
)
3.使用多个字段匹配更新嵌入文档
$操作符可以更新匹配$elemMatch操作符指定的多个查询条件的第一个数组元素
db.students.insert({
  _id: 5,
  grades: [
     { grade: 80, mean: 75, std: 8 },
     { grade: 85, mean: 90, std: 5 },
     { grade: 90, mean: 85, std: 3 }
  ]
}
)
db.students.updateOne(
   {
     _id: 5,
     grades: { $elemMatch: { grade: { $lte: 90 }, mean: { $gt: 80 } } }
   },
   { $set: { "grades.$.std" : 6 } }
)

重要：
You must include the array field as part of the query document.

$[]
含义：
充当占位符来更新数组中与查询条件匹配的所有元素。
Acts as a placeholder to update all elements in an array for the documents that match the query condition
语法：
{ <update operator>: { "<array>.$[]" : value } }
db.collection.updateMany(
   { <query conditions> },
   { <update operator>: { "<array>.$[]" : value } }
)
注意：
1.如果一个upsert操作导致新增一条文档，查询条件必须在数组字段中包含一个精确的相等匹配条件，以便在update语句中使用$[]位置运算符。
If an upsert operation results in an insert, the query must include an exact equality match on the array field in order to use the $[] positional operator in the update statement.
实例：
1.upsert更新
db.collection.update({ myArray: [ 5, 8 ] },{ $set: { "myArray.$[]": 10 } },{ upsert: true })
如果upsert操作没有包含完全相等的匹配，并且找不到匹配的文档进行更新，则upsert操作将会出错。
If the upsert operation did not include an exact equality match and no matching documents were found to update, the upsert operation would error.
2.更新数组全部元素
db.students.insertMany([
{ "_id" : 1, "grades" : [ 85, 82, 80 ] },
{ "_id" : 2, "grades" : [ 88, 90, 92 ] },
{ "_id" : 3, "grades" : [ 85, 100, 90 ] }
])
db.students.update(
   { },
   { $inc: { "grades.$[]": 10 } },
   { multi: true }
)
3.更新数组全部文档
db.students2.insertMany([{
   "_id" : 1,
   "grades" : [
      { "grade" : 80, "mean" : 75, "std" : 8 },
      { "grade" : 85, "mean" : 90, "std" : 6 },
      { "grade" : 85, "mean" : 85, "std" : 8 }
   ]
},
{
   "_id" : 2,
   "grades" : [
      { "grade" : 90, "mean" : 75, "std" : 8 },
      { "grade" : 87, "mean" : 90, "std" : 5 },
      { "grade" : 85, "mean" : 85, "std" : 6 }
   ]
}
])
db.students2.update(
   { },
   { $inc: { "grades.$[].std" : -2 } },
   { multi: true }
)
4.使用否定查询操作器指定更新数组
db.results.insertMany([
{ "_id" : 1, "grades" : [ 85, 82, 80 ] },
{ "_id" : 2, "grades" : [ 88, 90, 92 ] },
{ "_id" : 3, "grades" : [ 85, 100, 90 ] }
])
db.results.update(
   { "grades" : { $ne: 100 } },
   { $inc: { "grades.$[]": 10 } },
   { multi: true }
)
5.结合$[<identifier>]更新嵌套数组
db.students3.insert([
   { "_id" : 1,
      "grades" : [
        { type: "quiz", questions: [ 10, 8, 5 ] },
        { type: "quiz", questions: [ 8, 9, 6 ] },
        { type: "hw", questions: [ 5, 4, 3 ] },
        { type: "exam", questions: [ 25, 10, 23, 0 ] },
      ]
   }
])
db.students3.update(
   {},
   { $inc: { "grades.$[].questions.$[score]": 2 } },
   { arrayFilters: [  { "score": { $gte: 8 } } ], multi: true}
)

$[<identifier>]
含义：
1.$[<identifier>]充当占位符以更新与匹配查询条件的文档的arrayFilters条件匹配的所有元素
Acts as a placeholder to update all elements that match the arrayFilters condition for the documents that match the query condition
2.与arrayFIlters选项结合使用，可以更新所有符合查询条件的文档中匹配arrayFilters条件的元素。
Use in conjunction with arrayFilters option to update all elements that match the arrayFilters conditions in the document or documents that match the query conditions.
语法： 
{ <update operator>: { "<array>.$[<identifier>]" : value } },
{ arrayFilters: [ { <identifier>: <condition> } } ] }
db.collection.updateMany(
   { <query conditions> },
   { <update operator>: { "<array>.$[<identifier>]" : value } },
   { arrayFilters: [ { <identifier>: <condition> } } ] }
)
实例：
1.与upsert一起使用
db.collection.update(
   { myArray: [ 0, 1 ] },
   { $set: { "myArray.$[element]": 2 } },
   { arrayFilters: [ { element: 0 } ],
     upsert: true }
)
如果更新的文档不存在，则新增一条文档
如果upsert操作没有包含完全匹配的条件，并且找不到匹配的文档进行更新，则upsert操作将会出错。
db.array.update(
   { },
   { $set: { "myArray.$[element]": 10 } },
   { arrayFilters: [ { element: 9 } ],
     upsert: true }
)
WriteResult({
   "nMatched" : 0,
   "nUpserted" : 0,
   "nModified" : 0,
   "writeError" : {
      "code" : 2,
      "errmsg" : "The path 'myArray' must exist in the document in order to apply array updates."
   }
})
2.更新符合arrayFilters的全部数组元素
db.students.insertMany([
{ "_id" : 1, "grades" : [ 95, 92, 90 ] },
{ "_id" : 2, "grades" : [ 98, 100, 102 ] },
{ "_id" : 3, "grades" : [ 95, 110, 100 ] }
])
db.students.update(
   { },
   { $set: { "grades.$[element]" : 100 } },
   { multi: true,
     arrayFilters: [ { "element": { $gte: 100 } } ]
   }
)
3.更新数组中匹配arrayFilters的全部内嵌文档
Syntax:
db.collection.update(
   { <query selector> },
   { <update operator>: { "array.$[<identifier>].field" : value } },
   { arrayFilters: [ { <identifier>: <condition> } } ] }
)
db.students2.insertMany([{
   "_id" : 1,
   "grades" : [
      { "grade" : 80, "mean" : 75, "std" : 6 },
      { "grade" : 85, "mean" : 90, "std" : 4 },
      { "grade" : 85, "mean" : 85, "std" : 6 }
   ]
},
{
   "_id" : 2,
   "grades" : [
      { "grade" : 90, "mean" : 75, "std" : 6 },
      { "grade" : 87, "mean" : 90, "std" : 3 },
      { "grade" : 85, "mean" : 85, "std" : 4 }
   ]
}
])
db.students2.update(
   { },
   { $set: { "grades.$[elem].mean" : 100 } },
   {
     multi: true,
     arrayFilters: [ { "elem.grade": { $gte: 85 } } ]
   }
)
4.更新匹配多个条件的所有数组元素
db.students.insertMany([{
   "_id" : 1,
   "grades" : [
      { "grade" : 80, "mean" : 75, "std" : 6 },
      { "grade" : 85, "mean" : 100, "std" : 4 },
      { "grade" : 85, "mean" : 100, "std" : 6 }
   ]
},
{
   "_id" : 2,
   "grades" : [
      { "grade" : 90, "mean" : 100, "std" : 6 },
      { "grade" : 87, "mean" : 100, "std" : 3 },
      { "grade" : 85, "mean" : 100, "std" : 4 }
   ]
}
])
db.students.update(
   { },
   { $inc: { "grades.$[elem].std" : -1 } },
   { arrayFilters: [ { "elem.grade": { $gte: 80 }, "elem.std": { $gt: 5 } } ], multi: true }
)
5.使用否定操作符更改数组元素
db.alumni.insertMany([{
   "_id": 1,
   "name": "Christine Franklin",
   "degrees": [
      { "level": "Master",
        "major": "Biology",
        "completion_year": 2010,
        "faculty": "Science"
      },
      {
        "level": "Bachelor",
        "major": "Biology",
        "completion_year": 2008,
        "faculty": "Science"
      }
   ],
   "school_email": "cfranklin@example.edu",
   "email": "christine@example.com"
},
{
   "_id": 2,
   "name": "Reyansh Sengupta",
   "degrees": [
      { "level": "Bachelor",
        "major": "Chemical Engineering",
        "completion_year": 2002,
        "faculty": "Engineering"
      }
   ],
   "school_email": "rsengupta2@example.edu"
}
])
db.alumni.update(
   { },
   { $set : { "degrees.$[degree].gradcampaign" : 1 } },
   { arrayFilters : [ {"degree.level" : { $ne: "Bachelor" } } ],
     multi : true }
)
6.联合$[]更新内嵌数组
db.students3.insert([
   { "_id" : 1,
      "grades" : [
        { type: "quiz", questions: [ 10, 8, 5 ] },
        { type: "quiz", questions: [ 8, 9, 6 ] },
        { type: "hw", questions: [ 5, 4, 3 ] },
        { type: "exam", questions: [ 25, 10, 23, 0 ] },

      ]
   }
])
db.students3.update(
   {},
   { $inc: { "grades.$[t].questions.$[score]": 2 } },
   { arrayFilters: [ { "t.type": "quiz" } , { "score": { $gte: 8 } } ], multi: true}
)
db.students3.update(
   {},
   { $inc: { "grades.$[].questions.$[score]": 2 } },
   { arrayFilters: [  { "score": { $gte: 8 } } ], multi: true}
)

$addToSet
含义：
$addToSet操作符为数组添加一个值，除非该值已经存在，在这种情况下$addToSet对该数组不做任何操作。
The $addToSet operator adds a value to an array unless the value is already present,in which case $addToSet does nothing to that array.
语法： 
{ $addToSet: { <field1>: <value1>, ... } }
实例：
db.inventory.insert({ _id: 1, item: "polarizing_filter", tags: [ "electronics", "camera" ] })
1.添加到数组
db.inventory.update(
   { _id: 1 },
   { $addToSet: { tags: "accessories" } }
)
2.添加存在的值
db.inventory.update(
   { _id: 1 },
   { $addToSet: { tags: "camera"  } }
)
3.$each调节器
db.inventory.insert({ _id: 2, item: "cable", tags: [ "electronics", "supplies" ] })
db.inventory.update(
   { _id: 2 },
   { $addToSet: { tags: { $each: [ "camera", "electronics", "accessories" ] } } }
)

$pop
含义：
$pop操作符删除数组的第一个货最后一个元素。$pop通过数字-1来删除数组的第一个元素，通过1来删除数组中的最后一个元素。
The $pop operator removes the first or last element of an array.Pass $pop a value of -1 to remove the first element of an array and 1 to remove the last element in array.
注意：
1.$pop操作符只能应用于数组字段
The $pop operation fails if the <field> is not an array.
2.如果$pop操作符删除了数组字段的最后一个元素，这个字段将持有一个空数组。
If the $pop operator removes the last item in the <field>,the <field> will then hold an empty array
3.$pop expects 1 or -1
语法：
{ $pop: { <field>: <-1 | 1>, ... } }
实例：
数据：
db.student.insert({_id:1,scores:[8,9,10]})
1.删除数组的第一个元素
db.students.update( { _id: 1 }, { $pop: { scores: -1 } } )
2.删除数组的最后一个元素
db.students.update( { _id: 1 }, { $pop: { scores: 1 } } )

$pull
含义：
$pull操作符从现有数组中移除一个或多个匹配指定条件的实例。
The $pull operator removes from an existing array all instances of a value or values that match a specified condition.
语法： 
{ $pull: { <field1>: <value|condition>, <field2>: <value|condition>, ... } }
实例：
1.删除说有等于指定元素的项目
db.stores.insertMany([{
   _id: 1,
   fruits: [ "apples", "pears", "oranges", "grapes", "bananas" ],
   vegetables: [ "carrots", "celery", "squash", "carrots" ]
},
{
   _id: 2,
   fruits: [ "plums", "kiwis", "oranges", "bananas", "apples" ],
   vegetables: [ "broccoli", "zucchini", "carrots", "onions" ]
}
])
db.stores.update(
    { },
    { $pull: { fruits: { $in: [ "apples", "oranges" ] }, vegetables: "carrots" } },
    { multi: true }
)
2.删除所有匹配指定$pull条件的项目
db.profiles.insert({ _id: 1, votes: [ 3, 5, 6, 7, 7, 8 ] })
db.profiles.update( { _id: 1 }, { $pull: { votes: { $gte: 6 } } } )
3.从文档数组中删除项目
db.survey.insertMany([{
   _id: 1,
   results: [
      { item: "A", score: 5 },
      { item: "B", score: 8, comment: "Strongly agree" }
   ]
},
{
   _id: 2,
   results: [
      { item: "C", score: 8, comment: "Strongly agree" },
      { item: "B", score: 4 }
   ]
}
])
db.survey.update(
  { },
  { $pull: { results: { score: 8 , item: "B" } } },
  { multi: true }
)
db.survey.update(
  { },
  { $pull: { results: { $elemMatch: { score: 8 , item: "B" } } } },
  { multi: true }
)
db.survey.update(
  { },
  { $pull: { results: { answers: { $elemMatch: { q: 2, a: { $gte: 8 } } } } } },
  { multi: true }
)

$push
含义：
1.$push操作符添加指定的值到数组
The $push operator appends a specified value to an array.
2.数组或内嵌文档的子元素字段，使用逗点操作符
To specify a <field> in an embedded document or in an array, use dot notation.
注意：
1.如果该字段在文档中不存在以进行更新，则$push将会将数组字段的值添加为其元素。
If the field is absent in the document to update,$push adds the array field with the value as its element.
2.如果字段不是数组，操作将会失败。
If the field is not an array.the operation will fail. 
3.如果该值是一个数组，$push将会将这整个数组作为单个元素添加到指定的数组。要分别添加该值的每个元素，请使用$push与$each操作符。
If the value is an array,$push appends the whole array as a single element.To add each element of the value separately, use the $each modifier with $push.
语法： 
{ $push: { <field1>: <value1>, ... } }
{ $push: { <field1>: { <modifier1>: <value1>, ... }, ... } }
The processing of the push operation with modifiers occur in the following order, regardless of the order in which the modifiers appear:
Update array to add elements in the correct position.
Apply sort, if specified.
Slice the array, if specified.
Store the array.
实例：
1.添加一个值到数组
db.students.update(
   { _id: 1 },
   { $push: { scores: 89 } }
)
2.添加多个值到数组
db.students.update(
   { name: "joe" },
   { $push: { scores: { $each: [ 90, 92, 85 ] } } }
)
3.搭配多个调度器使用$push
db.student.insert({
   "_id" : 5,
   "quizzes" : [
      { "wk": 1, "score" : 10 },
      { "wk": 2, "score" : 8 },
      { "wk": 3, "score" : 5 },
      { "wk": 4, "score" : 6 }
   ]
}
)
db.students.update(
   { _id: 5 },
   {
     $push: {
       quizzes: {
          $each: [ { wk: 5, score: 8 }, { wk: 6, score: 7 }, { wk: 7, score: 6 } ],
          $sort: { score: -1 },
          $slice: 3
       }
     }
   }
)

$pullAll
含义：
$pullAll操作符从存在的数组中删除指定的全部值。与$pull操作符不同其通过一个指定的查询来删除元素，$pullAll通过列表来删除元素。
The $pullAll operator removes all instances of the specified values from an existing array.Unlike the $pull operator that removes elements by specifying a query,$pullAll removes elements that match the listed values.
注意：
如果要移除的值是文档或数组，则$pullAll将仅精确的移除数组中与指定的值相匹配的元素，包括顺序。
If a <value> to remove is a document or an array,$pullAll removes only the elements in the array that match the specified <value> exactly,including order.
语法： 
{ $pullAll: { <field1>: [ <value1>, <value2> ... ], ... } }
实例：
db.survey.insert({ _id: 1, scores: [ 0, 2, 5, 5, 1, 0 ] })
db.survey.update( { _id: 1 }, { $pullAll: { scores: [ 0, 5 ] } } )

三.更新操作符调节器（Update Operator Modifiers）
$each
含义：
$each修饰符可用于$addToSet操作符和$push操作符。
The $each modifier is available for use with the $addToSet operator and the $push operator.
语法：
{ $addToSet: { <field>: { $each: [ <value1>, <value2> ... ] } } }
{ $push: { <field>: { $each: [ <value1>, <value2> ... ] } } }

$position
含义：
$position修饰符与$push操作符搭配使用插入元素时，指定插入元素在数组中的位置。如果没有$position修饰符，$push操作符会将元素插入数组的末端。
The $position modifier specifies the location in the array at which the $push operator inserts elements.Without the $position modifier,the $push operator inserts elements to the end of the array.
语法：
{
  $push: {
    <field>: {
       $each: [ <value1>, <value2>, ... ],
       $position: <num>
    }
  }
}
注意：
1.非负数对应于数组中的位置，从数组的开始处开始。如果<num>的值大于或等于数组的长度，$position修饰符不起作用，$push将元素插入到数组的末尾。
A non-negative number corresponds to the position in the array,starting from the beginning of the array.If the value of <num> is greater or equal to length of the array,the $position modifier has no effect and $push adds elements to the end of the array.
2.负数对应于数组中的位置，从（但不包括）数组的最后一个元素开始计数。例如，-1表示数组中最后一个元素之前的位置。如果在$each数组中指定了多个元素，则最后添加的元素将在末尾的指定位置。如果<num>的绝对值大于或等于数组的长度，则$push会将元素添加到数组的开头。
A negative number corresponds to the position in the array,counting from (but not including) the last element of the array.For example,-1 indicates the position just before the last element in the array. If you specify multiple elements in the $each array, the last added element is in the specified position from the end.If the absolute value of <num> is greater than or equal to the length of the array,the $push adds elements to the beginning of the array.
实例：
1.num为正数
db.students.update({ _id: 1 },{$push: {scores: {$each: [ 50, 60, 70 ],$position: 0}}})
{_id:1,scores:[50, 60, 70,100]}
db.students.update({ _id: 1 },{$push: {scores: {$each: [ 20, 30 ],$position: 2}}})
{_id:1,scores:[50,60,20,30,70,100]}
2.num为负数
db.students.update({ _id: 1 },{$push: {scores: {$each: [ 90, 80 ],$position: -2}}})
{_id:1,scores:[50,60,20,30,90,80,70,100]}
3.在数组开始的位置添加元素
db.students.insert({ "_id" : 1, "scores" : [ 100 ] })
db.students.update(
   { _id: 1 },
   {
     $push: {
        scores: {
           $each: [ 50, 60, 70 ],
           $position: 0
        }
     }
   }
)
4.在数组中间位置添加元素
db.students.insert({ "_id" : 1, "scores" : [  50,  60,  70,  100 ] })
db.students.update(
   { _id: 1 },
   {
     $push: {
        scores: {
           $each: [ 20, 30 ],
           $position: 2
        }
     }
   }
)
5.使用一个相反的序列添加元素到数组
db.students.insert({ "_id" : 1, "scores" : [  50,  60,  20,  30,  70,  100 ] })
db.students.update(
   { _id: 1 },
   {
     $push: {
        scores: {
           $each: [ 90, 80 ],
           $position: -2
        }
     }
   }
)

$slice
含义：
1.$slice修饰符在$push操作期间现在数组元素的数量。要从读取操作中投影或返回指定的数组元素。
The $slice modifier limits the numbers of array elements during a $push operation.To projec,or reutrn,a specified number of array elements from a read operation.
2.要使用$slice修饰符，它必须与$each修饰符一起出现。你可以传递一个空数组给$each操作符，这样只有$slice操作符有效。
To use the $slice modifier,it must appear with the $each modifier.You can pass an empty array [] to the $each modifier such that only the $slice modifier has an effect.
语法：
{$push: {<field>: {$each: [ <value1>, <value2>, ... ],$slice: <num>}}}
<num>可以为下列情况：
Zero:更新数组字段为一个空数组
Negative:更新数组字段为一个包含倒数<num>个元素的数组
Positive:更新数组字段只包含正数<num>个元素的数组
实例：
1.从后切割数组
db.students.insert({ "_id" : 1, "scores" : [ 40, 50, 60 ] })
db.students.update(
   { _id: 1 },
   {
     $push: {
       scores: {
         $each: [ 80, 78, 86 ],
         $slice: -5
       }
     }
   }
)
2.从前面切割数组
db.students.insert({ "_id" : 2, "scores" : [ 89, 90 ] })
db.students.update(
   { _id: 2 },
   {
     $push: {
       scores: {
         $each: [ 100, 20 ],
         $slice: 3
       }
     }
   }
)
3.仅切割数组
db.students.insert({ "_id" : 3, "scores" : [  89,  70,  100,  20 ] })
db.students.update(
  { _id: 3 },
  {
    $push: {
      scores: {
         $each: [ ],
         $slice: -3
      }
    }
  }
)
4.搭配其他$push操作符
db.students.insert({
   "_id" : 5,
   "quizzes" : [
      { "wk": 1, "score" : 10 },
      { "wk": 2, "score" : 8 },
      { "wk": 3, "score" : 5 },
      { "wk": 4, "score" : 6 }
   ]
}
)
db.students.update(
   { _id: 5 },
   {
     $push: {
       quizzes: {
          $each: [ { wk: 5, score: 8 }, { wk: 6, score: 7 }, { wk: 7, score: 6 } ],
          $sort: { score: -1 },
          $slice: 3
       }
     }
   }
)

$sort
含义：
1.$sort修饰符在$push操作符中排序数组元素
The $sort modifier orders the elements of an array during a $push operation.
2.使用$sort修饰符，它必须与$each修饰符一起出现。你可以在$push操作符中使用一个空数组[],这样只有$sort修饰符有效果。
To use the $sort modifier,it must appear with the $each modifier.You can pass an empty array [] to the $each modifier such that only the $sort modifier has an effect.
语法：
{
  $push: {
     <field>: {
       $each: [ <value1>, <value2>, ... ],
       $sort: <sort specification>
     }
  }
}
For <sort specification>:
To sort array elements that are not documents, or if the array elements are documents, to sort by the whole documents, specify 1 for ascending or -1 for descending.
If the array elements are documents, to sort by a field in the documents, specify a sort document with the field and the direction, i.e. { field: 1 } or { field: -1 }. Do not reference the containing array field in the sort specification (e.g. { "arrayField.field": 1 } is incorrect).
注意：
1.对不是文档的数组元素进行排序，或者如果数组元素是文档，则按整个文档进行排序，请将1指定为升序，将-1指定为降序。
To sort array elements that are not documents,or if the array elements are documents,to sort by the whole documents,specify 1 for ascending or -1 for descending.
2.如果数组元素是文档，要按文档中的字段进行排序，请指定具有字段和方向的排序文档，即{field:1}或{field:-1}。不要在排序规范中引用包含数组的字段（例如{arrayField.field:1}不正确)。
If the array elements are documents,to sort by a field in the documents,specify a sort document with the field the direction,i.e {field:1} or {field:-1}.Do not reference the containing array field in the sort specification (e.g. {arrayField.field:1} is incorrect).
要点：
1.$sort修饰符可以对不是文档的数组进行排序。在以前的版本中，$sort要求数组元素是文档。
The $sort modifier can sort array elements that are not documents.In previous versions, the $sort modifier required the array elements be documents.
2.如果数组元素是文档，则修饰符可以按整个文档或文档中的特定字段进行排序。在以前的版本中，$sort修饰符只能按文档中特定字段进行排序。
If the array elements are documents, the modifier can sort by either the whole document or by a specific field in the documents.In previous versions, the $sort modifier can only sort by a specific field in the documents.
3.尝试在没有$each修饰符的情况下使用$sort修饰符会导致错误。$sort不再需要$slice修饰符。
Trying to use the $sort modifier without the $each modifier results in an error.The $sort no longer requires the $slice modifier.For a list of modifiers available for $push,see Modifiers.
实例：
1.在文档中排序一个文档数组
db.students.insert({
  "_id": 1,
  "quizzes": [
    { "id" : 1, "score" : 6 },
    { "id" : 2, "score" : 9 }
  ]
}
)
db.students.update(
   { _id: 1 },
   {
     $push: {
       quizzes: {
         $each: [ { id: 3, score: 8 }, { id: 4, score: 7 }, { id: 5, score: 6 } ],
         $sort: { score: 1 }
       }
     }
   }
)
2.排序非文档类型的数组元素
db.students.insert({ "_id" : 2, "tests" : [  89,  70,  89,  50 ] })
db.students.update(
   { _id: 2 },
   { $push: { tests: { $each: [ 40, 60 ], $sort: 1 } } }
)
3.仅使用排序更新数组
db.students.insert({ "_id" : 3, "tests" : [  89,  70,  100,  20 ] })
db.students.update(
   { _id: 3 },
   { $push: { tests: { $each: [ ], $sort: -1 } } }
)






