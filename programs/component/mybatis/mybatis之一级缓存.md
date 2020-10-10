[TOC]

![20181018234543196](https://gitee.com/caijingquan/imagebed/raw/master/1602318950_20200306142846968_1581187570.png)



# 一级缓存
## 介绍
在应用运行过程中，我们有可能在一次数据库会话中，执行多次查询条件完全相同的SQL，MyBatis提供了一级缓存的方案优化这部分场景，如果是相同的SQL语句，会优先命中一级缓存，避免直接对数据库进行查询，提高性能。具体执行过程如下图所示。
![6e38df6a](https://gitee.com/caijingquan/imagebed/raw/master/1602318951_20200306150306163_2001507809.jpg)
每个SqlSession中持有了Executor，每个Executor中有一个LocalCache。当用户发起查询时，MyBatis根据当前执行的语句生成MappedStatement，在Local Cache进行查询，如果缓存命中的话，直接返回结果给用户，如果缓存没有命中的话，查询数据库，结果写入Local Cache，最后返回结果给用户。具体实现类的类关系图如下图所示。

![d76ec5fe](https://gitee.com/caijingquan/imagebed/raw/master/1602318951_20200306150936174_1983044805.jpg)
# 配置
我们来看看如何使用MyBatis一级缓存。开发者只需在MyBatis的配置文件中，添加如下语句，就可以使用一级缓存。共有两个选项，SESSION或者STATEMENT，默认是SESSION级别，即在一个MyBatis会话中执行的所有语句，都会共享这一个缓存。一种是STATEMENT级别，可以理解为缓存只对当前执行的这一个Statement有效。

```java
<setting name="localCacheScope" value="SESSION"/>
```

# 一级缓存实验
接下来通过实验，了解MyBatis一级缓存的效果，每个单元测试后都请恢复被修改的数据。

首先是创建示例表student，创建对应的POJO类和增改的方法，具体可以在entity包和mapper包中查看。
```sql
CREATE TABLE `student` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(200) COLLATE utf8_bin DEFAULT NULL,
  `age` tinyint(3) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
```
在以下实验中，id为1的学生名称是凯伦。
## 实验1
开启一级缓存，范围为会话级别，调用三次getStudentById，代码如下所示：
```java
public void getStudentById() throws Exception {
    SqlSession sqlSession = factory.openSession(true); // 自动提交事务
    StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
    System.out.println(studentMapper.getStudentById(1));
    System.out.println(studentMapper.getStudentById(1));
    System.out.println(studentMapper.getStudentById(1));
}
```
![9e996384](https://gitee.com/caijingquan/imagebed/raw/master/1602318952_20200306153725190_1055389169.jpg)
我们可以看到，只有第一次真正查询了数据库，后续的查询使用了一级缓存。

## 实验2
增加了对数据库的修改操作，验证在一次数据库会话中，如果对数据库发生了修改操作，一级缓存是否会失效。
```java
@Test
public void addStudent() throws Exception {
    SqlSession sqlSession = factory.openSession(true); // 自动提交事务
    StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
    System.out.println(studentMapper.getStudentById(1));
    System.out.println("增加了" + studentMapper.addStudent(buildStudent()) + "个学生");
    System.out.println(studentMapper.getStudentById(1));
    sqlSession.close();
}
```
![fb6a78e0](https://gitee.com/caijingquan/imagebed/raw/master/1602318952_20200306153844018_354398559.jpg)
我们可以看到，在修改操作后执行的相同查询，查询了数据库，一级缓存失效。
## 实验3
开启两个SqlSession，在sqlSession1中查询数据，使一级缓存生效，在sqlSession2中更新数据库，验证一级缓存只在数据库会话内部共享。
```java
@Test
public void testLocalCacheScope() throws Exception {
    SqlSession sqlSession1 = factory.openSession(true); 
    SqlSession sqlSession2 = factory.openSession(true); 

    StudentMapper studentMapper = sqlSession1.getMapper(StudentMapper.class);
    StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);

    System.out.println("studentMapper读取数据: " + studentMapper.getStudentById(1));
    System.out.println("studentMapper读取数据: " + studentMapper.getStudentById(1));
    System.out.println("studentMapper2更新了" + studentMapper2.updateStudentName("小岑",1) + "个学生的数据");
    System.out.println("studentMapper读取数据: " + studentMapper.getStudentById(1));
    System.out.println("studentMapper2读取数据: " + studentMapper2.getStudentById(1));
}
```
![f480ac76](https://gitee.com/caijingquan/imagebed/raw/master/1602318953_20200306153949709_1068302639.jpg)
sqlSession2更新了id为1的学生的姓名，从凯伦改为了小岑，但session1之后的查询中，id为1的学生的名字还是凯伦，出现了脏数据，也证明了之前的设想，一级缓存只在数据库会话内部共享。

# 一级缓存工作流程&源码分析
那么，一级缓存的工作流程是怎样的呢？我们从源码层面来学习一下。

# 工作流程
一级缓存执行的时序图，如下图所示。

![bb851700](https://gitee.com/caijingquan/imagebed/raw/master/1602318954_20200306154105124_562206636.png)



# 总结
+ MyBatis一级缓存的生命周期和SqlSession一致。
+ MyBatis一级缓存内部设计简单，只是一个没有容量限定的HashMap，在缓存的功能性上有所欠缺。
+ MyBatis的一级缓存最大范围是SqlSession内部，有多个SqlSession或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为Statement。






---

默认情况下，mybatis开启并使用了一级缓存。
单元测试用例：

    /**
     * 开启事务，测试一级缓存效果
     * 缓存命中顺序：二级缓存---> 一级缓存---> 数据库
     **/
    @Test
    @Transactional(rollbackFor = Throwable.class)
    public void testFistCache(){
        // 第一次查询，缓存到一级缓存
        userMapper.selectById(1);
        // 第二次查询，直接读取一级缓存
        userMapper.selectById(1);

    }

可以看到，虽然进行了两次查询，但最终只请求了一次数据库，第二次查询命中了一级缓存，直接返回了数据。
这里有两点需要说明一下：
1、为什么开启事务
由于使用了数据库连接池，默认每次查询完之后自动commite，这就导致两次查询使用的不是同一个sqlSessioin，根据一级缓存的原理，它将永远不会生效。
当我们开启了事务，两次查询都在同一个sqlSession中，从而让第二次查询命中了一级缓存。读者可以自行关闭事务验证此结论。
2、两种一级缓存模式
一级缓存的作用域有两种：session（默认）和statment，可通过设置local-cache-scope 的值来切换，默认为session。
二者的区别在于session会将缓存作用于同一个sqlSesson，而statment仅针对一次查询，所以，local-cache-scope: statment可以理解为关闭一级缓存。

