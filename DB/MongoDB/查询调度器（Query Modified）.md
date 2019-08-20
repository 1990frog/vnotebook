# 查询调度器（Query Modified）

$comment
含义：
$comment元命令可以在$query可能出现的任何上下文中对查询附加注释。
The $comment meta-operator makes it possible to attach a comment to a query in any context that $query may appear.
由于评论传递到配置文件日志，添加一个注释可以让你的配置文件数据更容易解释和追踪。
Beacause comments propagate to the profile log,adding a comment can make your profile data easier to interpret and trace.
示例 ：
db.collection.find( { <query> } ).comment( <comment> )
db.collection.find( { $query: { <query> }, $comment: <comment> } )

$explain
含义：
$explain操作符提供有关执行计划的信息。它返回一个描述用于返回查询的进程与索引的文档。这可能会在尝试优化查询时提供有用的信息。
The $explain operator provides information on the query plan.It returns a document that describes the process and indexes used to return the query.This may provide useful insight when attempting to optimize a query.
示例 ：
db.collection.find( { $query: {}, $explain: 1 } )
db.collection.find().explain()
db.collection.find()._addSpecial( "$explain", 1 )

$hint
含义：
$hint操作符强制查询优化器使用特定的索引来完成查询。通过索引名称或文档指定索引。
The $hint operator forces the query optimizer to use a specific index to fulfill the query.Specify the index either by the index name or by document.
使用$hint来测试查询性能和索引策略。mongo shell为$hint操作符提供了一个辅助方法hint().
Use $hint to testing query performance and indexing strategies.The mongo shell provides a helper method hint() for the $hint operator.
示例：
db.users.find().hint( { age: 1 } )
db.users.find()._addSpecial( "$hint", { age : 1 } )
db.users.find( { $query: {}, $hint: { age : 1 } } )

$maxScan
含义：
限制查询只在执行查询时扫描指定数量的文档。
Constrains the query to only scan the specified number of documents when fulfilling the query.
示例：
db.collection.find( { <query> } )._addSpecial( "$maxScan" , <number> )
db.collection.find( { $query: { <query> }, $maxScan: <number> } )


$max($min)
含义：
指定一个$max值来指定特定索引的唯一上限，以约束find()的结果。$max按顺序指定特定索引的所有键上限。
Specify a $max value to specify the exclusive upper bound for a specific index in order to constrain the results of find().The $max specifies the upper bound for all keys of a specific index in order.
If you use $max with $min to specify a range, the index bounds specified in $min and $max must both refer to the keys of the same index.
The min and max operators indicate that the system should avoid normal query planning. Instead they construct an index scan where the index bounds are explicitly specified by the values given in min and max.
语法：
db.collection.find( { <query> } ).max( { field1: <max value>, ... fieldN: <max valueN> } )
db.collection.find( { $query: { <query> }, $max: { field1: <max value1>, ... fieldN: <max valueN> } } )
db.collection.find( { <query> } )._addSpecial( "$max", { field1: <max value1>, ... fieldN: <max valueN> } )
示例：
1.指定专属上限（Specify Exclusive Upper Boud）
db.collection.find({<query>}).max({age:100})
2.索引选择（Index Selection）
You can explicitly specify the corresponding index with hint(). Otherwise, MongoDB selects the index using the fields in the $max and $min bounds; however, if multiple indexes exist on same fields with different sort orders, the selection of the index may be ambiguous.
collection集合有以下两个索引：
{age:1,type:-1}
{age:1,type:1}
如果不显示使用hint(),MongoDB可以为以下操作选择任一索引：
db.collection.find().max( { age: 50, type: 'B' } ) 3.$max与$min搭配使用:
Use $max alone or in conjunction with $min to limit results to a specific range for the same index, as in the following example:
db.collection.find().min( { age: 20 } ).max( { age: 25 } 

$maxTimeMS
含义：
Specifies a cumulative time limit in milliseconds for processing operations on a cursor. See maxTimeMS().

示例：
db.collection.find().maxTimeMS(100)
db.collection.find( { $query: { }, $maxTimeMS: 100 } )
db.collection.find( { } )._addSpecial("$maxTimeMS", 100)

$orderby
含义：
等同$sort
示例：
db.collection.find().sort( { age: -1 } )
db.collection.find()._addSpecial( "$orderby", { age : -1 } )
db.collection.find( { $query: {}, $orderby: { age : -1 } } )

$query
含义：
Wraps a query document.
示例：
db.collection.find( { $query: { age : 25 } } )
db.collection.find( { age : 25 }

$returnKey
含义：
只返回查询结果的索引的单个字段或多个字段。如果$returnKey设置为true，并且查询不适用索引来执行读取操作，则返回的文档将不包含任何字段。
Only return the index field or fields for the results of the query.If $returnKey is set to true and the query does not use an index to perform the read operation,the returned documents wil not contain any fields.
示例：
db.collection.find( { <query> } )._addSpecial( "$returnKey", true )
db.collection.find( { $query: { <query> }, $returnKey: true } )




$showDiskLoc
$natural
