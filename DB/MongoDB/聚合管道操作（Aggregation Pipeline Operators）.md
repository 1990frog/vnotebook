# 聚合管道操作（Aggregation Pipeline Operators）

$abs(aggregation)
含义：
返回数字的绝对值。
Returns the absolute value of a number.
语法：
{ $abs: <number> }
实例：
db.ratings.insertMany([
{ _id: 1, start: 5, end: 8 },
{ _id: 2, start: 4, end: 4 },
{ _id: 3, start: 9, end: 7 },
{ _id: 4, start: 6, end: 7 }
])
db.ratings.aggregate([{$project: { delta: { $abs: { $subtract: [ "$start", "$end" ] } } }}])

$add(aggregation)
含义：
相加数字并返回总值，或者将数字与日期相加并返回一个新日期。如果将日期与数值相加，会将数值看作为毫秒。接受任意数量的参数表达式，但最多只有一个表达式可以解析为日期。
Adds numbers to return the sum, or adds numbers and a date to return a new date. If adding numbers and a date, treats the numbers as milliseconds. Accepts any number of argument expressions, but at most, one expression can resolve to a date.
语法：
{ $add: [ <expression1>, <expression2>, ... ] }
实例：
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "fee" : 2, date: ISODate("2014-03-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "fee" : 1, date: ISODate("2014-03-01T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5,  "fee" : 0, date: ISODate("2014-03-15T09:00:00Z") }
])
1.number相加
db.sales.aggregate([{ $project: { item: 1, total: { $add: [ "$price", "$fee" ] } } }])
2.number加date
db.sales.aggregate([{ $project: { item: 1, billing_date: { $add: [ "$date", 3*24*60*60000 ] } } }])

$addToSet(aggregation)
含义：
1.返回一个不包含雷同元素的数组，其数组通过group操作，可将相同的key对应的组中的字段添加于一个新的字段，其字段是set格式，其输出结果不执行排序操作。
Returns an array of all unique values that results from applying an expression to each document in a group of documents that share the same group by key.Order of the elements in the output array is unspecified.
2.只在$group阶段有效
Available in $group stage only.
3.如果表达式的值是一个数组，$addToSet会把整个数组添加为一个元素
If the value of the expression is an array,$addToSet appends the whole array as a single element.
语法：
{ $addToSet: <expression> }
实例：
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") },
{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") },
{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:12:00Z") }
])
db.sales.aggregate([{$group:{_id: { day: { $dayOfYear: "$date"}, year: { $year: "$date" } },itemsSold: { $addToSet: "$item" }}}])

$allElementsTrue(aggregation)
含义：
作为一个集合评估一个数组，如果数组中没有元素为false，则返回true。反之，则返回false。一个空数组则返回true。
Evaluates an array as a set and returns true if no element in the array is false.Otherwise,returns false.An empty array returns true.
语法：
{ $allElementsTrue: [ <expression> ] }
{ $allElementsTrue: [ [ true, 1, "someString" ] ] },true
{ $allElementsTrue: [ [ [ false ] ] ] },true
{ $allElementsTrue: [ [ ] ] },true
{ $allElementsTrue: [ [ null, false, 0 ] ] },false
实例：
db.survey.insertMany([
{ "_id" : 1, "responses" : [ true ] },
{ "_id" : 2, "responses" : [ true, false ] },
{ "_id" : 3, "responses" : [ ] },
{ "_id" : 4, "responses" : [ 1, true, "seven" ] },
{ "_id" : 5, "responses" : [ 0 ] },
{ "_id" : 6, "responses" : [ [ ] ] },
{ "_id" : 7, "responses" : [ [ 0 ] ] },
{ "_id" : 8, "responses" : [ [ false ] ] },
{ "_id" : 9, "responses" : [ null ] },
{ "_id" : 10, "responses" : [ undefined ] }
])
db.survey.aggregate([{ $project: { responses: 1, isAllTrue: { $allElementsTrue: [ "$responses" ] }, _id: 0 } }])
空元素：0，false，null，underfined
非空元素[0],[false]…...

$and(aggregation)
含义：
评估一个或多个表达式，如果所有表达式都为真或没有表达式的则返回true，否则，$and返回false。
Evaluates one or more expressions and returns true if all of the expressions are true or if evoked with no argument expressions.Otherwise,$and returns false.
语法：
{ $and: [ <expression1>, <expression2>, ... ] }
{ $and: [ 1, "green" ] },true
{ $and: [ ] },true
{ $and: [ [ null ], [ false ], [ 0 ] ] },true
{ $and: [ null, true ] },false
{ $and: [ 0, true ] },false
false:0,null,undefined
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            qty: 1,
            result: { $and: [ { $gt: [ "$qty", 100 ] }, { $lt: [ "$qty", 250 ] } ] }
          }
     }
   ]
)

$anyElementTrue(aggregation)
含义：
1.将数组视为一个集合，如果其中任何一个元素为真返回true，反之返回false。如果数组为空数组返回false。
Evaluates an array as a set and returns true if any of the elements are true and false otherwise.An empty array returns false.
2.如果一个集合包含嵌套数组元素，$anyElementTrue不能向下操作嵌套数组，而是在顶层评估数组。
If a set contains a nested array element,$anyElementTrue does not descend into the nested array but evaluates the array at top-level.
语法：
{ $anyElementTrue: [ <expression> ] }
{ $anyElementTrue: [ [ true, false ] ] },true
{ $anyElementTrue: [ [ [ false ] ] ] },true
{ $anyElementTrue: [ [ null, false, 0 ] ] },false
{ $anyElementTrue: [ [ ] ] },false
实例：
db.survey.insertMany([
{ "_id" : 1, "responses" : [ true ] },
{ "_id" : 2, "responses" : [ true, false ] },
{ "_id" : 3, "responses" : [ ] },
{ "_id" : 4, "responses" : [ 1, true, "seven" ] },
{ "_id" : 5, "responses" : [ 0 ] },
{ "_id" : 6, "responses" : [ [ ] ] },
{ "_id" : 7, "responses" : [ [ 0 ] ] },
{ "_id" : 8, "responses" : [ [ false ] ] },
{ "_id" : 9, "responses" : [ null ] },
{ "_id" : 10, "responses" : [ undefined ] }
])
db.survey.aggregate(
   [
     { $project: { responses: 1, isAnyTrue: { $anyElementTrue: [ "$responses" ] }, _id: 0 } }
   ]
)

$arrayElemAt(aggregation)
含义：
返回指定数组下标所代表的元素
Returns the element at the specified array index.
语法：
{ $arrayElemAt: [ <array>, <idx> ] }
{ $arrayElemAt: [ [ 1, 2, 3 ], 0 ] }返回1
{ $arrayElemAt: [ [ 1, 2, 3 ], -2 ] }返回2
{ $arrayElemAt: [ [ 1, 2, 3 ], 15 ] }返回空   
实例：
db.users.insertMany([
{ "_id" : 1, "name" : "dave123", favorites: [ "chocolate", "cake", "butter", "apples" ] },
{ "_id" : 2, "name" : "li", favorites: [ "apples", "pudding", "pie" ] },
{ "_id" : 3, "name" : "ahn", favorites: [ "pears", "pecans", "chocolate", "cherries" ] },
{ "_id" : 4, "name" : "ty", favorites: [ "ice cream" ] }
])
db.users.aggregate([
   {
     $project:
      {
         name: 1,
         first: { $arrayElemAt: [ "$favorites", 0 ] },
         last: { $arrayElemAt: [ "$favorites", -1 ] }
      }
   }
])

$arrayToObject(aggregation)
含义：
1.将一个数组转化为单个文档，数组必须是：
（1）一个包含两个子数组的数组，其第一个元素作为field字段，其第二个元素作为field字段的值。
（2）一个文档数组其包含2个值，k和v
    k为field名称
    v为field的值
2.如果数组中field字段重复，$arrayToObject使用第一个值作为field的值
1.Coverts an array into a single document; the array must be either:
(1).An array of two-element arrays where the first element is the field name, and the second element is the field value
(2).An array of documents that contains two fields, k and v where:
    the k field contains the field name.
    the v field contains the value of the field.
2.If the name of a field repeats in the array,$arrayToObject uses the first value for that field.
语法：
{ $arrayToObject: <expression> }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "ABC1",  dimensions: [ { "k": "l", "v": 25} , { "k": "w", "v": 10 }, { "k": "uom", "v": "cm" } ] },
{ "_id" : 2, "item" : "ABC2",  dimensions: [ [ "l", 50 ], [ "w",  25 ], [ "uom", "cm" ] ] },
{ "_id" : 3, "item" : "ABC3",  dimensions: [ [ "l", 50 ], [ "l",  25 ], [ "l", "cm" ] ] }
])
db.inventory.aggregate(
   [
      {
         $project: {
            item: 1,
            dimensions: { $arrayToObject: "$dimensions" }
         }
      }
   ]
)

$avg(aggregation)
含义：
返回数值的平均值。$avg会忽略非数字的值。
Returns the average value of the numeric values.$avg ignores non-numeric values.
在3.2版本中：$avg可以在$group与$project阶段中使用。在以前的版本中仅在$group中可用。
Changed in version 3.2: $avg is available in the $group and $project stages. In previous versions of MongoDB, $avg is available in the $group stage only.
注意：
1.$avg忽略非数字值，包括缺失值。如果平均值的全部操作都是非数字的，$avg返回null，因为零值的平均值是未定义的。
$avg ignores non-numeric values, including missing values.If all of the operands for the average are non-numeric,$avg returns null since the average of zero values is undefined.
2.在$project操作中使用单一表达式作为其操作，如果这个表达式解析为一个数组，$avg将遍历这个数组的每一个数字元素以返回单个值。
With a single expression as its operand, if the expression resolves to an array,$avg traverses into the array to operate on the numerical elements of the array to return a single value.
3.在$project操作中使用多个表达式，如果有任何表达式被解析为数组，$avg不会遍历数组，而是将该数组视为非数值。
With a list of expressions as its operand, if any of the expressions resolves to an array,$avg does not traverse into array but instead treats the array as a non-numerical value.
语法：
{ $avg: <expression> }
{ $avg: [ <expression1>, <expression2> ... ]  }
实例：
1.在$group阶段使用
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") },
{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") },
{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:12:00Z") }
])
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: "$item",
           avgAmount: { $avg: { $multiply: [ "$price", "$quantity" ] } },
           avgQuantity: { $avg: "$quantity" }
         }
     }
   ]
)

2.在$project阶段使用
db.student.insertMany([
{ "_id": 1, "quizzes": [ 10, 6, 7 ], "labs": [ 5, 8 ], "final": 80, "midterm": 75 },
{ "_id": 2, "quizzes": [ 9, 10 ], "labs": [ 8, 8 ], "final": 95, "midterm": 80 },
{ "_id": 3, "quizzes": [ 4, 5, 5 ], "labs": [ 6, 5 ], "final": 78, "midterm": 70 }
])
db.students.aggregate([
   {
     $project: {
       quizAvg: { $avg: "$quizzes"},
       labAvg: { $avg: "$labs" },
       examAvg: { $avg: [ "$final", "$midterm" ] }
     }
   }
])

$ceil(aggregation)
含义：
返回大于或等于指定数字的最小整数。
Returns the smallest integer greater than or equal to the specified number.
语法：
{ $ceil: <number> }
{ $ceil: 1 }返回1
{ $ceil: 7.80 }返回8
{ $ceil: -2.8 }返回-2
实例：
db.samples.insertMany([
{ _id: 1, value: 9.25 },
{ _id: 2, value: 8.73 },
{ _id: 3, value: 4.32 },
{ _id: 4, value: -5.34 }
])
db.samples.aggregate([
   { $project: { value: 1, ceilingValue: { $ceil: "$value" } } }
])

$cmp(aggregation)
含义：
Compares two values and returns:
-1 if the first value is less than the second.
1 if the first value is greater than the second.
0 if the two values are equivalent.
语法：
{ $cmp: [ <expression1>, <expression2> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            qty: 1,
            cmpTo250: { $cmp: [ "$qty", 250 ] },
            _id: 0
          }
     }
   ]
)

$concat(aggregation)
含义：
concat为concatenate缩写
连接字符串并返回连接的字符串。
Concatenates strings and returns the concatenated string.
参数可以是任何有效的表达式，只要它们解析为字符串即可。
The arguments can be any valid expression as long as they resolve to strings.
如果参数解析为null或指向不存在字段的空指针，$concat返回null。
If the argument resolves to a value of null or refers to a field that is missing,$concat returns null.
语法：
{ $concat: [ <expression1>, <expression2>, ... ] }
实例：
db.inventory.isnertMany([
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "product 1" },
{ "_id" : 2, "item" : "ABC2", quarter: "13Q4", "description" : "product 2" },
{ "_id" : 3, "item" : "XYZ1", quarter: "14Q2", "description" : null }
])
db.inventory.aggregate(
   [
      { $project: { itemDescription: { $concat: [ "$item", " - ", "$description" ] } } }
   ]
)

$concatArrays(aggregation)
含义：
合并数组并返回合并后的数组。
Concatenates arrays to return the concatenated array.
只要他们解析为一个数组，那么<array>表达式可以是任何有效的表达式。
The <array> expressions can be any valid expression as long as they resolve to an array.
如果参数解析为null或引用缺失的字段，$concatArrays将返回null。
If an argument resolves to a value of null or refers to a field that is missing,$concatArrays returns null.
语法：
{ $concatArrays: [ <array1>, <array2>, ... ] }
{ $concatArrays: [ [ "hello", " "], [ "world" ] ] }    
[ "hello", " ", "world" ]
{ $concatArrays: [ [ "hello", " "], [ [ "world" ], "again"] ] }    
[ "hello", " ", [ "world" ], "again" ]
实例：
db.warehouses.isnertMany([
{ "_id" : 1, instock: [ "chocolate" ], ordered: [ "butter", "apples" ] },
{ "_id" : 2, instock: [ "apples", "pudding", "pie" ] },
{ "_id" : 3, instock: [ "pears", "pecans"], ordered: [ "cherries" ] },
{ "_id" : 4, instock: [ "ice cream" ], ordered: [ ] }
])
db.warehouses.aggregate([
   { $project: { items: { $concatArrays: [ "$instock", "$ordered" ] } } }
])

$cond(aggregation)
含义：
评估一个布尔表达式来返回两个指定的返回表达式的其中一个。
Evaluates a boolean expression to return one of the two specified return expressions.
语法：
1.使用if, then,else
{ $cond: { if: <boolean-expression>, then: <true-case>, else: <false-case-> } }
2.不使用
{ $cond: [ <boolean-expression>, <true-case>, <false-case> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", qty: 300 },
{ "_id" : 2, "item" : "abc2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", qty: 250 }
])
db.inventory.aggregate(
   [
      {
         $project:
           {
             item: 1,
             discount:
               {
                 $cond: { if: { $gte: [ "$qty", 250 ] }, then: 30, else: 20 }
               }
           }
      }
   ]
)
db.inventory.aggregate(
   [
      {
         $project:
           {
             item: 1,
             discount:
               {
                 $cond: [ { $gte: [ "$qty", 250 ] }, 30, 20 ]
               }
           }
      }
   ]
)

$dayOfMonth(aggregation)，
$dayOfWeek(aggregation)，
$dayOfYear(aggregation)，
$hour(aggregation)，
$millisecond(aggregation) ，
$minute(aggregation) ，
$month(aggregation)，
$second(aggregation)，
$week(aggregation)，
$year(aggregation)
含义：
Returns the day of the month for a date as a number between 1 and 31.
{ $dayOfMonth: <dateExpression> }
Returns the day of the week for a date as a number between 1 (Sunday) and 7 (Saturday).
{ $dayOfWeek: <dateExpression> }
Returns the day of the year for a date as a number between 1 and 366 (leap year).
{ $dayOfYear: <dateExpression> }
Returns the hour for a date as a number between 0 and 23.
{ $hour: <dateExpression> }
Returns the milliseconds of a date as a number between 0 and 999.
{ $millisecond: <dateExpression> }
Returns the minute for a date as a number between 0 and 59.
{ $minute: <dateExpression> }
Returns the month for a date as a number between 1 (January) and 12 (December).
{ $month: <dateExpression> }
Returns the seconds for a date as a number between 0 and 60 (leap seconds).
{ $second: <dateExpression> }
Returns the week number for a date as a number between 0 (the partial week that precedes the first Sunday of the year) and 53 (leap year).
{ $week: <dateExpression> }
Returns the year for a date as a number (e.g. 2014).
{ $year: <dateExpression> }
支持数据类型：
{ $dayOfMonth: new Date("2016-01-01") }返回1
{ $dayOfMonth: { date: new Date("Jan 7, 2003") } }返回7
{ $dayOfMonth: {
    date: new Date("August 14, 2011"),
    timezone: "America/Chicago"
} }返回14
{ $dayOfMonth: ISODate("1998-11-07T00:00:00Z") }返回7
{ $dayOfMonth: {
    date: ISODate("1998-11-07T00:00:00Z"),
    timezone: "-0400"å
} }返回6
{ $dayOfMonth: "March 28, 1976" }返回error
{ $dayOfMonth: Date("2016-01-01") }返回error
{ $dayOfMonth: "2009-04-09" }返回error
实例：
db.sales.insert({
  "_id" : 1,
  "item" : "abc",
  "price" : 10,
  "quantity" : 2,
  "date" : ISODate("2014-01-01T08:15:39.736Z")
}
)
db.sales.aggregate(
   [
     {
       $project:
         {
           year: { $year: "$date" },
           month: { $month: "$date" },
           day: { $dayOfMonth: "$date" },
           hour: { $hour: "$date" },
           minutes: { $minute: "$date" },
           seconds: { $second: "$date" },
           milliseconds: { $millisecond: "$date" },
           dayOfYear: { $dayOfYear: "$date" },
           dayOfWeek: { $dayOfWeek: "$date" },
           week: { $week: "$date" }
         }
     }
   ]
)

$divide(aggregation)
含义：
将一个数字除以另一个数字并返回结果，传递一个数组给$divide操作。
Divides one number by another and returns the result.Pass the arguments to $divide in an array.
语法：
{ $divide: [ <expression1>, <expression2> ] }
实例：
db.planning.insertMany([
{ "_id" : 1, "name" : "A", "hours" : 80, "resources" : 7 },
{ "_id" : 2, "name" : "B", "hours" : 40, "resources" : 4 }
])
db.planning.aggregate(
   [
     { $project: { name: 1, workdays: { $divide: [ "$hours", 8 ] } } }
   ]
)

$eq(aggregation)
含义：
Compares two values and returns:
true when the values are equivalent.
false when the values are not equivalent.
语法：
{ $eq: [ <expression1>, <expression2> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            qty: 1,
            qtyEq250: { $eq: [ "$qty", 250 ] },
            _id: 0
          }
     }
   ]



$filter(aggregation)
含义：
根据指定的条件选择并返回一个数组的子集。返回与条件匹配的元素所组成的数组。返回的元素是按元素顺序排列。
Selects a subset of an array to return based on the specified condition. Returns an array with only those elements that match the condition. The returned elements are in the original order.
语法：
{ $filter: { input: <array>, as: <string>, cond: <expression> } }
input
一个可以解析为数组的表达式
An expression that resolves to an array.
as
可选项。为指向子数组的指针命名。如果没有指定名称，这个变量默认为this。
Optional. A name for the variable that represents each individual element of the input array. If no name is specified, the variable name defaults to this.
cond    
一个解析为布尔值的表达式，用于确定元素是否应包含在输出数组中。该表达式使用as指定变量名称。
An expression that resolves to a boolean value used to determine if an element should be included in the output array. The expression references each element of the input array individually with the variable name specified in as.
实例：
db.sales.insertMany([
{_id: 0,items: [{ item_id: 43, quantity: 2, price: 10 },{ item_id: 2, quantity: 1, price: 240 }]},
{_id: 1,items: [{ item_id: 23, quantity: 3, price: 110 },{ item_id: 103, quantity: 4, price: 5 },{ item_id: 38, quantity: 1, price: 300 }]},
{_id: 2,items: [{ item_id: 4, quantity: 1, price: 23 }]}
])
db.sales.aggregate([
   {
      $project: {
         items: {
            $filter: {
               input: "$items",
               as: "item",
               cond: { $gte: [ "$$item.price", 100 ] }
            }
         }
      }
   }
])

$first(aggregation)
含义：
返回group所划分的全部文档中按顺序排列的第一个文档，只有当文档按照定义的顺序时才有意义。
Returns the value that results from applying an expression to the first document in a group of documents that share the same group by key.Only meaningful when documents are in a defined order.
$first只有在$group阶段才有效
$first is only available in the $group stage.
语法：
{ $first: <expression> }
实例：
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") },
{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") },
{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:05:00Z") },
{ "_id" : 6, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-15T12:05:10Z") },
{ "_id" : 7, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T14:12:12Z") }
])
db.sales.aggregate(
   [
     { $sort: { item: 1, date: 1 } },
     {
       $group:
         {
           _id: "$item",
           firstSalesDate: { $first: "$date" }
         }
     }
   ]
)

$floor(aggregation)
含义：
返回小于或等于指定数字的最大整数。
Returns the largest integer less than or equal to the specified number.
语法：
{ $floor: <number> }
实例：
db.samples.isnertMany([
{ _id: 1, value: 9.25 },
{ _id: 2, value: 8.73 },
{ _id: 3, value: 4.32 },
{ _id: 4, value: -5.34 }
])
db.samples.aggregate([
   { $project: { value: 1, floorValue: { $floor: "$value" } } }
])

$gt(aggregation)
含义：
Compares two values and returns:
true when the first value is greater than the second value.
false when the first value is less than or equivalent to the second value.
语法：
{ $gt: [ <expression1>, <expression2> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            qty: 1,
            qtyGt250: { $gt: [ "$qty", 250 ] },
            _id: 0
          }
     }
   ]
)

$gte(aggregation)
含义：
Compares two values and returns:
true when the first value is greater or equivalent than the second value.
false when the first value is less than to the second value.
语法：
{ $gte: [ <expression1>, <expression2> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            qty: 1,
            qtyGt250: { $gte: [ "$qty", 250 ] },
            _id: 0
          }
     }
   ]
)

$ifNull(aggregation)
含义：
计算表达式并且如果表达计算结果为非空值时返回表达式的值。如果表达式求值为空值（包括未定义字段或字段不存在）则返回替换表达式的值。
Evaluates an expression and returns the value of the expression if the expression evaluates to a non-null value.If the expression evaluates to a null value，including instances of undefined values or missing fields，returns the value of the replacement expression.
语法：
{ $ifNull: [ <expression>, <replacement-expression-if-null> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: null, qty: 200 },
{ "_id" : 3, "item" : "xyz1", qty: 250 }
])
db.inventory.aggregate(
   [
      {
         $project: {
            item: 1,
            description: { $ifNull: [ "$description", "Unspecified" ] }
         }
      }
   ]
)

$in(aggregation)
含义：
返回一个boolean来指示指定的值是否存在于数组中
Returns a boolean indicating whether a specified value is in an array.
语法：
{ $in: [ <expression>, <array expression> ] }
{ $in: [ 2, [ 1, 2, 3 ] ] }返回true
{ $in: [ "abc", [ "xyz", "abc" ] ] }返回true
{ $in: [ "xy", [ "xyz", "abc" ] ] }返回false
{ $in: [ [ "a" ], [ "a" ] ] }返回false
{ $in: [ [ "a" ], [ [ "a" ] ] ] }返回true
{ $in: [ /^a/, [ "a" ] ] }返回false
{ $in: [ /^a/, [ /^a/ ] ] }返回true
实例：
db.fruit.insertMany([
{ "_id" : 1, "location" : "24th Street”, "in_stock" : [ "apples", "oranges", "bananas" ] },
{ "_id" : 2, "location" : "36th Street”, "in_stock" : [ "bananas", "pears", "grapes" ] },å
{ "_id" : 3, "location" : "82nd Street”, "in_stock" : [ "cantaloupes", "watermelons", "apples" ] }
])
db.fruit.aggregate([
  {
    $project: {
      "store location" : "$location",
      "has bananas" : {
        $in: [ "bananas", "$in_stock" ]
      }
    }
  }
])

$indexOfArray(aggregation)
含义：
在数组中查询指定值出现的位置，并且返回这个值第一次出现的数组序列。如果这个值没有发现，返回-1。
Searches an array for an occurence of a specified value and returns the array index(zero-based) of the first occurence.If the value is not found,returns -1.
语法：
{ $indexOfArray: [ <array expression>, <search expression>, <start>, <end> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "items" : ["one", "two", "three"] },
{ "_id" : 2, "items" : [1, 2, 3] },
{ "_id" : 3, "items" : [null, null, 2] },
{ "_id" : 4, "items" : null },
{ "_id" : 5, "amount" : 3 }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            index: { $indexOfArray: [ "$items", 2 ] },
          }
      }
   ]
)

$indexOfBytes(aggregation)
含义：
搜索字符串以查找子字符串，并返回第一个出现的UTF-8字节索引（从零开始）。如果未查询到子字符串，则返回-1.
Searches a string for an occurence of a substring and returns the UTF-8 byte index(zero-based) of the first occurence.If the substring is not found,returns -1.
语法：
{ $indexOfBytes: [ <string expression>, <substring expression>, <start>, <end> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "foo" },
{ "_id" : 2, "item" : "fóofoo" },
{ "_id" : 3, "item" : "the foo bar" },
{ "_id" : 4, "item" : "hello world fóo" },
{ "_id" : 5, "item" : null },
{ "_id" : 6, "amount" : 3 }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            byteLocation: { $indexOfBytes: [ "$item", "foo" ] },
          }
      }
   ]
)

$isArray(aggregation)
含义：
确定操作的是否是一个数组，返回一个布尔值。
Determines if the operand is an array.Returns a boolean.
语法：
{ $isArray: [ <expression> ] }
实例：
db.warehouses.insertMany([
{ "_id" : 1, instock: [ "chocolate" ], ordered: [ "butter", "apples" ] },
{ "_id" : 2, instock: [ "apples", "pudding", "pie" ] },
{ "_id" : 3, instock: [ "pears", "pecans"], ordered: [ "cherries" ] },
{ "_id" : 4, instock: [ "ice cream" ], ordered: [ ] }
])
db.warehouses.aggregate([
   { $project:
      { items:
          { $cond:
            {
              if: { $and: [ { $isArray: "$instock" }, { $isArray: "$ordered" } ] },
              then: { $concatArrays: [ "$instock", "$ordered" ] },
              else: "One or more fields is not an array."
            }
          }
      }
   }
])

$isoDayOfWeek(aggregation)
含义：
以ISO 8601的格式返回星期号码，范围从1（星期一）到7（星期天）。
Returns the weekday number in ISO 8601 format, ranging from 1 (for Monday) to 7 (for Sunday).
语法：
{ $isoDayOfWeek: <dateExpression> }
实例：
db.birthdays.insertMany([
{ "_id" : 1, "name" : "Betty", "birthday" : ISODate("1993-09-21T00:00:00Z") },
{ "_id" : 2, "name" : "Veronica", "birthday" : ISODate("1981-11-07T00:00:00Z") }
])
db.dates.aggregate( [
  {
    $project: {
      _id: 0,
      name: "$name",
      dayOfWeek: { $isoDayOfWeek: "$birthday" }
    }
  }
] )

$isoWeek(aggregation)
含义：
返回ISO 8601格式的范围从1到53星期数，星期数从1开始，包含年份的第一个星期的第一个星期四。
Returns the week number in ISO 8601 format, ranging from 1 to 53. Week numbers start at 1 with the week (Monday through Sunday) that contains the year’s first Thursday.
语法：
{ $isoWeek: <dateExpression> }
{ $isoWeek: { date: new Date("Jan 4, 2016") } }返回1
{ $isoWeek: new Date("2016-01-01") }返回53
{ $isoWeek: {date: new Date("August 14, 2011"),timezone: "America/Chicago"} }返回32
{ $isoWeek: ISODate("1998-11-02T00:00:00Z") }返回45
{ $isoWeek: {date: ISODate("1998-11-02T00:00:00Z"),timezone: "-0500"} }返回44
{ $isoWeek: "March 28, 1976" }返回error
{ $isoWeek: Date("2016-01-01") }返回error
{ $isoWeek: "2009-04-09" }返回error
实例：
db.deliveries.insertMany([
{ "_id" : 1, "date" : ISODate("2006-10-24T00:00:00Z"), "city" : "Boston" },
{ "_id" : 2, "date" : ISODate("2011-08-18T00:00:00Z"), "city" : "Detroit" }
])
db.deliveries.aggregate( [
  {
    $project: {
      _id: 0,
      city: "$city",
      weekNumber: { $isoWeek: "$date" }
    }
  }
] )

$isoWeekYear(aggregation)
含义：
返回ISO8601格式的年份编号。这一年从第一周的周一开始，到最后一周的周末为止。
Returns the year number in ISO 8601 format. The year starts with the Monday of week 1 and ends with the Sunday of the last week.
语法：
{ $isoWeekYear: <dateExpression> }
{ $isoWeekYear: new Date("2015-05-26") }2015
{ $isoWeekYear: { date: new Date("Jan 7, 2003") } }2003
{ $isoWeekYear: ISODate("2017-01-02T00:00:00Z") }2017
{ $isoWeekYear: {date: ISODate("2017-01-02T00:00:00Z"),timezone: "-0500"} }2016
{ $isoWeekYear: {date: new Date("April 08, 2024"),timezone: "America/Chicago"} }2024
{ $isoWeekYear: "March 28, 1976" }error
{ $isoWeekYear: Date("2016-01-01") }error
{ $isoWeekYear: "2009-04-09" }error
实例：
db.anniversaries.isnertMany([
{ "_id" : 1, "date" : ISODate("2016-01-01T00:00:00Z") },
{ "_id" : 2, "date" : ISODate("2016-01-04T00:00:00Z") },
{ "_id" : 3, "date" : ISODate("2015-01-01T00:00:00Z") },
{ "_id" : 4, "date" : ISODate("2014-04-21T00:00:00Z") }
])
db.anniversaries.aggregate( [
  {
    $project: {
      yearNumber: { $isoWeekYear: "$date" }
    }
  }
] )

$last(aggregation)
含义：
返回分组文档所含括的全部文档中的最后一个文档。仅在文档进行排序时，此功能才有意义。
Returns the value that results from applying an expression to the last document in a group of documents that share the same group by a field.Only meaningful when documents are in a defined order.
语法：
{ $last: <expression> }
实例：
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "date" : ISODate("2014-01-01T08:00:00Z"), "price" : 10, "quantity" : 2 }
{ "_id" : 2, "item" : "jkl", "date" : ISODate("2014-02-03T09:00:00Z"), "price" : 20, "quantity" : 1 },
{ "_id" : 3, "item" : "xyz", "date" : ISODate("2014-02-03T09:05:00Z"), "price" : 5, "quantity" : 5 },
{ "_id" : 4, "item" : "abc", "date" : ISODate("2014-02-15T08:00:00Z"), "price" : 10, "quantity" : 10 },
{ "_id" : 5, "item" : "xyz", "date" : ISODate("2014-02-15T09:05:00Z"), "price" : 5, "quantity" : 10 },
{ "_id" : 6, "item" : "xyz", "date" : ISODate("2014-02-15T12:05:10Z"), "price" : 5, "quantity" : 5 },
{ "_id" : 7, "item" : "xyz", "date" : ISODate("2014-02-15T14:12:12Z"), "price" : 5, "quantity" : 10 }
])
db.sales.aggregate(
   [
     { $sort: { item: 1, date: 1 } },
     {
       $group:
         {
           _id: "$item",
           lastSalesDate: { $last: "$date" }
         }
     }
   ]
)

$let(aggregation)
含义：
为使用的指定表达式绑定变量，并返回表达式的结果。
Binds variables for use in the specified expression,and returns the result of the expression.
语法：
{
  $let:
     {
       vars: { <var1>: <expression>, ... },
       in: <expression>
     }
}
vars:
在in表达式中未可访问的变量分配块空间。要声明一个变量，指定一个字符串变量名并为这个值声明一个字段表达式。
Assignment block for the variables accessible in the in expression.To assign a variable,specify a string for the variable name and assign a valid expression for the value.
变量赋值在表达式之外没有意义，甚至在变量块本身内部也没有意义。
The variable assignments have no meaning outside the in expression,not even within the vars block itself
in:
The expression to evaluate.
注意：
1.$$:
要访问聚合表达式内的变量，使用两个$$作为变量名的前缀并且用冒号括起来。
To access variables in aggregation expressions,prefix the variable name with double dollar signs ($$) and enclosed in quotes.
$let可以访问定义在表达式块外的变量，包括系统变量。
$let can access variables defined outside its expression block,including system variables.
如果你在vars代码块中修改在外面定义的变量，这个新的变量值仅在in表达式模块内有效。在表达式外变量保留其以前的值。
If you modify the values of externally defined variables in the vars block, the new values take effect only in the in expression.Outside of the  in expression, the variables retain their previous values.
在vars声明代码块中，赋值的顺序并不重要，变量赋值仅在in表达式内才有意义。因此，访问变量赋值块中的变量值在变量块之外定义的变量的值，而不是在同一变量块内。
In the vars assignment block,the order of the assignment does not matter,and the variable assignments only have meaning inside the in expression.As such,accessing a variable’s value in the vars assignment block refers to the value of the variable defined outside the vars block and not inside the same vars block.
实例：
db.sales.insertMany([
{ _id: 1, price: 10, tax: 0.50, applyDiscount: true },
{ _id: 2, price: 10, tax: 0.25, applyDiscount: false }
])
db.sales.aggregate( [
   {
      $project: {
         finalTotal: {
            $let: {
               vars: {
                  total: { $add: [ '$price', '$tax' ] },
                  discounted: { $cond: { if: '$applyDiscount', then: 0.9, else: 1 } }
               },
               in: { $multiply: [ "$$total", "$$discounted" ] }
            }
         }
      }
   }
] )

$literal(aggregation)
含义：
返回一个没有解析的值，用于聚合管道中可以解析为表达式的值。
Returns a value without parsing. Use for values that the aggregation pipeline may interpret as an expression.
语法：
{ $literal: <value> }
{ $literal: { $add: [ 2, 3 ] } }         
{ "$add" : [ 2, 3 ] }
{ $literal:  { $literal: 1 } }         
{ "$literal" : 1 }
实例：
1.Treat $ as a Literal
db.records.insertMany([
{ "_id" : 1, "item" : "abc123", price: "$2.50" },
{ "_id" : 2, "item" : "xyz123", price: "1" },
{ "_id" : 3, "item" : "ijk123", price: "$1" }
])
db.records.aggregate( [
   { $project: { costsOneDollar: { $eq: [ "$price", { $literal: "$1" } ] } } }
] )
2.Project a new Field with Value1
db.bids.isnertMany([
{ "_id" : 1, "item" : "abc123", condition: "new" },
{ "_id" : 2, "item" : "xyz123", condition: "new" }
])
db.bids.aggregate( [
   { $project: { item: 1, startAt: { $literal: 1 } } }
] )

$ln(aggregation)
含义：
计算一个数的自然对数ln并将结果返回为double。
Calculates the natural logarithm ln (i.e loge) of a number and returns the result as a double.
语法：
{ $ln: <number> }
{ $ln: 1 }    
0
{ $ln: Math.E }
1
{ $ln: 10  }    
2.302585092994046
实例：
db.sales.isnertMany([
{ _id: 1, year: "2000", sales: 8700000 },
{ _id: 2, year: "2005", sales: 5000000 },
{ _id: 3, year: "2010", sales: 6250000 }
])
db.sales.aggregate( [ { $project: { x: "$year", y: { $ln: "$sales"  } } } ] )

$log(aggregation)
含义：
计算指定底数的对数，并返回一个double值。
Calculates the log of a number in the specified base and returns the result as a double.
语法：
{ $log: [ <number>, <base> ] }
{ $log: [ 100, 10 ] }2
{ $log: [ 100, Math.E ] }4.605170185988092
实例：
db.examples.insertMany([
{ _id: 1, positiveInt: 5 },
{ _id: 2, positiveInt: 2 },
{ _id: 3, positiveInt: 23 },
{ _id: 4, positiveInt: 10 }
])
db.examples.aggregate([
   { $project: { bitsNeeded:
      {
         $floor: { $add: [ 1, { $log: [ "$positiveInt", 2 ] } ] } } }
      }
])

$log10(aggregation)
含义：
计算一个以10为的底的对数，返回double
Calculates the log base 10 of a number and returns the result as a double.
语法：
{ $log10: <number> }
{ $log10: 1 }    0
{ $log10: 10 }    1
{ $log10: 100 }    2
{ $log10: 1000 }    3
实例：
db.samples.insertMany([
{ _id: 1, H3O: 0.0025 },
{ _id: 2, H3O: 0.001 },
{ _id: 3, H3O: 0.02 }
])
db.samples.aggregate( [
   { $project: { pH: { $multiply: [ -1, { $log10: "$H3O" } ] } } }
] )

$lt(aggregation)
含义：
比较两个值：
第一个小于第二个true
第一个大于或等于第二个false
Compares two values and returns:
true when the first value is less than the second value.
false when the first value is greater than or equivalent to the second value.
语法：
{ $lt: [ <expression1>, <expression2> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            qty: 1,
            qtyLt250: { $lt: [ "$qty", 250 ] },
            _id: 0
          }
     }
   ]
)

$lte(aggregation)
含义：
比较两个值：
第一个小于或等于第二个true
第一个大于第二个false
Compares two values and returns:
true when the first value is less than or equivalent to the second value.
false when the first value is greater than the second value.
语法：
{ $lte: [ <expression1>, <expression2> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            qty: 1,
            qtyLt250: { $lte: [ "$qty", 250 ] },
            _id: 0
          }
     }
   ]
)

$map(aggregation)
含义：
将一个表达式应用于数组中的每一个项目，并返回一个包含应用结果的数组。
Applies an expression to each item in an array and returns an array with the applied results.
语法：
{ $map: { input: <expression>, as: <string>, in: <expression> } }
input:
一个可以解析为数组的表达式。
An expression that resolves to an array.
as:
可选项。一个代表输入数组中每个独立的元素的变量名称。如果没有指定特定的名称，默认使用this。
Optional.A name for the variable that represents each individual element of the input array.If no name is specified,the variable name defaults to this.
in:
一个应用于输入数组每个独立元素的表达式。这个表达式使用as中指定的变量名分别引用每个元素。
An expression that is applied to each element of the input array.The expression references each element individually with the variable name specified in as.
实例：
1.使用$map添加数组中的每个元素
db.grades.insertMany([
{ _id: 1, quizzes: [ 5, 6, 7 ] },
{ _id: 2, quizzes: [ ] },
{ _id: 3, quizzes: [ 3, 8, 9 ] }
])
db.grades.aggregate(
   [
      { $project:
         { adjustedGrades:
            {
              $map:
                 {
                   input: "$quizzes",
                   as: "grade",
                   in: { $add: [ "$$grade", 2 ] }
                 }
            }
         }
      }
   ]
)
2.使用$map截断每个数组元素的尾数
db.deliveries.insertMany([
{ "_id" : 1, "city" : "Bakersfield", "distances" : [ 34.57, 81.96, 44.24 ] },
{ "_id" : 2, "city" : "Barstow", "distances" : [ 73.28, 9.67, 124.36 ] },
{ "_id" : 3, "city" : "San Bernadino", "distances" : [ 16.04, 3.25, 6.82 ] }
])
db.deliveries.aggregate(
   [
      { $project:
         {  city: "$city",
            integerValues:
               { $map:
                  {
                     input: "$distances",
                     as: "integerValue",
                     in: { $trunc: "$$integerValue" }
                  }
            }
         }
      }
   ]
)
3.在管道中使用两个$map
db. temperatures.insertMany([
{ "_id" : 1, "date" : "June 23", "temps" : [ 4, 12, 17 ] },
{ "_id" : 2, "date" : "July 7", "temps" : [ 14, 24, 11 ] },
{ "_id" : 3, "date" : "October 30", "temps" : [ 18, 6, 8 ] }
])
db.temperatures.aggregate(
   [
      { $project:
         {  _id: 0,
            date: "$date",
            tempsStep1:
               { $map:
                  {
                     input: "$temps",
                     as: "tempInCelsius",
                     in: { $multiply: [ "$$tempInCelsius", 9/5 ] }
                  }
               }
         }
      },
      { $project:
         {  date: "$date",
            "temps in Fahrenheit":
               { $map:
                  {
                     input: "$tempsStep1",
                     as: "tempStep1",
                     in: { $add: [ "$$tempStep1", 32 ] }
                  }
               }
         }
      }
   ]
)
与$let毛区别

$max(aggregation)
含义：
Returns the maximum value.
语法：
{ $max: <expression> }
{ $max: [ <expression1>, <expression2> ... ]  }
实例：
1.Use in $group Stage
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") },
{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") },
{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:05:00Z") }
])
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: "$item",
           maxTotalAmount: { $max: { $multiply: [ "$price", "$quantity" ] } },
           maxQuantity: { $max: "$quantity" }
         }
     }
   ]
)
2.Use in $project Stage
db.students.insertMany([
{ "_id": 1, "quizzes": [ 10, 6, 7 ], "labs": [ 5, 8 ], "final": 80, "midterm": 75 },
{ "_id": 2, "quizzes": [ 9, 10 ], "labs": [ 8, 8 ], "final": 95, "midterm": 80 },
{ "_id": 3, "quizzes": [ 4, 5, 5 ], "labs": [ 6, 5 ], "final": 78, "midterm": 70 }
])
db.students.aggregate([
   {
     $project: {
       quizMax: { $max: "$quizzes"},
       labMax: { $max: "$labs" },
       examMax: { $max: [ "$final", "$midterm" ] }
     }
   }
])

$mergeObjects(aggregation)
含义：
将多个文档合并成一个文档
Combines multiple documents into a single document
语法：
{ $mergeObjects: <document> }
{ $mergeObjects: [ <document1>, <document2>, ... ] }
{ $mergeObjects: [ { a: 1 }, null ] }
{ a: 1 }
{ $mergeObjects: [ null, null ] }
{ }
{ $mergeObjects: [ { a: 1 }, { a: 2, b: 2 }, { a: 3, c: 3 } ] }
{ a: 3, b: 2, c: 3 }
{ $mergeObjects: [ { a: 1 }, { a: 2, b: 2 }, { a: 3, b: null, c: 3 } ] }
{ a: 3, b: null, c: 3 }
注意：
1.mergeObjects忽略null操作符。如果全部操作符都被mergeObjects解析为null，mergeObjects返回一个空文档{}。
mergeObjects ignores null operands.If all the operands to mergeObjects resolves to null,mergeObjects returns an empty document{}.
2.mergeObjects会在合并文档时覆盖字段值。如果要合并的文档包含相同的字段名称，则生成的文档中的字段具有合并字段的最后一个文档的值。
mergeObjects overwrites the field values as it merges the documents.If documents to merge include the same field name, the field,in the resulting document, has the value from the last document merged for the field.
实例：
1.$mergeObjects
db.orders.insert([
  { "_id" : 1, "item" : "abc", "price" : 12, "ordered" : 2 },
  { "_id" : 2, "item" : "jkl", "price" : 20, "ordered" : 1 }
])
db.items.insert([
  { "_id" : 1, "item" : "abc", description: "product 1", "instock" : 120 },
  { "_id" : 2, "item" : "def", description: "product 2", "instock" : 80 },
  { "_id" : 3, "item" : "jkl", description: "product 3", "instock" : 60 }
])
db.orders.aggregate([
   {
      $lookup: {
         from: "items",
         localField: "item",    // field in the orders collection
         foreignField: "item",  // field in the items collection
         as: "fromItems"
      }
   },
   {
      $replaceRoot: { newRoot: { $mergeObjects: [ { $arrayElemAt: [ "$fromItems", 0 ] }, "$$ROOT" ] } }
   },
   { $project: { fromItems: 0 } }
])
2.$mergeObjects as an Accumulator
db.sales.insert( [
   { _id: 1, year: 2017, item: "A", quantity: { "2017Q1": 500, "2017Q2": 500 } },
   { _id: 2, year: 2016, item: "A", quantity: { "2016Q1": 400, "2016Q2": 300, "2016Q3": 0, "2016Q4": 0 } } ,
   { _id: 3, year: 2017, item: "B", quantity: { "2017Q1": 300 } },
   { _id: 4, year: 2016, item: "B", quantity: { "2016Q3": 100, "2016Q4": 250 } }
] )
db.sales.aggregate( [
   { $group: { _id: "$item", mergedSales: { $mergeObjects: "$quantity" } } }
])

$meta(aggregation)
含义：
返回元数据
Returns the metadata associated with a document in a pipeline operations, e.g. "textScore" when performing text search.
语法：
{ $meta: <metaDataKeyword> }
textScore:
Returns the score associated with the corresponding $text query for each matching document. The text score signifies how well the document matched the search term or terms. If not used in conjunction with a $text query, returns a score of null.    
注意：
The { $meta: "textScore" } expression is the only expression that the $sort stage accepts.

Although available for use everywhere expressions are accepted in the pipeline, the { $meta: "textScore" } expression is only meaningful in a pipeline that includes a $match stage with a $text query.
实例：
db. articles.insertMany([
{ "_id" : 1, "title" : "cakes and ale" },
{ "_id" : 2, "title" : "more cakes" },
{ "_id" : 3, "title" : "bread" },
{ "_id" : 4, "title" : "some cakes" }
])
db.articles.aggregate(
   [
     { $match: { $text: { $search: "cake" } } },
     { $group: { _id: { $meta: "textScore" }, count: { $sum: 1 } } }
   ]
)

$min(aggregation)
含义：
返回最小值。$min比较类型与值，使用指定的BSON比较顺序。
Returns the minimum value. $min compares both value and type, using the specified BSON comparison order for values of different types.
在3.2版本中更改：$min可用于$group与$project阶段。在以前的MongoDB版本中，$min仅可用于$group阶段。
Changed in version 3.2: $min is available in the $group and $project stages. In previous versions of MongoDB, $min is available in the $group stage only.
语法：
{ $min: <expression> }用于数组或$group同组中最小值
{ $min: [ <expression1>, <expression2> ... ]  }不接受数组
1.With a single expression as its operand, if the expression resolves to an array, $min traverses into the array to operate on the numerical elements of the array to return a single value.
2.With a list of expressions as its operand, if any of the expressions resolves to an array, $min does not traverse into the array but instead treats the array as a non-numerical value.
实例：
1.Use in $group Stage
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") },
{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") },
{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:05:00Z") }
])
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: "$item",
           minQuantity: { $min: "$quantity" }
         }
     }
   ]
)
2.Use in $project Stage
db.students.insertMany([
{ "_id": 1, "quizzes": [ 10, 6, 7 ], "labs": [ 5, 8 ], "final": 80, "midterm": 75 },
{ "_id": 2, "quizzes": [ 9, 10 ], "labs": [ 8, 8 ], "final": 95, "midterm": 80 },
{ "_id": 3, "quizzes": [ 4, 5, 5 ], "labs": [ 6, 5 ], "final": 78, "midterm": 70 }
])
db.students.aggregate([
   {
     $project: {
       quizMin: { $min: "$quizzes"},
       labMin: { $min: "$labs" },
       examMin: { $min: [ "$final", "$midterm" ] }
     }
   }
])

$mod(aggregation)
含义：
取余运算
Divides one number by another and returns the remainder.
语法：
{ $mod: [ <expression1>, <expression2> ] }
实例：
db.planning.insertMany([
{ "_id" : 1, "project" : "A", "hours" : 80, "tasks" : 7 },
{ "_id" : 2, "project" : "B", "hours" : 40, "tasks" : 4 }
])
db.planning.aggregate(
   [
     { $project: { remainder: { $mod: [ "$hours", "$tasks" ] } } }
   ]
)

$multiply(aggregation)
含义：
将数字相乘并返回结果
Multiplies numbers together and returns the result. Pass the arguments to $multiply in an array.
语法：
{ $multiply: [ <expression1>, <expression2>, ... ] }
实例：
db.sales.insertMany([{ "_id" : 1, "item" : "abc", "price" : 10, "quantity": 2, date: ISODate("2014-03-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity": 1, date: ISODate("2014-03-01T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity": 10, date: ISODate("2014-03-15T09:00:00Z") }
])
db.sales.aggregate(
   [
     { $project: { date: 1, item: 1, total: { $multiply: [ "$price", "$quantity" ] } } }
   ]
)

$ne(aggregation)
含义：
Compares two values and returns:
true when the values are not equivalent.
false when the values are equivalent.
语法：
{ $ne: [ <expression1>, <expression2> ] }
实例：
db.inventory.insertMany([{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            qty: 1,
            qtyNe250: { $ne: [ "$qty", 250 ] },
            _id: 0
          }
     }
   ]
)

$not(aggregation)
含义：
计算一个boolean，并返回与其相反的boolean值。当处理的表达式计算为true，$not返回false；当计算结果为false，返回true。
Evaluates a boolean and returns the opposite boolean value; i.e. when passed an expression that evaluates to true, $not returns false; when passed an expression that evaluates to false, $not returns true.
语法：
{ $not: [ <expression> ] }
{ $not: [ true ] }false
{ $not: [ [ false ] ] }false
{ $not: [ false ] }true
{ $not: [ null ] }true
{ $not: [ 0 ] }true
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            result: { $not: [ { $gt: [ "$qty", 250 ] } ] }
          }
     }
   ]
)

$objectToArray(aggregation)
含义：
转化一个文档为数组。返回的数组包含原文档中每个字段/值对应的元素。返回数组中的每个元素都是包含两个字段k与v的文档。
Converts a document to an array. The return array contains a element for each field/value pair in the original document. Each element in the return array is a document that contains two fields k and v:
The k field contains the field name in the original document.
The v field contains the value of the field in the original document.
语法：
{ $objectToArray: <object> }
{ $objectToArray: { item: "foo", qty: 25 } }    
[
   {"k" : "item","v" : "foo"},
   {"k" : "qty","v" : 25}
]
{ $objectToArray: {item: "foo",qty: 25,size: { len: 25, w: 10, uom: "cm" }} }
[
   {"k" : "item","v" : "foo"},
   {"k" : "qty","v" : 25},
   {"k" : "size","v" : {"len" : 25,"w" : 10,"uom" : "cm"}}
]
实例：
1.$objectToArray Example
db.inventory.insertMany([
{ "_id" : 1, "item" : "ABC1",  dimensions: { l: 25, w: 10, uom: "cm" } },
{ "_id" : 2, "item" : "ABC2",  dimensions: { l: 50, w: 25, uom: "cm" } },
{ "_id" : 3, "item" : "XYZ1",  dimensions: { l: 70, w: 75, uom: "cm" } }
])
db.inventory.aggregate(
   [
      {
         $project: {
            item: 1,
            dimensions: { $objectToArray: "$dimensions" }
         }
      }
   ]
)
2.$objectToArray to Sum Nested Fields
db.inventory.insertMany([
{ "_id" : 1, "item" : "ABC1", instock: { warehouse1: 2500, warehouse2: 500 } },
{ "_id" : 2, "item" : "ABC2", instock: { warehouse2: 500, warehouse3: 200} }
])
db.inventory.aggregate([
   { $project: { warehouses: { $objectToArray: "$instock" } } },
   { $unwind: "$warehouses" },
   { $group: { _id: "$warehouses.k", total: { $sum: "$warehouses.v" } } }
])
3.$objectToArray+$arrayToObject
db.inventory.insertMany([
{ "_id" : 1, "item" : "ABC1", instock: { warehouse1: 2500, warehouse2: 500 } },
{ "_id" : 2, "item" : "ABC2", instock: { warehouse2: 500, warehouse3: 200} }
])
db.inventory.aggregate( [
   { $addFields: { instock: { $objectToArray: "$instock" } } },
   { $addFields: { instock: { $concatArrays: [ "$instock", [ { "k": "total", "v": { $sum: "$instock.v" } } ] ] } } } ,
   { $addFields: { instock: { $arrayToObject: "$instock" } } }
] )

$or(aggregation)
含义：
计算一个或多个表达式，如果任何表达式为true，则返回true。否则，返回false。
Evaluates one or more expressions and returns true if any of the expressions are true.Otherwise,$or returns false.
语法：
{ $or: [ <expression1>, <expression2>, ... ] }
{ $or: [ true, false ] }         true
{ $or: [ [ false ], false ] }         true
{ $or: [ null, 0, undefined ] }         false
{ $or: [ ] }         false
注意：
$or使用短路逻辑：操作在遇到第一个true之后结束。
$or uses short-circuit logic:the operation shops evaluation after encountering the first true expression.
除了false波尔条件之外，$or计算结果为false的条件如下：null,0,undefined。$or评估其余全部的值为true，包含非零的数字和数组。
In addition to the false boolean value,$or evaluates as false the following:null,0,and undefined value.The $or evaluates all other values as true, including non-zero numeric values and arrays.
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            result: { $or: [ { $gt: [ "$qty", 250 ] }, { $lt: [ "$qty", 200 ] } ] }
          }
     }
   ]
)

$pow(aggregation)
含义：
给一个数字指定指数并返回结果。
Raises a number to the specified exponent and returns the result.
语法：
{ $pow: [ <number>, <exponent> ] }
实例：
{ $pow: [ 5, 0 ] }    1
{ $pow: [ 5, 2 ] }    25
{ $pow: [ 5, -2 ] }    0.04
{ $pow: [ -5, 0.5 ] }    NaN

$push(aggregation)
含义：
通过相同的key产生的分组文档，添加到一个数组中并返回。
Returns an array of all values that result from applying an expression to each document in a group of documents that share the same group by key.
$push只在$group中有效
$push is only available in the $group stage.
语法：
{ $push: <expression> }
实例：
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") },
{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") },
{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:05:00Z") },
{ "_id" : 6, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-15T12:05:10Z") },
{ "_id" : 7, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T14:12:12Z") }
])
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: { day: { $dayOfYear: "$date"}, year: { $year: "$date" } },
           itemsSold: { $push:  { item: "$item", quantity: "$quantity" } }
         }
     }
   ]
)

$range(aggregation)
含义：
返回一个数组，其元素是生成的数字序列。$range通过指定的步长持续递增，从start位置开始生成序列小于end值。
Returns an array whose elements are a generated sequence of numbers.$range generates the sequence from the specified starting number by successively incrementing the starting number by the specified step value up to but not including the end point.
语法：
{ $range: [ <start>, <end>, <non-zero step> ] }
{ $range: [ 0, 10, 2 ] }    [ 0, 2, 4, 6, 8 ]
{ $range: [ 10, 0, -2 ] }    [ 10, 8, 6, 4, 2 ]
{ $range: [ 0, 10, -2 ] }    [ ]
{ $range: [ 0, 5 ] }    [ 0, 1, 2, 3, 4 ]
实例：
db.distances.insertMany([
{ _id: 0, city: "San Jose", distance: 42 }
{ _id: 1, city: "Sacramento", distance: 88 }
{ _id: 2, city: "Reno", distance: 218 }
{ _id: 3, city: "Los Angeles", distance: 383 }
])
db.distances.aggregate([{
    $project: {
        _id: 0,
        city: 1,
        "Rest stops": { $range: [ 0, "$distance", 25 ] }
    }
}])

$reduce(aggregation)
含义：
将一个表达式应用于数组中的每一个元素，并将它们组合成为一个值。
Applies an expression to each element in an array and combines them into a single value.
语法：
{
    $reduce: {
        input: <array>,
        initialValue: <expression>,
        in: <expression>
    }
}
input<array>:
可以是解析为数组的任何有效表达式。
Can be any valid expression that resolves to an array.
如果参数解析为null，或者指向不存在的字段（undefined），$reduce返回null。
If the argument resolves to a value of null or refers to a missing field,$reduce returns null.
如果参数不能解析为数组或者null，也不算undefined，$reduce返回一个错误
If the argument does not resolve to an array or null nor refers to a missing field,$reduce returns an error.
initialValue<expression>:
在in之前设置的初始累计值应用于input数组中的第一个元素
The initial cumulative value set before in is applied to the first element of the input array.
in<expression>
A valid expression that $reduce applies to each element in the input array in left-to-right order.Wrap the input value with $reverseArray to yield the equivalent of applying the combining expression from right-to-left.
During evaluation of the in expression, two variables will be available:
value is the variable that represents the cumulative value of the expression.
this is the variable that refers to the element being processed.
注意：
If input resolves to an empty array,$reduce returns initialValue.
实例：
1.简单应用
{
   $reduce: {
      input: ["a", "b", "c"],
      initialValue: "",
      in: { $concat : ["$$value", "$$this"] }
    }
}
"abc"
{
   $reduce: {
      input: [ 1, 2, 3, 4 ],
      initialValue: { sum: 5, product: 2 },
      in: {
         sum: { $add : ["$$value.sum", "$$this"] },
         product: { $multiply: [ "$$value.product", "$$this" ] }
      }
   }
}
{ "sum" : 15, "product" : 48 }
{
   $reduce: {
      input: [ [ 3, 4 ], [ 5, 6 ] ],
      initialValue: [ 1, 2 ],
      in: { $concatArrays : ["$$value", "$$this"] }
   }
}
2.应用
db.probability.insertMany([
{_id:1, "type":"die", "experimentId":"r5", "description":"Roll a 5", "eventNum":1, "probability":0.16666666666667},
{_id:2, "type":"card", "experimentId":"d3rc", "description":"Draw 3 red cards", "eventNum":1, "probability":0.5},
{_id:3, "type":"card", "experimentId":"d3rc", "description":"Draw 3 red cards", "eventNum":2, "probability":0.49019607843137},
{_id:4, "type":"card", "experimentId":"d3rc", "description":"Draw 3 red cards", "eventNum":3, "probability":0.48},
{_id:5, "type":"die", "experimentId":"r16", "description":"Roll a 1 then a 6", "eventNum":1, "probability":0.16666666666667},
{_id:6, "type":"die", "experimentId":"r16", "description":"Roll a 1 then a 6", "eventNum":2, "probability":0.16666666666667},
{_id:7, "type":"card", "experimentId":"dak", "description":"Draw an ace, then a king", "eventNum":1, "probability":0.07692307692308},
{_id:8, "type":"card", "experimentId":"dak", "description":"Draw an ace, then a king", "eventNum":2, "probability":0.07843137254902}
])
db.probability.aggregate(
  [
    {
      $group: {
        _id: "$experimentId",
        "probabilityArr": { $push: "$probability" }
      }
    },
    {
      $project: {
        "description": 1,
        "results": {
          $reduce: {
            input: "$probabilityArr",
            initialValue: 1,
            in: { $multiply: [ "$$value", "$$this" ] }
          }
        }
      }
    }
  ]
)
3.
db.clothes.insertMany([
{ "_id" : 1, "productId" : "ts1", "description" : "T-Shirt", "color" : "black", "size" : "M", "price" : 20, "discounts" : [ 0.5, 0.1 ] },
{ "_id" : 2, "productId" : "j1", "description" : "Jeans", "color" : "blue", "size" : "36", "price" : 40, "discounts" : [ 0.25, 0.15, 0.05 ] },
{ "_id" : 3, "productId" : "s1", "description" : "Shorts", "color" : "beige", "size" : "32", "price" : 30, "discounts" : [ 0.15, 0.05 ] },
{ "_id" : 4, "productId" : "ts2", "description" : "Cool T-Shirt", "color" : "White", "size" : "L", "price" : 25, "discounts" : [ 0.3 ] },
{ "_id" : 5, "productId" : "j2", "description" : "Designer Jeans", "color" : "blue", "size" : "30", "price" : 80, "discounts" : [ 0.1, 0.25 ] }
])
db.clothes.aggregate(
  [
    {
      $project: {
        "discountedPrice": {
          $reduce: {
            input: "$discounts",
            initialValue: "$price",
            in: { $multiply: [ "$$value", { $subtract: [ 1, "$$this" ] } ] }
          }
        }
      }
    }
  ]
)
4.
db.people.insertMany([
{ "_id" : 1, "name" : "Melissa", "hobbies" : [ "softball", "drawing", "reading" ] },
{ "_id" : 2, "name" : "Brad", "hobbies" : [ "gaming", "skateboarding" ] },
{ "_id" : 3, "name" : "Scott", "hobbies" : [ "basketball", "music", "fishing" ] },
{ "_id" : 4, "name" : "Tracey", "hobbies" : [ "acting", "yoga" ] },
{ "_id" : 5, "name" : "Josh", "hobbies" : [ "programming" ] },
{ "_id" : 6, "name" : "Claire" }
])
db.people.aggregate(
   [
     // Filter to return only non-empty arrays
     { $match: { "hobbies": { $gt: [ ] } } },
     {
       $project: {
         "name": 1,
         "bio": {
           $reduce: {
             input: "$hobbies",
             initialValue: "My hobbies include:",
             in: {
               $concat: [
                 "$$value",
                 {
                   $cond: {
                     if: { $eq: [ "$$value", "My hobbies include:" ] },
                     then: " ",
                     else: ", "
                   }
                 },
                 "$$this"
               ]
             }
           }
         }
       }
     }
   ]
)

$reverseArray(aggregation)
含义：
接受一个数组表达式作为参数，并且返回一个顺序相反的数组。
Accepts an array expression as an argument and returns an array with the elements in reverse order.
语法：
{ $reverseArray: <array expression> }
{ $reverseArray: [ 1, 2, 3 ] }    
[ 3, 2, 1 ]
{ $reverseArray: { $slice: [ [ "foo", "bar", "baz", "qux" ], 1, 2 ] } }    
[ "baz", "bar" ]
{ $reverseArray: null }   
null
{ $reverseArray: [ ] }    
[ ]
{ $reverseArray: [ [ 1, 2, 3 ], [ 4, 5, 6 ] ] }    
[ [ 4, 5, 6 ], [ 1, 2, 3 ] ]
注意：
如果参数解析为空值或者引用缺失，则$reverseArray返回null。
If the argument resolves to a value of null or refers to a missing field,$reverseArray returns null.
如果参数不解析为数组或者null，也不引用缺失的字段，$reverseArray将返回一个错误。
If the argument does not resolve to an array or null nor refers to a missing field,$reverseArray returns an error.
如果参数是空数组，$reverseArray返回空数组。
$reverseArray returns an empty array when the argument is an empty array.
如果参数包含子数组，$reverseArray只操作顶层数组元素并且不会颠倒子数组的内部。
If the argument contains subarrays,$reverseArray only operates on the top level array elements and will not reverse the contents of subarrays.
实例：
db.users.insertMany([
{ "_id" : 1, "name" : "dave123", "favorites" : [ "chocolate", "cake", "butter", "apples" ] },
{ "_id" : 2, "name" : "li", "favorites" : [ "apples", "pudding", "pie" ] },
{ "_id" : 3, "name" : "ahn", "favorites" : [ ] },
{ "_id" : 4, "name" : "ty" }
])
db.users.aggregate([
   {
     $project:
      {
         name: 1,
         reverseFavorites: { $reverseArray: "$favorites" }
      }
   }
])

$setDifference(aggregation)
含义：
比较两个集合A与B，如果A与B相同则返回空数组，如果A中包含B中不存在的元素则返回该元素。
Takes two sets and returns an array containing the elements that only exist in the first set; i.e. performs a relative complement of the second set relative to the first.
语法：
{ $setDifference: [ <expression1>, <expression2> ] }
{ $setDifference: [ [ "a", "b", "a" ], [ "b", "a" ] ] }         [ ]
{ $setDifference: [ [ "a", "b" ], [ [ "a", "b" ] ] ] }         [ "a", "b" ]
实例：
db.experiments.insertMany([
{ "_id" : 1, "A" : [ "red", "blue" ], "B" : [ "red", "blue" ] },
{ "_id" : 2, "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ] },
{ "_id" : 3, "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ] },
{ "_id" : 4, "A" : [ "red", "blue" ], "B" : [ "green", "red" ] },
{ "_id" : 5, "A" : [ "red", "blue" ], "B" : [ ] },
{ "_id" : 6, "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ] },
{ "_id" : 7, "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ] },
{ "_id" : 8, "A" : [ ], "B" : [ ] },
{ "_id" : 9, "A" : [ ], "B" : [ "red" ] }
])
db.experiments.aggregate(
   [
     { $project: { A: 1, B: 1, inBOnly: { $setDifference: [ "$B", "$A" ] }, _id: 0 } }
   ]
)

$setEquals(aggregation)
含义：
比较两个或多个数组，如果它们具有相同的独立元素，则返回true，否则false。
Compares two or more arrays and returns true if they have the same distinct elements and false otherwise.
语法：
{ $setEquals: [ <expression1>, <expression2>, ... ] }
{ $setEquals: [ [ "a", "b", "a" ], [ "b", "a" ] ] }         true
{ $setEquals: [ [ "a", "b" ], [ [ "a", "b" ] ] ] }         false
实例：
db.experiments.insertMany([
{ "_id" : 1, "A" : [ "red", "blue" ], "B" : [ "red", "blue" ] },
{ "_id" : 2, "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ] },
{ "_id" : 3, "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ] },
{ "_id" : 4, "A" : [ "red", "blue" ], "B" : [ "green", "red" ] },
{ "_id" : 5, "A" : [ "red", "blue" ], "B" : [ ] },
{ "_id" : 6, "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ] },
{ "_id" : 7, "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ] },
{ "_id" : 8, "A" : [ ], "B" : [ ] },
{ "_id" : 9, "A" : [ ], "B" : [ "red" ] }
])
db.experiments.aggregate(
   [
     { $project: { A: 1, B: 1, sameElements: { $setEquals: [ "$A", "$B" ] }, _id: 0 } }
   ]
)

$setIntersection(aggregation)
含义：
传入一个或多个数组，返回一个包含在每个传入数组中都出现的元素的数组
Takes two or more arrays and returns an array that contains the elements that appear in every input array.
语法：
{ $setIntersection: [ <array1>, <array2>, ... ] }
{ $setIntersection: [ [ "a", "b", "a" ], [ "b", "a" ] ] }         [ "b", "a" ]
{ $setIntersection: [ [ "a", "b" ], [ [ "a", "b" ] ] ] }         [ ]
实例：
db.experiments.insertMany([
{ "_id" : 1, "A" : [ "red", "blue" ], "B" : [ "red", "blue" ] },
{ "_id" : 2, "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ] },
{ "_id" : 3, "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ] },
{ "_id" : 4, "A" : [ "red", "blue" ], "B" : [ "green", "red" ] },
{ "_id" : 5, "A" : [ "red", "blue" ], "B" : [ ] },
{ "_id" : 6, "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ] },
{ "_id" : 7, "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ] },
{ "_id" : 8, "A" : [ ], "B" : [ ] },
{ "_id" : 9, "A" : [ ], "B" : [ "red" ] }
])
db.experiments.aggregate(
   [
     { $project: { A: 1, B: 1, commonToBoth: { $setIntersection: [ "$A", "$B" ] }, _id: 0 } }
   ]
)

$setIsSubset(aggregation)
含义：
传入两个数组，如果第二个数组是第一个数组的子集（包括两个数组相同）返回true，反之返回false。
Takes two arrays and returns true when the first array is a subset of the second,including when the first array equals the second array,and false otherwise.
语法：
{ $setIsSubset: [ <expression1>, <expression2> ] }
{ $setIsSubset: [ [ "a", "b", "a" ], [ "b", "a" ] ] }         true
{ $setIsSubset: [ [ "a", "b" ], [ [ "a", "b" ] ] ] }         false
实例：
db.experiments.insertMany([
{ "_id" : 1, "A" : [ "red", "blue" ], "B" : [ "red", "blue" ] },
{ "_id" : 2, "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ] },
{ "_id" : 3, "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ] },
{ "_id" : 4, "A" : [ "red", "blue" ], "B" : [ "green", "red" ] },
{ "_id" : 5, "A" : [ "red", "blue" ], "B" : [ ] },
{ "_id" : 6, "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ] },
{ "_id" : 7, "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ] },
{ "_id" : 8, "A" : [ ], "B" : [ ] },
{ "_id" : 9, "A" : [ ], "B" : [ "red" ] }
])
db.experiments.aggregate(
   [
     { $project: { A:1, B: 1, AisSubset: { $setIsSubset: [ "$A", "$B" ] }, _id:0 } }
   ]
)

$setUnion(aggregation)
含义：
将两个或多个数组传入，返回一个包含出现在任何输入数组中的元素的数组。
Takes two or more arrays and returns an array containing the elements that appear in any input array.
语法：
{ $setUnion: [ <expression1>, <expression2>, ... ] }
{ $setUnion: [ [ "a", "b", "a" ], [ "b", "a" ] ] }         [ "b", "a" ]
{ $setUnion: [ [ "a", "b" ], [ [ "a", "b" ] ] ] }         [ [ "a", "b" ], "b", "a” ]
注意：
$setUnion对数组执行一个set操作，将数组视为集合。如果一个数组包含重复的值，$setUnion将会忽略这个重复的值。$setUnion忽略元素的顺序。
$setUnion performs set operation on arrays, treating arrays as sets. If an array contains duplicate entries, $setUnion ignores the duplicate entries. $setUnion ignores the order of the elements.
$setUnion在输出的结果数组中过滤重复元素，不指定数组中元素的顺序。
$setUnion filters out duplicates in its result to output an array that contain only unique entries. The order of the elements in the output array is unspecified.
如果一个集合包含内嵌元素，$setUnion不解析内嵌元素，而是将其视为顶层元素。
If a set contains a nested array element, $setUnion does not descend into the nested array but evaluates the array at top-level.
实例：
db.experiments.insertMany([
{ "_id" : 1, "A" : [ "red", "blue" ], "B" : [ "red", "blue" ] },
{ "_id" : 2, "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ] },
{ "_id" : 3, "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ] },
{ "_id" : 4, "A" : [ "red", "blue" ], "B" : [ "green", "red" ] },
{ "_id" : 5, "A" : [ "red", "blue" ], "B" : [ ] },
{ "_id" : 6, "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ] },
{ "_id" : 7, "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ] },
{ "_id" : 8, "A" : [ ], "B" : [ ] },
{ "_id" : 9, "A" : [ ], "B" : [ "red" ] }
])
db.experiments.aggregate(
   [
     { $project: { A:1, B: 1, allValues: { $setUnion: [ "$A", "$B" ] }, _id: 0 } }
   ]
)

$size(aggregation)
含义：
计算并返回数组子元素的数量。
Counts and returns the total the number of items in an array.
语法：
{ $size: <expression> }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "ABC1", "description" : "product 1", colors: [ "blue", "black", "red" ] },
{ "_id" : 2, "item" : "ABC2", "description" : "product 2", colors: [ "purple" ] },
{ "_id" : 3, "item" : "XYZ1", "description" : "product 3", colors: [ ] }
])
db.inventory.aggregate(
   [
      {
         $project: {
            item: 1,
            numberOfColors: { $size: "$colors" }
         }
      }
   ]
)

$slice(aggregation)
含义：
返回一部分数组
Returns a subset of an array.
语法：
{ $slice: [ <array>, <n> ] }
{ $slice: [ <array>, <position>, <n> ] }
<array>:
只要能解析为数组的任何有效的表达式。
Any valid expression as long as it resolves to an array.
<position>:
可选的。任何有效的表达式只要其能解析为int。
* 如果为正，则$slice决定从数组开始的位置开始计算。如果<position>比数组的元素总数还要大，$slice操作返回一个空数组。
* 如果为负，则$slice决定从数组结束的位置开始计算。如果<position>的绝对值比数组元素量大，则起始位置是数组的起始位置。
Optional.Any valid expression as long as it resolves to an integer.
* If positive,$slice determines the starting position from the start of the array.If <position> is greater than the number of elements,the $slice retruns an empty array.
* If negative,$slice determines the starting position from the end of the array.If the absolute value of the <position> is greater then the number of elements,the starting position is the start of the array.
<n>:
任何有效的表达式只要其是int类型。如果指定了<position>，则<n>必须解析为一个正整数
* 如果为正，$slice返回数组中的前n个元素。如果指定了<position>，则$slice返回数组从该位置开始的前n个元素。
* 如果为负，$slice返回数组中的后n个元素。如果指定了<position>,那么n不能解析为负数。
Any valid expression as long as it resolves to an integer.If <position> is specified,<n> must resolve to a positive integer.
* If positive,$slice returns up to the first n elements in the array.If the <position> is specified,$slice returns the first n elements starting from the position.
* If negative,$slice returns up to the last n elements in the array.n cannot resolve to a negative number if <position> is specified.
{ $slice: [ [ 1, 2, 3 ], 1, 1 ] }    [ 2 ]
{ $slice: [ [ 1, 2, 3 ], -2 ] }    [ 2, 3 ]
{ $slice: [ [ 1, 2, 3 ], 15, 2 ] }    [  ]
{ $slice: [ [ 1, 2, 3 ], -15, 2 ] }    [ 1, 2 ]
实例：
db.users.insertMany([
{ "_id" : 1, "name" : "dave123", favorites: [ "chocolate", "cake", "butter", "apples" ] },
{ "_id" : 2, "name" : "li", favorites: [ "apples", "pudding", "pie" ] },
{ "_id" : 3, "name" : "ahn", favorites: [ "pears", "pecans", "chocolate", "cherries" ] },
{ "_id" : 4, "name" : "ty", favorites: [ "ice cream" ] }
])
db.users.aggregate([
   { $project: { name: 1, threeFavorites: { $slice: [ "$favorites", 3 ] } } }
])

$split(aggregation)
含义：
通过一个界定符拆分一个字符串为数组。$split删除这个界定符并且返回结果被拆分的字符串到数组。如果界定符没有在字符串中找到。$split返回原字符串为数组的唯一元素。
Divides a string into an array of substrings based on a delimiter. $split removes the delimiter and returns the resulting substrings as elements of an array. If the delimiter is not found in the string, $split returns the original string as the only element of an array.
语法：
{ $split: [ <string expression>, <delimiter> ] }
{ $split: [ "June-15-2013", "-" ] }    [ "June", "15", "2013" ]
{ $split: [ "banana split", "a" ] }    [ "b", "n", "n", " split" ]
{ $split: [ "Hello World", " " ] }    [ "Hello", "World" ]
{ $split: [ "astronomical", "astro" ] }    [ "", "nomical" ]
{ $split: [ "pea green boat", "owl" ] }    [ "pea green boat" ]
{ $split: [ "headphone jack", 7 ] }    "$split requires an expression that evaluates to a string as a second argument, found: double"
{ $split: [ "headphone jack", /jack/ ] }    "$split requires an expression that evaluates to a string as a second argument, found: regex"
实例：
db.deliveries.insertMany([
{ "_id" : 1, "city" : "Berkeley, CA", "qty" : 648 },
{ "_id" : 2, "city" : "Bend, OR", "qty" : 491 },
{ "_id" : 3, "city" : "Kensington, CA", "qty" : 233 },
{ "_id" : 4, "city" : "Eugene, OR", "qty" : 842 },
{ "_id" : 5, "city" : "Reno, NV", "qty" : 655 },
{ "_id" : 6, "city" : "Portland, OR", "qty" : 408 },
{ "_id" : 7, "city" : "Sacramento, CA", "qty" : 574 }
])
db.deliveries.aggregate([
  { $project : { city_state : { $split: ["$city", ", "] }, qty : 1 } },
  { $unwind : "$city_state" },
  { $match : { city_state : /[A-Z]{2}/ } },
  { $group : { _id: { "state" : "$city_state" }, total_qty : { "$sum" : "$qty" } } },
  { $sort : { total_qty : -1 } }
]);

$sqrt(aggregation)
含义：
计算一个正数的平方根，并将结果作为双精度返回。
Calculates the square root of a positive number and returns the result as a double.
语法：
{ $sqrt: <number> }
{ $sqrt: 25 }    5
{ $sqrt: 30 }    5.477225575051661
{ $sqrt: null }    null
实例：
db.points.insertMany([
{ _id: 1, p1: { x: 5, y: 8 }, p2: { x: 0, y: 5} },
{ _id: 2, p1: { x: -2, y: 1 }, p2: { x: 1, y: 5} },
{ _id: 3, p1: { x: 4, y: 4 }, p2: { x: 4, y: 0} }
])
db.points.aggregate([
   {
     $project: {
        distance: {
           $sqrt: {
               $add: [
                  { $pow: [ { $subtract: [ "$p2.y", "$p1.y" ] }, 2 ] },
                  { $pow: [ { $subtract: [ "$p2.x", "$p1.x" ] }, 2 ] }
               ]
           }
        }
     }
   }
])

$strcasecmp(aggregation)
含义：
Performs case-insensitive comparison of two strings.Returns:
1 if first string is “greater than” the second string
0 if the two strings are equal.
-1 if the first string is “less than” the second string.
语法：
{ $strcasecmp: [ <expression1>, <expression2> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "product 1" },
{ "_id" : 2, "item" : "ABC2", quarter: "13Q4", "description" : "product 2" },
{ "_id" : 3, "item" : "XYZ1", quarter: "14Q2", "description" : null }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            comparisonResult: { $strcasecmp: [ "$quarter", "13q4" ] }
          }
      }
   ]
)

$strLenBytes(aggregation)
含义：
返回指定字符串中UTF-8编码字节数。
Returns the number of UTF-8 encoded bytes in the specified string.
语法：
{ $strLenBytes: <string expression> }
{ $strLenBytes: "abcde" }    5    Each character is encoded using one byte.
{ $strLenBytes: "Hello World!" }    12    Each character is encoded using one byte.
{ $strLenBytes: "cafeteria" }    9    Each character is encoded using one byte.
{ $strLenBytes: "cafétéria" }    11    é is encoded using two bytes.
{ $strLenBytes: "" }    0    Empty strings return 0.
{ $strLenBytes: "$€λG" }    7    € is encoded using three bytes. λ is encoded using two bytes.
{ $strLenBytes: "寿司" }    6    Each character is encoded using three bytes.
实例：
db.food.insertMany([
{ "_id" : 1, "name" : "apple" },
{ "_id" : 2, "name" : "banana" },
{ "_id" : 3, "name" : "éclair" },
{ "_id" : 4, "name" : "hamburger" },
{ "_id" : 5, "name" : "jalapeño" },
{ "_id" : 6, "name" : "pizza" },
{ "_id" : 7, "name" : "tacos" },å
{ "_id" : 8, "name" : "寿司" }
])
db.food.aggregate(
  [
    {
      $project: {
        "name": 1,
        "length": { $strLenBytes: "$name" }
      }
    }
  ]
)

$strLenCP(aggregation)
含义：
返回指定字符串中的UTF-8code points
Returns the number of UTF-8 code points in the specified string.
语法：
{ $strLenCP: <string expression> }
{ $strLenCP: "abcde" }    5
{ $strLenCP: "Hello World!" }    12
{ $strLenCP: "cafeteria" }    9
{ $strLenCP: "cafétéria" }    9
{ $strLenCP: "" }    0
{ $strLenCP: "$€λA" }    4
{ $strLenCP: "寿司" }    2
实例：
db.food.insertMany([
{ "_id" : 1, "name" : "apple" },
{ "_id" : 2, "name" : "banana" },
{ "_id" : 3, "name" : "éclair" },
{ "_id" : 4, "name" : "hamburger" },
{ "_id" : 5, "name" : "jalapeño" },
{ "_id" : 6, "name" : "pizza" },
{ "_id" : 7, "name" : "tacos" },
{ "_id" : 8, "name" : "寿司" }
])
db.food.aggregate(
  [
    {
      $project: {
        "name": 1,
        "length": { $strLenCP: "$name" }
      }
    }
  ]
)

$substr(aggregation)
含义：
返回一个字符串的子字符串，从指定的索引位置开始，包括指定数量的字符。该指数是从零开始的。
Returns a substring of a string, starting at a specified index position and including the specified number of characters. The index is zero-based.
语法：
{ $substr: [ <string>, <start>, <length> ] }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "product 1" },
{ "_id" : 2, "item" : "ABC2", quarter: "13Q4", "description" : "product 2" },
{ "_id" : 3, "item" : "XYZ1", quarter: "14Q2", "description" : null }
])
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            yearSubstring: { $substr: [ "$quarter", 0, 2 ] },
            quarterSubtring: { $substr: [ "$quarter", 2, -1 ] }
          }
      }
   ]
)

$substrCP(aggregation)
含义：
返回字符串的子字符串。这个子字符串起始从指定的位置开始计算到截取数量界限为终止。
Returns the substring of a string.The substring starts with the character at the specified UTF-8 code point(CP) index(zero-based) in the string for the number of code points specified.
语法
{ $substrCP: [ <string expression>, <code point index>, <code point count> ] }
{ $substrCP: [ "abcde", 1, 2 ] }    "bc"
{ $substrCP: [ "Hello World!", 6, 5 ] }    "World"
{ $substrCP: [ "cafétéria", 0, 5 ] }    "cafét"
{ $substrCP: [ "cafétéria", 5, 4 ] }    "tér"
{ $substrCP: [ "cafétéria", 7, 3 ] }    "ia"
{ $substrCP: [ "cafétéria", 3, 1 ] }    "é"
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "product 1" },
{ "_id" : 2, "item" : "ABC2", quarter: "13Q4", "description" : "product 2" },
{ "_id" : 3, "item" : "XYZ1", quarter: "14Q2", "description" : null }
])
db.inventory.aggregate(
  [
    {
      $project: {
        item: 1,
        yearSubstring: { $substrCP: [ "$quarter", 0, 2 ] },
        quarterSubtring: {
          $substrCP: [
            "$quarter", 2, { $subtract: [ { $strLenCP: "$quarter" }, 2 ] }
          ]
        }
      }
    }
  ]
)

$subtract(aggregation)
含义：
返回第一个参数减去第二个参数的差值。如果两个值都是数字返回它们的差值。如果两个值都是时间格式则返回它们相差的毫秒数。如果两个值一个是日期另一个是毫秒数返回减去毫秒的日期。如果两个值分别是日期与数字，那么只允许日期为第一个参数。
Returns the result of subtracting the second value from the first. If the two values are numbers, return the difference. If the two values are dates, return the difference in milliseconds. If the two values are a date and a number in milliseconds, return the resulting date. Accepts two argument expressions. If the two values are a date and a number, specify the date argument first as it is not meaningful to subtract a date from a number.
语法：
{ $subtract: [ <expression1>, <expression2> ] }
实例：
1.参数都是数字
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "fee" : 2, "discount" : 5, "date" : ISODate("2014-03-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "fee" : 1, "discount" : 2, "date" : ISODate("2014-03-01T09:00:00Z") }
])
db.sales.aggregate( [ { $project: { item: 1, total: { $subtract: [ { $add: [ "$price", "$fee" ] }, "$discount" ] } } } ] )
2.参数都是日期格式
db.sales.aggregate( [ { $project: { item: 1, dateDifference: { $subtract: [ new Date(), "$date" ] } } } ] )
3.日期减去毫秒
db.sales.aggregate( [ { $project: { item: 1, dateDifference: { $subtract: [ "$date", 5 * 60 * 1000 ] } } } ] )

$sum(aggregation)
含义：
计算并返回两个数字的值。$sum忽略非数字元素。
Calculates and returns the sum of numeric values.$sum ignores no-numeric value.
$sum在$group与$project阶段都有效。
$sum is avaliable in the $group and $project stages.
语法：
$group阶段
{ $sum: <expression> }
$project阶段
{ $sum: [ <expression1>, <expression2> ... ]  }
实例：
1.$group阶段使用
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") },
{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") },
{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:05:00Z") }
])
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: { day: { $dayOfYear: "$date"}, year: { $year: "$date" } },
           totalAmount: { $sum: { $multiply: [ "$price", "$quantity" ] } },
           count: { $sum: 1 }
         }
     }
   ]
)
使用sum计数
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: { day: { $dayOfYear: "$date"}, year: { $year: "$date" } },
           totalAmount: { $sum: "$qty" },
           count: { $sum: 1 }
         }
     }
   ]
)
2.$project阶段
db.students.insertMany([
{ "_id": 1, "quizzes": [ 10, 6, 7 ], "labs": [ 5, 8 ], "final": 80, "midterm": 75 },
{ "_id": 2, "quizzes": [ 9, 10 ], "labs": [ 8, 8 ], "final": 95, "midterm": 80 },
{ "_id": 3, "quizzes": [ 4, 5, 5 ], "labs": [ 6, 5 ], "final": 78, "midterm": 70 }
])
db.students.aggregate([
   {
     $project: {
       quizTotal: { $sum: "$quizzes"},
       labTotal: { $sum: "$labs" },
       examTotal: { $sum: [ "$final", "$midterm" ] }
     }
   }
])

$switch(aggregation)
含义：
评估一系列的条件表达式。当其与表达式匹配，$switch执行指定的表达式并且中断控制流。
Evaluates a series of case expressions. When it finds an expression which evaluates to true, $switch executes a specified expression and breaks out of the control flow.
语法：
$switch: {
   branches: [
      { case: <expression>, then: <expression> },
      { case: <expression>, then: <expression> },
      ...
   ],
   default: <expression>
}
branches:
    An array of control branch documents.Each branch is document with the following fields:
        case
        Can be any valid expression that resolves to a boolean.If the result is not a boolean, it is coerced to a boolean        value.More information about how MongoDB evaluates expressions as either true of false can be found here.
    then
        Can be any valid expression.
        The branches array must contain at least one branch document.
default:
    Optional.The path to take if no branch case expression evaluates to true.
    Although optional, if default is unspecified and no branch case evaluates to true,$switch returns an error.
实例：
db.grades.insertMany([
{ "_id" : 1, "name" : "Susan Wilkes", "scores" : [ 87, 86, 78 ] },
{ "_id" : 2, "name" : "Bob Hanna", "scores" : [ 71, 64, 81 ] },
{ "_id" : 3, "name" : "James Torrelio", "scores" : [ 91, 84, 97 ] }
])
db.grades.aggregate( [
  {
    $project:
      {
        "name" : 1,
        "summary" :
        {
          $switch:
            {
              branches: [
                {
                  case: { $gte : [ { $avg : "$scores" }, 90 ] },
                  then: "Doing great!"
                },
                {
                  case: { $and : [ { $gte : [ { $avg : "$scores" }, 80 ] },
                                   { $lt : [ { $avg : "$scores" }, 90 ] } ] },
                  then: "Doing pretty well."
                },
                {
                  case: { $lt : [ { $avg : "$scores" }, 80 ] },
                  then: "Needs improvement."
                }
              ],
              default: "No scores found."
            }
         }
      }
   }
] )

$toLower(aggregation)
含义：
将String转为小写。允许一个表达式参数。
Converts a string to lowercase. Accepts a single argument expression.
语法：
{ $toLower: <expression> }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "PRODUCT 1" },
{ "_id" : 2, "item" : "abc2", quarter: "13Q4", "description" : "Product 2" },
{ "_id" : 3, "item" : "xyz1", quarter: "14Q2", "description" : null }
])
db.inventory.aggregate(
   [
     {
       $project:
         {
           item: { $toLower: "$item" },
           description: { $toLower: "$description" }
         }
     }
   ]
)

$toUpper(aggregation)
含义：
字符串转换为大写
Converts a string to uppercase. Accepts a single argument expression.
语法：
{ $toUpper: <expression> }
实例：
db.inventory.insertMany([
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "PRODUCT 1" },
{ "_id" : 2, "item" : "abc2", quarter: "13Q4", "description" : "Product 2" },
{ "_id" : 3, "item" : "xyz1", quarter: "14Q2", "description" : null }
])
db.inventory.aggregate(
   [
     {
       $project:
         {
           item: { $toUpper: "$item" },
           description: { $toUpper: "$description" }
         }
     }
   ]
)

$trunc(aggregation)
含义：
将数字截取为整数
Truncates a number to its integer.
语法：
{ $trunc: <number> }
{ $trunc: 0 }    0
{ $trunc: 7.80 }    7
{ $trunc: -2.3 }    -2
实例：
db.samples.insertMany([
{ _id: 1, value: 9.25 },
{ _id: 2, value: 8.73 },
{ _id: 3, value: 4.32 },
{ _id: 4, value: -5.34 }
])
db.samples.aggregate([
   { $project: { value: 1, truncatedValue: { $trunc: "$value" } } }
])

$type(aggregation)
含义：
返回字段的BSON格式
Return the BSON data type of the field.
语法：
{ $type: <expression> }
{ $type: "a" }    "string"
{ $type: /a/ }    "regex"
{ $type: 1 }    "double"
{ $type: NumberLong(627) }    "long"
{ $type: { x: 1 } }    "object"
{ $type: [ [ 1, 2, 3 ] ] }    "array"
实例：
db.coll.insertMany([
{ _id: 0, a : 8 },
{ _id: 1, a : [ 41.63, 88.19 ] },
{ _id: 2, a : { a : "apple", b : "banana", c: "carrot" } },
{ _id: 3, a :  "caribou" },
{ _id: 4, a : NumberLong(71) },
{ _id: 5 }
])
db.coll.aggregate([{
    $project: {
       a : { $type: "$a" }
    }
}])

$zip(aggregation)
$dateFromParts(aggregation)
$dateToParts(aggregation)
$dateFromString(aggregation)
$dateToString(aggregation)
$substrBytes(aggregation)
$exp(aggregation)
$stdDevPop(aggregation)
$stdDevSamp(aggregation)
$indexOfCP(aggregation)
