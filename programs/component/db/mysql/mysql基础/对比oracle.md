对我来说，mysql最令我在意的短板（其实也就是Oracle的相对长处）在于：
hash join 。Oracle引进这的时候应该是90年代，而今天，Mysql还没有。这样连接性能肯定不行。当然，这个用变通解决可以绕过去。
with as  。Oracle中可以select查询作为一个中间结果，然后多处引用。这个mysql好像最近的8.X才有。多土啊，这可是最有用的功能之一。
rownumber。到目前为止，mysql还没有row_number。这可是程序员最常用到的功能之一。当然，可以变通解决。
Ananlytic function 和window function。 这个Oracle很强大（比如当前行引用前一行的值）。Mysql基本可以说是没有（当然也可以变通解决）
Stored procedure 这个mysql差太远太远，基本上是步枪对机枪。
Mysql 没有role吧？程序员得造轮子。（查了下，8.0有了。）
ORACLE数据库自带oracle apex 开发。好像数据库里只有oracle愿意投钱搞这个。
vpd,materialized view ,workflow,geological…… 这些mysql或者没有或者菜，当然oracle也不是每个版本都有。不过这些功能并不是日常必然用到的，所以还好啦。
总之，mysql短处不少，不过，用熟了之后，大多数是可以找到变通解决方法的。比如大家诟病很多的Hashjoin连接，用冗余字段解决，效果肯定比hashjoin好。