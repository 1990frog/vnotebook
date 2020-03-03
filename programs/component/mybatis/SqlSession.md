[TOC]

使用Mybatis的主要java接口就是SqlSession。可以通过这个接口来执行命令，获取映射器和管理事务。SqlSession接口源代码如下所示：

# 接口源码
```java
package org.apache.ibatis.session;

import java.io.Closeable;
import java.sql.Connection;
import java.util.List;
import java.util.Map;
import org.apache.ibatis.executor.BatchResult;

public interface SqlSession extends Closeable {
    <T> T selectOne(String var1);

    <T> T selectOne(String var1, Object var2);

    <E> List<E> selectList(String var1);

    <E> List<E> selectList(String var1, Object var2);

    <E> List<E> selectList(String var1, Object var2, RowBounds var3);

    <K, V> Map<K, V> selectMap(String var1, String var2);

    <K, V> Map<K, V> selectMap(String var1, Object var2, String var3);

    <K, V> Map<K, V> selectMap(String var1, Object var2, String var3, RowBounds var4);

    void select(String var1, Object var2, ResultHandler var3);

    void select(String var1, ResultHandler var2);

    void select(String var1, Object var2, RowBounds var3, ResultHandler var4);

    int insert(String var1);

    int insert(String var1, Object var2);

    int update(String var1);

    int update(String var1, Object var2);

    int delete(String var1);

    int delete(String var1, Object var2);

    void commit();

    void commit(boolean var1);

    void rollback();

    void rollback(boolean var1);

    List<BatchResult> flushStatements();

    void close();

    void clearCache();

    Configuration getConfiguration();

    <T> T getMapper(Class<T> var1);

    Connection getConnection();
}
```
# 执行命令
执行命令指的是执行增删改查等数据库操作。如源代码中的select、insert、update、delete等方法，这些方法被用来执行定义在SQL映射的XML文件中的SELECT、INSERT、UPDATE、DELETE语句。

# 管理事务
在SqlSession中定义了4个方法来控制事务，如果你设置了事务自动提交或者使用外部的事务管理，这些方法不会起作用。但是如果你使用了JDBC的事务管理，这些方法就会起作用。

```java
void commit();
void commit(boolean var1);
void rollback();
void rollback(boolean var1);
```

# 缓存
Mybatis使用2种缓存：本地缓存（一级缓存）和二级缓存。

每当创建一个会话，mybatis就会创建一个本地缓存，并将本地缓存与该会话关联。在会话中执行过的任何查询语句都会被保存在本地缓存中，保存在本地缓存中后，以后相同的查询语句和相同的参数所产生的更改就不会二度影响数据库了。当数据库更新、提交事务、关闭事务以及关闭session时，本地缓存会被清空。

默认情况下，本地缓存数据可在整个session的周期内使用，这一缓存需要被用来解决循环引用错误和加快重复嵌套查询的速度，所以它可以不被禁用掉，但是你可以设置localCacheScope=STATEMENT表示缓存仅在语句执行时有效。

注意，如果localCacheScope（本地缓存作用域） 设置为SESSION，MyBatis所返回的引用是保存在本地缓存里相同对象的引用，而非内存中对象的引用。所以当你对返回的对象做修改时（比如循环list，对某个参数做了一些修改）。将会影响本地缓存中相同内容，进而影响存活在 session 生命周期中的缓存所返回的值。因此，不要对MyBatis所返回的对象作出更改。

# 清除缓存
当想清除缓存时，可以调用clearCache方法。
```java
void clearCache();
```

# 映射器
源代码中的insert、update、delete 和 select方法都很强大，但是使用起来比较繁琐，而且可能会产生类型安全问题。直接使用SqlSession，需要先获取实体类对应mapper文件的namespace，然后还需要自己去根据需要选择调用selectList还是selectOne，还需要根据实际需要组装各类参数，最后还涉及类型转换等。所以说繁琐。

因此，执行映射语句更通用的方式来是使用映射器类。一个映射器类就是一个仅需声明与 SqlSession 方法相匹配的方法的接口类。下面的示例展示了一些方法签名以及它们是如何映射到 SqlSession 上的。

```java
public interface AuthorMapper {
  // (Author) selectOne("selectAuthor",5);
  Author selectAuthor(int id);

  // (List<Author>) selectList(“selectAuthors”)
  List<Author> selectAuthors();

  // (Map<Integer,Author>) selectMap("selectAuthors", "id")
  @MapKey("id")
  Map<Integer, Author> selectAuthors();

  // insert("insertAuthor", author)
  int insertAuthor(Author author);

  // updateAuthor("updateAuthor", author)
  int updateAuthor(Author author);

  // delete("deleteAuthor",5)
  int deleteAuthor(int id);
}
```

总之，每个映射器方法签名应该匹配相关联的 SqlSession 方法，而字符串参数 ID 无需匹配。相反，方法名必须匹配映射语句的 ID。方法签名，在例子中就是Author selectAuthor(int id);

此外，返回类型必须匹配期望的结果类型，单返回值时为指定类，多返回值时为数组或集合。所有常用的类型都是支持的，包括：原生类型、Map、POJO 和 JavaBean。
