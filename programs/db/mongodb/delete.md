

db.<collection>.remove(<query>,<options>)
<query>文档定义了删除操作时筛选文档条件
<options>文档声明了一些删除操作的参数

在默认情况下，remove命令会删除所有符合筛选条件的文档

如果只想删除满足筛选条件的第一篇文档，可以使用justOne选项

db.<collection>.drop({writeConcern:<document>})
这里的writeConcern文档定义了本次集合删除操作的安全写级别

remove可以删除集合内的所有文档，但是不会删除集合
drop命令可以删除整个集合，包括集合中的所有文档，以及集合的索引