# 聚合管道阶段（Aggregation Pipeline Stages）

$addFields(aggregate)
含义：
1.添加新的字段到文档，$addFiles输出包含新添加的字段或已存在的字段。
Adds new fields to documents.$addFields outputs documents that contain all existing fields from the input documents and newly added fields.
2.$addFields相当于$project，它明确指定输入文档中所有字段并添加新字段。
The $addFields stage is equivalent to a $project stage that explicitly specifies all existing fields in the input documents and adds the new fields.
3.如果新字段的名称与现有字段名称（包括_id）相同，$addFields将用指定表达式的值覆盖该字段的现有值。
If the name of the new field is the same as an existing field name(including _id),$addFields overwrites the existing value of that field with the value of the specified expression.
语法：
{ $addFields: { <newField>: <expression>, ... } }
实例：
1.使用两个$addFields
db.scores.insertMany([{
  _id: 1,
  student: "Maya",
  homework: [ 10, 5, 10 ],
  quiz: [ 10, 8 ],
  extraCredit: 0
},
{
  _id: 2,
  student: "Ryan",
  homework: [ 5, 6, 5 ],
  quiz: [ 8, 8 ],
  extraCredit: 8
}
])
db.scores.aggregate( [
   {
     $addFields: {
       totalHomework: { $sum: "$homework" } ,
       totalQuiz: { $sum: "$quiz" }
     }
   },
   {
     $addFields: { totalScore:
       { $add: [ "$totalHomework", "$totalQuiz", "$extraCredit" ] } }
   }
] )
2.为字段添加一个内嵌文档
db.vehicles.insertMany([
{ _id: 1, type: "car", specs: { doors: 4, wheels: 4 } },
{ _id: 2, type: "motorcycle", specs: { doors: 0, wheels: 2 } },
{ _id: 3, type: "jet ski" }
])
db.vehicles.aggregate( [
        {
           $addFields: {
              "specs.fuel_type": "unleaded"
           }
        }
   ] )
3.覆盖一个已存在的字段
db.animals.insert({ _id: 1, dogs: 10, cats: 15 })
db.animals.aggregate( [
  {
    $addFields: { "cats": 20 }
  }
] )
db.fruit.insertMany([
{ "_id" : 1, "item" : "tangerine", "type" : "citrus" },
{ "_id" : 2, "item" : "lemon", "type" : "citrus" },
{ "_id" : 3, "item" : "grapefruit", "type" : "citrus" }
])
db.fruit.aggregate( [
  {
    $addFields: {
      _id : "$item",
      item: "fruit"
    }
  }
] )


$bucket(aggregate)     
含义：
1.根据指定的表达式和存储区边界将传入文档分组。
Categorizes incoming documents into groups,called buckets, based on a specified expression and bucket boundaries.
2.每个bucket在输出中表示为一个文档。文档为每个桶都提供一个_id字段，这个字段的值是一个数字其指定bucket的下限。当输出未指定时，默认包含计数字段。
Each bucket is represented as a document in the output.The document for each bucket contains an _id field, whose value specifies the inclusive lower bound of the bucket and a count field that contains the number of documents in the bucket.The count field is included by default when the output is not specified.
3.$bucket只会生成包含至少一个输入文档的bucket的输出文档。
$bucket only produces output documents for buckets that contain at least one input document.
语法：
{
  $bucket: {
      groupBy: <expression>,
      boundaries: [ <lowerbound1>, <lowerbound2>, ... ],
      default: <literal>,
      output: {
         <output1>: { <$accumulator expression> },
         ...
         <outputN>: { <$accumulator expression> }
      }
   }
}
groupBy:
用来将文档分组的表达式。指定一个字段路径，使用美元符合$作为字段的前缀并且使用冒号包裹住。
An expression to group documents by.To specify a field path,prefix the field name with a dollar sign $ and enclose it in quotes.
除非$bucket包含默认规范，否则每个输入的表达式都必须包含一个groupBy字段或者一个在指定范围内的值（boundaries字段）。
Unless $bucket includes a default specification, each input document must resolve the groupBy field path or expression to a value that within one of the ranges specified by the boundaries.
boundaries:
一个基于groupBy表达符的数组为每个bucket划分界限。每个相邻的值作为一个bucket的上下限边界。boundaries的值最少为两个。
An array of values based on the groupBy expression that specify the boundaries for each bucket.Each adjacent part of values acts as the inclusive lower boundary and the exclusive upper boundary for the bucket.You must specify act least two boundaries.
default:
可选变量。其指定一个不在groupBy字段boundaries参数范围内的字面量。
Optional.A literal that specifies the _id of an additional bucket that contains all documents whose groupBy expression result does not fall into a bucket specified by boundaries.
如果未指定，则每个输入文档都必须将groupBy表达式解析为由边界指定的某个bucket的boundaries参数范围内的值，否则该操作将引发错误。
If unspecified, each input document must resolve the groupBy expression to a value within one of the bucket ranges specified by boundaries of the operation throws an error.
默认值必须小于最小边界值，或者大雨或等于最大边界值。
The default value must be less than the lowest boundaries value,or greater than or equal to the highest boundaries value.
默认值可以时与边界中的条目不同的类型。
The default value can be of a different type than the entries in boundaries.
output:
可选参数，指定与_id字段一起输出的其他文档字段。
Optional.A document that specifies the fields to include in the output documents in addition to the _id field.To specify the field to include, you must use accumulator expressions
实例：
1.单独使用$bucket
db.artwork.insertMany([
{ "_id" : 1, "title" : "The Pillars of Society", "artist" : "Grosz", "year" : 1926,
    "price" : NumberDecimal("199.99") },
{ "_id" : 2, "title" : "Melancholy III", "artist" : "Munch", "year" : 1902,
    "price" : NumberDecimal("280.00") },
{ "_id" : 3, "title" : "Dancer", "artist" : "Miro", "year" : 1925,
    "price" : NumberDecimal("76.04") },
{ "_id" : 4, "title" : "The Great Wave off Kanagawa", "artist" : "Hokusai",
    "price" : NumberDecimal("167.30") },
{ "_id" : 5, "title" : "The Persistence of Memory", "artist" : "Dali", "year" : 1931,
    "price" : NumberDecimal("483.00") },
{ "_id" : 6, "title" : "Composition VII", "artist" : "Kandinsky", "year" : 1913,
    "price" : NumberDecimal("385.00") },
{ "_id" : 7, "title" : "The Scream", "artist" : "Munch", "year" : 1893
    /* No price*/ },
{ "_id" : 8, "title" : "Blue Flower", "artist" : "O'Keefe", "year" : 1918,
    "price" : NumberDecimal("118.42") }
])
db.artwork.aggregate( [
  {
    $bucket: {
      groupBy: "$price",
      boundaries: [ 0, 200, 400 ],
      default: "Other",
      output: {
        "count": { $sum: 1 },
        "titles" : { $push: "$title" }
      }
    }
  }
] )
2.搭配$facet使用$bucket
db.artwork.aggregate( [
  {
    $facet: {
      "price": [
        {
          $bucket: {
              groupBy: "$price",
              boundaries: [ 0, 200, 400 ],
              default: "Other",
              output: {
                "count": { $sum: 1 },
                "artwork" : { $push: { "title": "$title", "price": "$price" } }
}}}],
      "year": [
        {
          $bucket: {
            groupBy: "$year",
            boundaries: [ 1890, 1910, 1920, 1940 ],
            default: "Unknown",
            output: {
              "count": { $sum: 1 },
              "artwork": { $push: { "title": "$title", "year": "$year" } }
}}}]}}] )

$count(aggregate)     
含义：
向控制台输出文档数量
Returns a document that contains a count of the number of documents input to the stage.
语法：
{ $count: <string> }
<string>是一个存在数量值的输出字段的名称。<string>必须是一个非空字符串，必须不能以$开头并且不能包含逗点操作符。
<string> is the name of the output field which has the count as its value.<string> must be a non-empty string, must not start with $ and must not contain the . character.
实例：
db.scores.insertMany([
{ "_id" : 1, "subject" : "History", "score" : 88 },
{ "_id" : 2, "subject" : "History", "score" : 92 },
{ "_id" : 3, "subject" : "History", "score" : 97 },
{ "_id" : 4, "subject" : "History", "score" : 71 },
{ "_id" : 5, "subject" : "History", "score" : 79 },
{ "_id" : 6, "subject" : "History", "score" : 83 }
])
db.scores.aggregate(
  [
    {
      $match: {
        score: {
          $gt: 80
        }
      }
    },
    {
      $count: "passing_scores"
    }
  ]
)
1.$match阶段排除分数小于或等于80的文档，将分数大于80的文档传递到下一阶段。
The $match stage excludes documents that have a score value of less than or equal to 80 to pass along the documents with score greater than 80 to the next stage.
2.$count阶段返回聚合管道中剩余文档的计数值,并且指定该字段为passing_scores。
The $count stage returns a count of the remaining documents in the aggregation pipeline and assigns the value to a field called passing_scores.


$facet(aggregate)    
含义：
1.在一个阶段处理同一组输入文档中的多个聚合管道。每个子管道在输出文档中都有自己的字段，其结果以文档数组形式存储。
Processes multiple aggregation pipelines within a single stage on the same set of input documents.Each sub-pipeline has its own field in the output document where its results are stored as an array of documents.
2.在$facet阶段允许你创建多个方面的聚合操作，在一个独立的聚和操作阶段，这些聚合操作的特征表现为数据穿过多个维度或方面。多面的聚合提供多个过滤器和归类来指导数据的浏览和分析。零售商通常通过在产品价格，制造商，尺寸等方面创建过滤器来缩小搜索结果的范围。
The $facet stage allows you to create multi-faceted aggregations which characterize data across multiple dimensions, or facets, within a single aggregation stage.Multi-faceted aggregations provide multiple filters and categorizations to guide data browsing and analysis.Retailers commonly use faceting to narrow search results by creating filters on product price, manufacturer,size,etc.
3.$facet阶段只接收来源文档一次。$facet可在同一组输入文档上启用各种汇总，从而无需多次检索输入文档。
Input documents are passed to the $facet stage only once.$facet enables various aggregations on the same set of input documents, without needing to retrieve the input documents multiple times.
4.$facet的每个子管道都传入完全相同的输入文档。这些子管道都是完全独立的，它们的输出文档数组都是完全独立在不同的字段中。一个子管道的输出不能用于同一个$facet阶段内不同子管道。如果需要进一步的聚合，请在$facet之后添加其他阶段，并指定所需子管道输出的字段名称<outputField>。
Each sub-pipeline within $facet is passed the exact same set of input document.These sub-pipelines are completely independent of one another and document array output by each is stored in separate fields in the output document.The output of one sub-pipeline can not be used as the input for a different sub-pipeline within the same $facet stage.If further aggregations are required, add additional stages after $facet and specify the field name,<outputField>,of the desired sub-pipeline output.
语法：
{ $facet:
   {
      <outputField1>: [ <stage1>, <stage2>, ... ],
      <outputField2>: [ <stage1>, <stage2>, ... ],
      ...

   }
}
实例：
db.inventory.insertMany([
{ "_id" : 1, "title" : "The Pillars of Society", "artist" : "Grosz", "year" : 1926,
  "price" : NumberDecimal("199.99"),
  "tags" : [ "painting", "satire", "Expressionism", "caricature" ] },
{ "_id" : 2, "title" : "Melancholy III", "artist" : "Munch", "year" : 1902,
  "price" : NumberDecimal("280.00"),
  "tags" : [ "woodcut", "Expressionism" ] },
{ "_id" : 3, "title" : "Dancer", "artist" : "Miro", "year" : 1925,
  "price" : NumberDecimal("76.04"),
  "tags" : [ "oil", "Surrealism", "painting" ] },
{ "_id" : 4, "title" : "The Great Wave off Kanagawa", "artist" : "Hokusai",
  "price" : NumberDecimal("167.30"),
  "tags" : [ "woodblock", "ukiyo-e" ] },
{ "_id" : 5, "title" : "The Persistence of Memory", "artist" : "Dali", "year" : 1931,
  "price" : NumberDecimal("483.00"),
  "tags" : [ "Surrealism", "painting", "oil" ] },
{ "_id" : 6, "title" : "Composition VII", "artist" : "Kandinsky", "year" : 1913,
  "price" : NumberDecimal("385.00"),
  "tags" : [ "oil", "painting", "abstract" ] },
{ "_id" : 7, "title" : "The Scream", "artist" : "Munch", "year" : 1893,
  "tags" : [ "Expressionism", "painting", "oil" ] },
{ "_id" : 8, "title" : "Blue Flower", "artist" : "O'Keefe", "year" : 1918,
  "price" : NumberDecimal("118.42"),
  "tags" : [ "abstract", "painting" ] }
])

db.artwork.aggregate( [
  {
    $facet: {
      "categorizedByTags": [
        { $unwind: "$tags" },
        { $sortByCount: "$tags" }
      ],
      "categorizedByPrice": [
        // Filter out documents without a price e.g., _id: 7
        { $match: { price: { $exists: 1 } } },
        {
          $bucket: {
            groupBy: "$price",
            boundaries: [  0, 150, 200, 300, 400 ],
            default: "Other",
            output: {
              "count": { $sum: 1 },
              "titles": { $push: "$title" }
            }
          }
        }
      ],
      "categorizedByYears(Auto)": [
        {
          $bucketAuto: {
            groupBy: "$year",
            buckets: 4
          }
        }
      ]
    }
  }
])

$group(aggregate)
含义：
根据指定的表达式分组文档并输出分组结果到下一个阶段。每组输出文档都以被分组的键生成一个_id字段。输出的结果可以是字段通过计算生成的新结果，该字段包含由$group的_id字段分组的一些表达式累加器的值。$group不能排序其输出文档。
Groups documents by some specified expression and outputs to the next stage a document for each distinct grouping.The output documents contain an _id field which contains the distinct group by key.The output documents can also contain computed fields that hold the values of some accumulator expression grouped by the $group’s _id field.$group does not order its output documents.
语法：
{ $group: { _id: <expression>, <field1>: { <accumulator1> : <expression1> }, ... } }
可以在$group语句中使用的叠加操作符
$sum,$avg,$first,$last,$max,$min,$push,$addToSet,$stdDevPop,$stdDevSamp
注意：
1.$group阶段的内存限制为100mb。默认情况下，如果当前阶段超过了这个限制，$group将会产生一个错误。但是，为了允许处理大型数据集，请将allowDiskUse选项设置为true以启用$group操作以写入临时文件。
The $group stage has a limit of 100 megabytes of RAM.By default, if the stage exceeds this limit,$group will produce an error.However,to allow for the handling of large datasets,set the allowDiskUse option to true to enable $group operations to write to temporary files.See db.collection.aggregate() method and the aggregate command for details.
实例：
db.sales.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-03-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-03-01T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-03-15T09:00:00Z") },
{ "_id" : 4, "item" : "xyz", "price" : 5, "quantity" : 20, "date" : ISODate("2014-04-04T11:21:39.736Z") },
{ "_id" : 5, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-04-04T21:23:13.331Z") }
])
1.根据年月日分组
db.sales.aggregate(
[{
        $group : {
           _id : { month: { $month: "$date" }, day: { $dayOfMonth: "$date" }, year: { $year: "$date" } },
           totalPrice: { $sum: { $multiply: [ "$price", "$quantity" ] } },
           averageQuantity: { $avg: "$quantity" },
           count: { $sum: 1 }
}}])
2.根据Null分组（打包全部数据于一组）
db.sales.aggregate(
[{$group : {
           _id : null,
           totalPrice: { $sum: { $multiply: [ "$price", "$quantity" ] } },
           averageQuantity: { $avg: "$quantity" },
           count: { $sum: 1 }
}}])
3.去重
db.sales.aggregate([{$group:{_id:"$item"}}])
4.Pivot Date
db.books.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-03-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-03-01T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-03-15T09:00:00Z") },
{ "_id" : 4, "item" : "xyz", "price" : 5, "quantity" : 20, "date" : ISODate("2014-04-04T11:21:39.736Z") },
{ "_id" : 5, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-04-04T21:23:13.331Z") }])
db.books.aggregate(
   [
     { $group : { _id : "$author", books: { $push: "$$ROOT" } } }
   ]
)
System Variables（系统变量）
ROOT:
引用顶层文档
References the root document,i.e the top-level document,currently being processed in the aggregation pipeline stage.
CURRENT:
从字段开始的位置引用全部内容传入聚合管道进行处理。除非另有说明，负责所有阶段都以CURRENT与ROOT相同的方式开始。
References the start of the field path being processed in the aggregation pipeline stage.Unless documented otherwise,all stages start with CURRENT the same as ROOT.
CURRENT is modifiable.However,since $<field> is equivalent to $$CURRENT .<field>,rebinding CURRENT changes the meaning of $ accesses.
REMOVE:
A variable which evaluates to the missing value.Allows for the conditional exclusion of fields.In a $projection,a field set to the variable REMOVE is excluded from output.
DESCEND:
One of the allowed results of a $redact expression.
PRUNE:
One of the allowed results of a $redact expression.
KEEP:
One of the allowed results of a $redact expression.

$indexStats(aggregate)
含义：
返回有关使用集合中每个索引的统计信息。如果使用访问控制运行，用户必须具有包含indexStats操作的权限。
Returns statistics regarding the use of each index for the collection.If running with access control,the user must have privileges that include indexStats action.
语法：
{$indexStats:{}}
实例：     
db.orders.insertMany([
{ "_id" : 1, "item" : "abc", "price" : 12, "quantity" : 2, "type": "apparel" },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "type": "electronics" },
{ "_id" : 3, "item" : "abc", "price" : 10, "quantity" : 5, "type": "apparel" }])
db.orders.find( { type: "apparel"} )
db.orders.find( { item: "abc" } ).sort( { quantity: 1 } )
db.orders.aggregate( [ { $indexStats: { } } ] )

$limit(aggregate)
含义：
限制管道传递到下一个阶段文档数量
Limits the number of documents passed to the next stage in the pipeline.
注意：
$sort优于$limit之前立即处理，$sort操作只会保留前n个结果，其中n是指定的界限，而MongoDB只需要在哪吃中存储n个元素。当allowDiskUse为true并且这n个元素超过聚合操作的内存限制时，此优化仍然适用。
When a $sort immediately precedes a $limit in the pipeline,the $sort operation only maintains the top n results as it progresses,where n is the specified limit,and MongoDB only needs to store n items in memory.This optimization still applies when allowDiskUse is true and the n items exceed the aggregation memory limit.
语法：
{ $limit: <positive integer> }
实例：     
db.article.aggregate({ $limit : 5 })

$lookup(aggregate) 
含义：
在同一数据库中从被连接的集合执行一个左连接到未分片的集合，对于每个输入文档，在$lookup阶段添加一个新的数组字段，其元素来自“joined”集合所匹配的文档。$lookup阶段将这些重新构建的文档传递到下一个阶段。
Performs a left outer join to an unsharded collection in the same database to filter in documents from the “joined” collection for processing.To each input document,the $lookup stage adds a new array field whose elements are the matching documents from the “joined” collection.The $lookup stage passes these reshaped documents to the next stage.
语法：
一、同等匹配（Equality Match）
{$lookup:{
       from: <collection to join>,
       localField: <field from the input documents>,
       foreignField: <field from the documents of the "from" collection>,
       as: <output array field>
}}
from:
在相同的数据库中指定集合来执行join连接。这个被from的集合不能被分片。
Specifies the collection in the same database to perform the join with.The from collection annot be sharded.
localField:
从文档中向$lookup阶段输入指定的字段。$lookup阶段在from指定的文档的localField字段和foreignField字段执行完全匹配。如果输入文档不包含localField字段，$lookup将会将该字段视为null以用于匹配目标字段。
Specifies the field from the documents input to the $lookup stage.$lookup performs an equality match on the localField to the foreignField from the documents of the from collection.If an input document does not contain the localField,The $lookup treats the field as having a value of null for matching purposes.
如果localField是一个数组，你可能需要添加一个$unwind阶段于你的聚合管道。否则，localField和foreignField之间的相对条件是foreignField: {$in :[localField.elem1,localField.elem2,…]}
If you localField is an array,you may want to add an $unwind stage to your pipeline.Otherwise,the equality condition between the localField and foreignField is foreignField: {$in :[localField.elem1,localField.elem2,…]}.
foreignField:
指定from所指向的集合文档的字段。$lookup在输入文档的foreignField与localField之间执行完全匹配。如果from所指向的集合文档不包含foreignField字段，$lookup将会将null视为匹配的目标。
Specifies the field from documents in the from collection.$lookup performs an equality match on the foreignField to the localField from the input documents.If a document in the from collection does not contain the foreignField,the $lookup treats the value as null for matching purposes.
as:
为输入文档指定新命名的数组字段。这个新数组字段包含来自from指向的集合中的匹配文档。如果指定的名称在输入文档中已存在，则现有字段将会被覆写。
Specifies the name of the new array field to add to the input documents.The new array field contains the matching documents from the from collection.If the specified name already exists in the input document,the existing field is overwritten.
所对应的pseudo-SQL：
SELECT *, <output array field>
FROM collection
WHERE <output array field> IN (SELECT *
                               FROM <collection to join>
                               WHERE <foreignField>= <collection.localField>);
二、加入条件和不相关的子查询（Join Conditions and Uncorrelated Sub-queries）
{$lookup:{
       from: <collection to join>,
       let: { <var_1>: <expression>, …, <var_n>: <expression> },
       pipeline: [ <pipeline to execute on the collection to join> ],
       as: <output array field>
}}
from:
在数据库中指定一个集合来执行join连接。form不能指向分片集合。
Specifies the collection in the same database to perform the join with.The form collection cannot be sharded.
let:
可选参数，指定在管道字段阶段使用的变量。使用变量表达式来访问输入到$lookup阶段的文档字段。
Optional.Specifies variables to use in the pipeline field stages.Use the variable expressions to access the fields from the documents input to the $lookup stage.
管道不能直接访问输入文档字段，相反，首先定义输入文档字段的变量，然后引用管道阶段的变量。
The pipline cannot directly access the input document fields.Instead,first define the variables for the input document fields,and then reference the variables in the stages in the pipeline.
要访问管道中的let变量，应该使用$expr操作符。
To access the let variables in the pipeline,use the $expr operator.
在管道阶段let变量是可以访问的，包括嵌套在管道中的$lookup阶段。
The let variables are accessible by the stages in the pipeline,including additional $lookup stages nested in the pipeline.
pipeline:
指定要在连接集合上运行的管道。管道从连接的集合中确定生成的文档。要返回所有的文档，请指定一个空管道[]。
Specifies the pipeline to run on the joined collection.The pipeline determines the resulting documents from the joined collection.To return all documents,specify an empty pipeline [].
管道不能直接访问输入文档字段。反而，首先定义输入文档字段的变量，然后引用管道中各个阶段的变量。
The pipeline cannot directly access the input document fields.Instead,first define the variables for the input document fields,and then reference the variables in the stages in the pipeline.
要访问管道中的let变量，应使用$expr操作符。
To access the let variables in the pipeline,use the $expr operator.
let变量可以在管道阶段访问，包括嵌套在管道中额外的$lookup阶段。
The let variables are accessible by the stages in the pipeline,including additional $lookup stages nested in the pipeline.
as:
指定要添加到输入文档的新数组字段。新数组字段包含来自from指向的集合中的所匹配的文档。如果指定的新字段名已经存在于文档中，则该字段名将会被覆盖。
Specifies the name of the new array field to add  to input documents.The new array field contains the matching documents from the from collection.If the specified name alaready exists in the document,the existing field is overwritten.
SELECT *, <output array field>
FROM collection
WHERE <output array field> IN (SELECT <documents as determined from the pipeline>
                               FROM <collection to join>
                               WHERE <pipeline> );
实例：
1.执行一个等值关联的$lookup
db.orders.insertMany([
   { "_id" : 1, "item" : "almonds", "price" : 12, "quantity" : 2 },
   { "_id" : 2, "item" : "pecans", "price" : 20, "quantity" : 1 },
   { "_id" : 3  }
])
db.inventory.insertMany([
   { "_id" : 1, "sku" : "almonds", description: "product 1", "instock" : 120 },
   { "_id" : 2, "sku" : "bread", description: "product 2", "instock" : 80 },
   { "_id" : 3, "sku" : "cashews", description: "product 3", "instock" : 60 },
   { "_id" : 4, "sku" : "pecans", description: "product 4", "instock" : 70 },
   { "_id" : 5, "sku": null, description: "Incomplete" },
   { "_id" : 6 }
])
db.orders.aggregate([
   {
     $lookup:
       {
         from: "inventory",
         localField: "item",
         foreignField: "sku",
         as: "inventory_docs"
       }
  }
])
SELECT *, inventory_docs
FROM orders
WHERE inventory_docs IN (SELECT *
FROM inventory
WHERE sku= orders.item);
2.在数组中使用$lookup
如果你的localField是一个数组，并且你希望将它内部的元素与一个foreignField（它是一个单独的元素）进行匹配，那么你需要使用$unwind作为聚合的第一个阶段。
If your localField is an array and you’d like to match the elements inside it against a foreignField which is a single element, you’ll need to $unwind the array as one stage of the aggregation pipeline.
db.orders.insert({ "_id" : 1, "item" : "MON1003", "price" : 350, "quantity" : 2, "specs" :
[ "27 inch", "Retina display", "1920x1080" ], "type" : "Monitor" }
)
db.inventory.insertMany([
{ "_id" : 1, "sku" : "MON1003", "type" : "Monitor", "instock" : 120,
"size" : "27 inch", "resolution" : "1920x1080" },
{ "_id" : 2, "sku" : "MON1012", "type" : "Monitor", "instock" : 85,
"size" : "23 inch", "resolution" : "1280x800" },
{ "_id" : 3, "sku" : "MON1031", "type" : "Monitor", "instock" : 60,
"size" : "23 inch", "display_type" : "LED" }
])
db.orders.aggregate([
   {
      $unwind: "$specs"
   },
   {
      $lookup:
         {
            from: "inventory",
            localField: "specs",
            foreignField: "size",
            as: "inventory_docs"
        }
   },
   {
      $match: { "inventory_docs": { $ne: [] } }
   }
])
3.搭配$lookup使用$mergeObjects
db.orders.insertMany([
   { "_id" : 1, "item" : "almonds", "price" : 12, "quantity" : 2 },
   { "_id" : 2, "item" : "pecans", "price" : 20, "quantity" : 1 }
])
db.items.insertMany([
  { "_id" : 1, "item" : "almonds", description: "almond clusters", "instock" : 120 },
  { "_id" : 2, "item" : "bread", description: "raisin and nut bread", "instock" : 80 },
  { "_id" : 3, "item" : "pecans", description: "candied pecans", "instock" : 60 }
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
4.为$lookup指定多个联合条件
db.orders.insertMany([
  { "_id" : 1, "item" : "almonds", "price" : 12, "ordered" : 2 },
  { "_id" : 2, "item" : "pecans", "price" : 20, "ordered" : 1 },
  { "_id" : 3, "item" : "cookies", "price" : 10, "ordered" : 60 }
])
db.warehouses.insertMany([
  { "_id" : 1, "stock_item" : "almonds", warehouse: "A", "instock" : 120 },
  { "_id" : 2, "stock_item" : "pecans", warehouse: "A", "instock" : 80 },
  { "_id" : 3, "stock_item" : "almonds", warehouse: "B", "instock" : 60 },
  { "_id" : 4, "stock_item" : "cookies", warehouse: "B", "instock" : 40 },
  { "_id" : 5, "stock_item" : "cookies", warehouse: "A", "instock" : 80 }
])
db.orders.aggregate([
   {
      $lookup:
         {
           from: "warehouses",
           let: { order_item: "$item", order_qty: "$ordered" },
           pipeline: [
              { $match:
                 { $expr:
                    { $and:
                       [
                         { $eq: [ "$stock_item",  "$$order_item" ] },
                         { $gte: [ "$instock", "$$order_qty" ] }
                       ]
                    }
                 }
              },
              { $project: { stock_item: 0, _id: 0 } }
           ],
           as: "stockdata"
         }
    }
])
SELECT *, stockdata
FROM orders
WHERE stockdata IN (SELECT warehouse, instock
                    FROM warehouses
                    WHERE stock_item= orders.item
                    AND instock >= orders.ordered );
5.不相关的子查询
db.absences.insertMany([
   { "_id" : 1, "student" : "Ann Aardvark", sickdays: [ new Date ("2018-05-01"),new Date ("2018-08-23") ] },
   { "_id" : 2, "student" : "Zoe Zebra", sickdays: [ new Date ("2018-02-01"),new Date ("2018-05-23") ] },
])
db.holidays.insertMany([
   { "_id" : 1, year: 2018, name: "New Years", date: new Date("2018-01-01") },
   { "_id" : 2, year: 2018, name: "Pi Day", date: new Date("2018-03-14") },
   { "_id" : 3, year: 2018, name: "Ice Cream Day", date: new Date("2018-07-15") },
   { "_id" : 4, year: 2017, name: "New Years", date: new Date("2017-01-01") },
   { "_id" : 5, year: 2017, name: "Ice Cream Day", date: new Date("2017-07-16") }
])
db.absences.aggregate([
   {
      $lookup:
         {
           from: "holidays",
           pipeline: [
              { $match: { year: 2018 } },
              { $project: { _id: 0, date: { name: "$name", date: "$date" } } },
              { $replaceRoot: { newRoot: "$date" } }
           ],
           as: "holidays"
         }
    }
])
SELECT *, holidays
FROM absences
WHERE holidays IN (SELECT name, date
                    FROM holidays
                    WHERE year = 2018);

$match(aggregate)
含义：
1.传递根据指定条件过滤后的文档到下一个管道阶段。
Filters the documents to pass only the documents that match the specified condition(s) to the next pipeline stage.
2.$match根据指定的查询条件获取一个文档。其查询语法与operation query syntax相同。
$match takes a document that specifies the query conditions.The query syntax is identical to the read operation query syntax.
3.尽早将$match放置于聚合管道。因为$match可以限制聚合管道中文档的数量，尽早使用$match操作符可以最大量的限制聚合处理的数量。
Place the $match as early in the aggregation pipeline as possible.Because $match limits the total number of documents in the aggregation pipeline,earlier $match operations minimize the amount of processing down the pipe.
语法：
{ $match: { <query> } }
实例：     
db.articles.insertMany([
{ "_id" : ObjectId("512bc95fe835e68f199c8686"), "author" : "dave", "score" : 80, "views" : 100 },
{"author" : "dave", "score" : 85, "views" : 521 },
{"author" : "ahn", "score" : 60, "views" : 1000 },
{"author" : "li", "score" : 55, "views" : 5000 },
{"author" : "annT", "score" : 60, "views" : 50 },
{"author" : "li", "score" : 94, "views" : 999 },
{"author" : "ty", "score" : 95, "views" : 1000 }
])
db.articles.aggregate(
    [ { $match : { author : "dave" } } ]
);
db.articles.aggregate( [
  { $match: { $or: [ { score: { $gt: 70, $lt: 90 } }, { views: { $gte: 1000 } } ] } },
  { $group: { _id: null, count: { $sum: 1 } } }
] );

$out(aggregate)
含义：
获取由聚合管道返回的文档，并将其写入指定的集合。$out操作符必须是管道中最后一个阶段。$out操作符让聚合框架返回任意大小的结果集。
Takes the documents returned by the aggregation pipeline and writes them to a specified collection.The $out operator must be the last stage in the pipeline.The $out operator lets the aggregation framework return result sets of any size.
语法：
{ $out: "<output-collection>" }
实例：
db.books.insertMany([
{ "_id" : 8751, "title" : "The Banquet", "author" : "Dante", "copies" : 2 },
{ "_id" : 8752, "title" : "Divine Comedy", "author" : "Dante", "copies" : 1 },
{ "_id" : 8645, "title" : "Eclogues", "author" : "Dante", "copies" : 2 },
{ "_id" : 7000, "title" : "The Odyssey", "author" : "Homer", "copies" : 10 },
{ "_id" : 7020, "title" : "Iliad", "author" : "Homer", "copies" : 10 }
])
db.books.aggregate( [
                      { $group : { _id : "$author", books: { $push: "$title" } } },
                      { $out : "authors" }
                  ] )

$project(aggregate)
含义：
将包含请求字段的文档传递到管道的下一个阶段。指定的字段可以是来自输入文档或通过计算出现的新字段。
Passes along the documents with the requested fields to the next stage in the pipeline.The specified fields can be existing fields from the input documents or newly computed fields.
语法：
{ $project: { <specification(s)> } }
实例：    
db.books.aggregate( [ { $project : { title : 1 , author : 1 } } ] )
db.collection.aggregate( [ { $project: { myArray: [ "$x", "$y", "$someField" ] } } ] )

$replaceRoot(aggregate)
含义：
1.提升指定的文档到顶层并且替换所有其他的字段。这个操作将替换掉输入文档中所有存在的字段，包括_id字段。你可以将存在的嵌入文档提升到最顶层，或者创建一个新的提升文档。
Promotes a specified document to the top level and replaces all other fields.The operation replaces all existing fields in the input document,including the _id fiedl.You can promote an existing embedded document to the top level,or create a new document for promotion.
2.如果<replacementDocument>不是一个文档则会导致$replaceRoot操作失败。
$replaceRoot operations fail with an error if <replacementDocument> is not a document.
3.如果被替换文档在输入文档中的对应字段不存在，则会导致操作失败。为了确保被替换的文档存在，应在$replaceRoot阶段之前使用$match来验证文档字段是否有效。
If the replacement document refers to a field in the input document that does not exist, the operation fails with an error.To ensure that the replacement document exists, use a $match stage first to check for existence before passing documents to the $replaceRoot stage.
语法：
{ $replaceRoot: { newRoot: <replacementDocument> } }
实例：     
1.在内嵌文档中使用$replaceRoot
db.produce.insertMany([{
   "_id" : 1,
   "fruit" : [ "apples", "oranges" ],
   "in_stock" : { "oranges" : 20, "apples" : 60 },
   "on_order" : { "oranges" : 35, "apples" : 75 }
},
{
   "_id" : 2,
   "vegetables" : [ "beets", "yams" ],
   "in_stock" : { "beets" : 130, "yams" : 200 },
   "on_order" : { "beets" : 90, "yams" : 145 }
}
])
db.produce.aggregate( [{$replaceRoot: { newRoot: "$in_stock" }}] )
result：
{ "oranges" : 20, "apples" : 60 }
{ "beets" : 130, "yams" : 200 }

2.搭配$match使用$replaceRoot
db.people.isnertMany([
{ "_id" : 1, "name" : "Arlene", "age" : 34, "pets" : { "dogs" : 2, "cats" : 1 } },
{ "_id" : 2, "name" : "Sam", "age" : 41, "pets" : { "cats" : 1, "hamsters" : 3 } },
{ "_id" : 3, "name" : "Maria", "age" : 25 }
])
db.people.aggregate( [
   {$match: { pets : { $exists: true } }},
   {$replaceRoot: { newRoot: "$pets" }}
] )
{ "dogs" : 2, "cats" : 1 }
{ "cats" : 1, "hamsters" : 3 }

3.$replaceRoot一个新创建的文档
你也可以创建一个新文档作为$replaceRoot阶段的一部分，并且使用它们替换原文档
You can also create new documents as part of the $replaceRoot stage and use them to replace all the other fields.
db.contacts.insertMany([
{ "_id" : 1, "first_name" : "Gary", "last_name" : "Sheffield", "city" : "New York" },
{ "_id" : 2, "first_name" : "Nancy", "last_name" : "Walker", "city" : "Anaheim" },
{ "_id" : 3, "first_name" : "Peter", "last_name" : "Sumner", "city" : "Toledo" }
])
db.contacts.aggregate( [
{$replaceRoot: {
         newRoot: {
            full_name: {
               $concat : [ "$first_name", " ", "$last_name" ]
}}}}] )
{ "full_name" : "Gary Sheffield" }
{ "full_name" : "Nancy Walker" }
{ "full_name" : "Peter Sumner” }

4.$replaceRoot一个数组元素
db.contacts.insertMany([
{ "_id" : 1, "name" : "Susan",
  "phones" : [ { "cell" : "555-653-6527" },
               { "home" : "555-965-2454" } ] },
{ "_id" : 2, "name" : "Mark",
  "phones" : [ { "cell" : "555-445-8767" },
               { "home" : "555-322-2774" } ] }
])
db.contacts.aggregate( [
   {
      $unwind: "$phones"
   },
   {
      $match: { "phones.cell" : { $exists: true } }
   },
   {
      $replaceRoot: { newRoot: "$phones"}
   }
] )
{ "cell" : "555-653-6527" }
{ "cell" : "555-445-8767" }

$sample(aggregate)
含义：
1.从输入文档中随机选择指定数量的文档。
Randomly selects the specified number of documents from its input.
语法：
{ $sample: { size: <positive integer> } }
实例：    
db.users.drop()
db.users.insertMany([
{ "_id" : 1, "name" : "dave123", "q1" : true, "q2" : true },
{ "_id" : 2, "name" : "dave2", "q1" : false, "q2" : false  },
{ "_id" : 3, "name" : "ahn", "q1" : true, "q2" : true  },
{ "_id" : 4, "name" : "li", "q1" : true, "q2" : false  }
{ "_id" : 5, "name" : "annT", "q1" : false, "q2" : true  },
{ "_id" : 6, "name" : "li", "q1" : true, "q2" : true  },
{ "_id" : 7, "name" : "ty", "q1" : false, "q2" : true  }
])
db.users.aggregate(
   [ { $sample: { size: 3 } } ]
)

$skip(aggregate)
含义：
在$skip阶段执行跳过指定数量的文档并将剩余的文档传递到下一个管道阶段。
Skips over the specified number of documents that pass into the stage and passes the remaining documents to the next stage in the pipeline.
语法：
{ $skip: <positive integer> }
实例：
db.article.aggregate({ $skip : 5 });

$sort(aggregate)
含义：
对所有输入文档进行排序，并按排序顺序将其返回到管道。
Sorts all input documents and returns them to the pipeline in sorted order.
1升序，-1降序
1 to specify ascending order.
-1 to specify descending order.
注意：
不同类型排序优先级：
1.MinKey (internal type)
2.Null
3.Numbers (ints, longs, doubles, decimals)
4.Symbol, String
5.Object
6.Array
7.BinData
8.ObjectId
9.Boolean
10.Date
11.Timestamp
12.Regular Expression
13.MaxKey (internal type)
语法：
{ $sort: { <field1>: <sort order>, <field2>: <sort order> ... } }
实例：     
db.users.aggregate([{ $sort : { age : -1, posts: 1 } }])
db.users.aggregate([{ $match: { $text: { $search: "operating" } } },{ $sort: { score: { $meta: "textScore" }, posts: -1 } }])

$sortByCount(aggregate) 
含义：
1.根据指定的表达式对传入的文档进行分组，然后计算每个不同的组中的文档数量。
Groups incoming documents based on the value of a specified expression,then computes the count of documents in each distinct group.
2.每个输出文档都包含两个字段：_id与count
Each output document contains two fields: an _id field containing the distinct grouping value,and a count field containing the number of documents belonging to that grouping or category.
3.$sortByCount等价于$group+$sort顺序执行
The $sortByCount stage is equivalent to the following $group+$sort sequence
{ $group: { _id: <expression>, count: { $sum: 1 } } },
{ $sort: { count: -1 } }
语法：
{ $sortByCount:  <expression> }
实例：   
db.exhibits.insertMany([
{ "_id" : 1, "title" : "The Pillars of Society", "artist" : "Grosz", "year" : 1926, "tags" : [ "painting", "satire", "Expressionism", "caricature" ] },
{ "_id" : 2, "title" : "Melancholy III", "artist" : "Munch", "year" : 1902, "tags" : [ "woodcut", "Expressionism" ] },
{ "_id" : 3, "title" : "Dancer", "artist" : "Miro", "year" : 1925, "tags" : [ "oil", "Surrealism", "painting" ] },
{ "_id" : 4, "title" : "The Great Wave off Kanagawa", "artist" : "Hokusai", "tags" : [ "woodblock", "ukiyo-e" ] },
{ "_id" : 5, "title" : "The Persistence of Memory", "artist" : "Dali", "year" : 1931, "tags" : [ "Surrealism", "painting", "oil" ] },
{ "_id" : 6, "title" : "Composition VII", "artist" : "Kandinsky", "year" : 1913, "tags" : [ "oil", "painting", "abstract" ] },
{ "_id" : 7, "title" : "The Scream", "artist" : "Munch", "year" : 1893, "tags" : [ "Expressionism", "painting", "oil" ] },
{ "_id" : 8, "title" : "Blue Flower", "artist" : "O'Keefe", "year" : 1918, "tags" : [ "abstract", "painting" ] }
])
db.exhibits.aggregate( [ { $unwind: "$tags" },  { $sortByCount: "$tags" } ] )

$unwind(aggregate)
含义：
从输入文档中解构（拆分）一个数组字段，为每个数组元素生成一个单独的文档。每个输出文档用一个元素值替换数组。对于每个输入文档，输出n个文档，其中n是数组元素的数量，对应空数组可以是0.
Deconstructs an array field from the input documents to output a document for each element. Each output document replaces the array with an element value. For each input document, outputs n documents where n is the number of array elements and can be zero for an empty array.
语法：
syntax1:
{ $unwind: <field path> }
syntax2:
{$unwind:{
      path: <field path>,字段引用
      includeArrayIndex: <string>,数组下标
      preserveNullAndEmptyArrays: <boolean>非实数组处理
}}
path:
数组字段的引用。要指定字段引用，用美元符号$连接上字段名称并用引号包括起来。
Field path to an array field.To specify a field path,prefix the field name with a dollar sign $ and enclose in quotes.
includeArrayIndex:
可选项，为拆分数组元素输出原数组下标，这个新名称不能以美元符号$开头。
Optional.The name of a new field to hold the array index of the element.The name cannot start with a dollar sign $.
preserveNullAndEmptyArrays:
可选项，设置为true，如果字段引用为空或者不存在或者不包含子元素，则$unwind输出文档。设置为false，则上述情况$unwind不输出子文档。
Optional.If true,if the path is null,missing,or an empty array,$unwind outputs the document.If false,$unwind does not output a document if the path is null,missing,or an empty array.
默认值为false。
The default value is false.
注意：
1.Non-Array Field Path
在语法1中，$unwind阶段non-array操作不会再报错。如果操作没有拆分数组（字段不存在，null，空数组），$unwind会将该数组字段视为单个元素数组。
$unwind stage no longer errors on non-array operands.If the operand does not resolve to an array but is not missing,null,or an empty array,$unwind treats the operand as a single element array.
2.Missing Field
如果在输入文档中该字段不存在或者该字段不是一个数组，$unwind默认会忽略输入文档，并且不会有输出文档。
If you specify a path for a field that does not exist in an input document or the field is an empty array,$unwind,by default,ignores the input document and will not output documents for that input document.
实例：
1.展开数组
db.inventory.isnert({ "_id" : 1, "item" : "ABC1", sizes: [ "S", "M", "L"] })
db.inventory.aggregate( [ { $unwind : "$sizes" } ] )
2.语法2中使用includeArrayIndex和preserveNullAndEmptyArrays
db.inventory.isnertMany([{ "_id" : 1, "item" : "ABC", "sizes": [ "S", "M", "L"] },
{ "_id" : 2, "item" : "EFG", "sizes" : [ ] },
{ "_id" : 3, "item" : "IJK", "sizes": "M" },
{ "_id" : 4, "item" : "LMN" },
{ "_id" : 5, "item" : "XYZ", "sizes" : null }
])
下面的$unwin是等价的，并且为sizes数组字段中的每个元素都返回一个文档。如果sizes字段不能解析为一个数组，但是其存在、非null、不是一个空数组，$unwind将这个non-array视为一个单元素数组。
The following $unwind operations are equivalent and return a document for each element in the sizes field.If the sizes field does not resolve to an array but is not missing,null,or an empty array,$unwind treats the non-array operand as a single element array.
db.inventory.aggregate( [ { $unwind: "$sizes" } ] )
db.inventory.aggregate( [ { $unwind: { path: "$sizes" } } ] )
输出数组下标：
db.inventory.aggregate( [ { $unwind: { path: "$sizes", includeArrayIndex: "arrayIndex" } } ] )
当sizes不存在、null、空数组时使用preserveNullAndEmptyArrays选项操作：
db.inventory.aggregate( [
   { $unwind: { path: "$sizes", preserveNullAndEmptyArrays: true } }
] )


$currentOp(aggregate)  
$bucketAuto(aggregate)     
$collStats(aggregate)   
$geoNear(aggregate)
$graphLookup(aggregate)     
$listLocalSessions 
$listSessions
$redact(aggregate)
