[TOC]

# Mybatis缓存的作用
每当我们使用MyBatis开启一次和数据库的会话，MyBatis会创建出一个SqlSession对象表示一次数据库会话。

在对数据库的一次会话中，我们有可能会反复地执行完全相同的查询语句，如果不采取一些措施的话，每一次查询都会查询一次数据库,而我们在极短的时间内做了完全相同的查询，那么它们的结果极有可能完全相同，由于查询一次数据库的代价很大，这有可能造成很大的资源浪费。

为了解决这一问题，减少资源的浪费，MyBatis会在表示会话的SqlSession对象中建立一个简单的缓存，将每次查询到的结果结果缓存起来，当下次查询的时候，如果判断先前有个完全一样的查询，会直接从缓存中直接将结果取出，返回给用户，不需要再进行一次数据库查询了。

mybatis的缓存有一级缓存和二级缓存。

# 一级缓存
一级缓存是默认开启的，作用域是session级别的，缓存的key格式如下：
`cache key: id + sql + limit + offset`
在commit之前，第一次查询结果换以key value的形式存起来，如果有相同的key进来，直接返回value，这样有助于减轻数据的压力。
```java
org.apache.ibatis.executor.BaseExecutor#createCacheKey

public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql) {
    if (this.closed) {
        throw new ExecutorException("Executor was closed.");
    } else {
        CacheKey cacheKey = new CacheKey();
        cacheKey.update(ms.getId());
        cacheKey.update(rowBounds.getOffset());
        cacheKey.update(rowBounds.getLimit());
        cacheKey.update(boundSql.getSql());
        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
        TypeHandlerRegistry typeHandlerRegistry = ms.getConfiguration().getTypeHandlerRegistry();

        for(int i = 0; i < parameterMappings.size(); ++i) {
            ParameterMapping parameterMapping = (ParameterMapping)parameterMappings.get(i);
            if (parameterMapping.getMode() != ParameterMode.OUT) {
                String propertyName = parameterMapping.getProperty();
                Object value;
                if (boundSql.hasAdditionalParameter(propertyName)) {
                    value = boundSql.getAdditionalParameter(propertyName);
                } else if (parameterObject == null) {
                    value = null;
                } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                    value = parameterObject;
                } else {
                    MetaObject metaObject = this.configuration.newMetaObject(parameterObject);
                    value = metaObject.getValue(propertyName);
                }

                cacheKey.update(value);
            }
        }

        if (this.configuration.getEnvironment() != null) {
            cacheKey.update(this.configuration.getEnvironment().getId());
        }

        return cacheKey;
    }
}
```
查询数据库并存入一级缓存的语句
```java
org.apache.ibatis.executor.BaseExecutor#queryFromDatabase

private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    this.localCache.putObject(key, ExecutionPlaceholder.EXECUTION_PLACEHOLDER);

    List list;
    try {
        list = this.doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
        this.localCache.removeObject(key);
    }

    //将查询出来的结果存入一级缓存
    this.localCache.putObject(key, list);
    if (ms.getStatementType() == StatementType.CALLABLE) {
     //如果是存储过程把参数存入localOutputParameterCache
        this.localOutputParameterCache.putObject(key, parameter);
    }

    return list;
}
```
并且当commit或者rollback的时候会清除缓存，并且当执行insert、update、delete的时候也会清除缓存。

相关源码：
```java
org.apache.ibatis.executor.BaseExecutor#update

public int update(MappedStatement ms, Object parameter) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
    if (this.closed) {
        throw new ExecutorException("Executor was closed.");
    } else {
        //删除一级缓存
        this.clearLocalCache();
        return this.doUpdate(ms, parameter);
    }
}


 org.apache.ibatis.executor.BaseExecutor#commit
 
 public void commit(boolean required) throws SQLException {
    if (this.closed) {
        throw new ExecutorException("Cannot commit, transaction is already closed");
    } else {
        //删除一级缓存
        this.clearLocalCache();
        this.flushStatements();
        if (required) {
            this.transaction.commit();
        }

    }
}

org.apache.ibatis.executor.BaseExecutor#rollback

public void rollback(boolean required) throws SQLException {
    if (!this.closed) {
        try {
            //删除一级缓存
            this.clearLocalCache();
            this.flushStatements(true);
        } finally {
            if (required) {
                this.transaction.rollback();
            }

        }
    }

}
```
# 二级缓存
二级缓存是手动开启的，作用域为sessionfactory（也可以说MapperStatement级缓存，也就是一个namespace就会有一个缓存），因为二级缓存的数据不一定都是存储到内存中，它的存储介质多种多样，实现二级缓存的时候，MyBatis要求返回的POJO必须是可序列化的，也就是要求实现Serializable接口，如果存储在内存中的话，实测不序列化也可以的。

如果开启了二级缓存的话，你的Executor将会被装饰成CachingExecutor，缓存是通过CachingExecutor来操作的，查询出来的结果会存在statement中的cache中，若有更新，删除类的操作默认就会清空该MapperStatement的cache（也可以通过修改xml中的属性，让它不执行），不会影响其他的MapperStatement。

相关源码：
```java
org.apache.ibatis.session.Configuration#newExecutor(org.apache.ibatis.transaction.Transaction, org.apache.ibatis.session.ExecutorType)

public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? this.defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Object executor;
    if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
    } else {
        executor = new SimpleExecutor(this, transaction);
    }

    //是否开启缓存，传入的参数为SimpleExecutor
    if (this.cacheEnabled) {
        executor = new CachingExecutor((Executor)executor);
    }

    //责任链模式拦截器
    Executor executor = (Executor)this.interceptorChain.pluginAll(executor);
    return executor;
}
```
query
```java
org.apache.ibatis.executor.CachingExecutor#query(org.apache.ibatis.mapping.MappedStatement, java.lang.Object, org.apache.ibatis.session.RowBounds, org.apache.ibatis.session.ResultHandler)

public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    BoundSql boundSql = ms.getBoundSql(parameterObject);
    CacheKey key = this.createCacheKey(ms, parameterObject, rowBounds, boundSql);
    return this.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}

org.apache.ibatis.executor.CachingExecutor#query(org.apache.ibatis.mapping.MappedStatement, java.lang.Object, org.apache.ibatis.session.RowBounds, org.apache.ibatis.session.ResultHandler, org.apache.ibatis.cache.CacheKey, org.apache.ibatis.mapping.BoundSql)

public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    //获得该MappedStatement的cache
    Cache cache = ms.getCache();
    //如果缓存不为空
    if (cache != null) {
        //看是否需要清除cache（在xml中可以配置flushCache属性决定何时清空cache）
        this.flushCacheIfRequired(ms);
        //若开启了cache且resultHandler 为空
        if (ms.isUseCache() && resultHandler == null) {
            this.ensureNoOutParams(ms, parameterObject, boundSql);
            //从TransactionalCacheManager中取cache
            List<E> list = (List)this.tcm.getObject(cache, key);
            //若取出来list是空的
            if (list == null) {
                //查询数据库
                list = this.delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                //将结果存入cache中
                this.tcm.putObject(cache, key, list);
            }

            return list;
        }
    }
    //如果缓存为空，去查询数据库
    return this.delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```
对于
```java
this.tcm.getObject(cache, key);
```
因同一个namespace下的MappedStatement的cache是同一个，而TransactionalCacheManager中统一管理cache是里面的属性transactionalCaches,该属性以MappedStatement中的Cache为key，TransactionalCache对象为Value。即一个namespace对应一个TransactionalCache。

相关源码：

TransactionalCacheManager
```java
org.apache.ibatis.cache.TransactionalCacheManager

public class TransactionalCacheManager {
private Map<Cache, TransactionalCache> transactionalCaches = new HashMap();

...
}
```
TransactionalCache
```java
org.apache.ibatis.cache.decorators.TransactionalCache

public class TransactionalCache implements Cache {
    private static final Log log = LogFactory.getLog(TransactionalCache.class);
    //namespace中的cache
    private Cache delegate;
    //提交的时候清除cache的标志位
    private boolean clearOnCommit;
    //待提交的集合
    private Map<Object, Object> entriesToAddOnCommit;
    //未查到的key存放的集合
    private Set<Object> entriesMissedInCache;

...
}
```
update
```java
//更新
org.apache.ibatis.executor.CachingExecutor#update

public int update(MappedStatement ms, Object parameterObject) throws SQLException {
     //看是否需要清除cache（在xml中可以配置flushCache属性决定何时清空cache）
    this.flushCacheIfRequired(ms);
    return this.delegate.update(ms, parameterObject);
}

org.apache.ibatis.executor.CachingExecutor#flushCacheIfRequired

private void flushCacheIfRequired(MappedStatement ms) {
    //获得cache
    Cache cache = ms.getCache();
    //若isFlushCacheRequired为true，则清除cache
    if (cache != null && ms.isFlushCacheRequired()) {
        this.tcm.clear(cache);
    }

}
```
# 一级、二级缓存测试
因为一级缓存是默认生效的，下面是二级缓存开启步骤。

mybatis-config.xml
```xml
<settings>
    <!--这个配置使全局的映射器(二级缓存)启用或禁用缓存-->
    <setting name="cacheEnabled" value="true" />
</settings>
```
在mapper.xml可以进行如下的配置
```xml
<mapper>
   <!--开启本mapper的namespace下的二级缓存-->
    <!--
        eviction:代表的是缓存回收策略，目前MyBatis提供以下策略。
        (1) LRU,最近最少使用的，一处最长时间不用的对象
        (2) FIFO,先进先出，按对象进入缓存的顺序来移除他们
        (3) SOFT,软引用，移除基于垃圾回收器状态和软引用规则的对象
        (4) WEAK,弱引用，更积极的移除基于垃圾收集器状态和弱引用规则的对象。这里采用的是LRU，
                移除最长时间不用的对形象
  
        flushInterval:刷新间隔时间，单位为毫秒，这里配置的是100秒刷新，如果你不配置它，那么当
        SQL被执行的时候才会去刷新缓存。
  
        size:引用数目，一个正整数，代表缓存最多可以存储多少个对象，不宜设置过大。设置过大会导致内存溢出。
        这里配置的是1024个对象
  
        readOnly:只读，意味着缓存数据只能读取而不能修改，这样设置的好处是我们可以快速读取缓存，缺点是我们没有
        办法修改缓存，他的默认值是false，不允许我们修改
    -->
    <cache eviction="LRU" flushInterval="100000" readOnly="true" size="1024"/>

    <!--刷新二级缓存-->
  <update id="updateByPrimaryKey" parameterType="com.demo.mybatis.pojo.User" flushCache="true">
    update user
    set name = #{name,jdbcType=VARCHAR},
      age = #{age,jdbcType=INTEGER}
    where id = #{id,jdbcType=INTEGER}
  </update>

   <!--可以通过设置useCache来规定这个sql是否开启缓存，ture是开启，false是关闭-->
   <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap"   useCache="true" >
     select
     <include refid="Base_Column_List" />
     from user
     where id = #{id,jdbcType=INTEGER}
   </select>
</mapper>
```
其中仅仅添加下面这个也可以
```xml
<cache/>
```
如果我们配置了二级缓存就意味着：
+ 映射语句文件中的所有select语句将会被缓存。
+ 映射语句文件中的所欲insert、update和delete语句会刷新缓存。
+ 缓存会使用默认的Least Recently Used（LRU，最近最少使用的）算法来收回。
+ 根据时间表，比如No Flush Interval,（CNFI没有刷新间隔），缓存不会以任何时间顺序来刷新。
+ 缓存会存储列表集合或对象(无论查询方法返回什么)的1024个引用。
+ 缓存会被视为是read/write(可读/可写)的缓存，意味着对象检索不是共享的，而且可以安全的被调用者修改，不干扰其他调用者或线程所做的潜在修改。

User.java
```java
public class User implements Serializable {
    private Integer id;

    private String name;

    private Integer age;

    private static final long serialVersionUID = 1L;
    ...
    set/get
    ...
}
```
UserMapper.java
```java
User selectByPrimaryKey(Integer id);
```
测试方法
```java
@Test
public void test03() throws IOException {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    //第一次
    User user = userMapper.selectByPrimaryKey(1);
    System.out.println("user1 => " + user.toString());
    //第二次
    User user2 = userMapper.selectByPrimaryKey(1);
    System.out.println("user2 => " + user2.toString());
    //session提交
    sqlSession.commit();
    SqlSession sqlSession2 = sqlSessionFactory.openSession();
    UserMapper userMapper2 = sqlSession2.getMapper(UserMapper.class);
    //第三次
    User user3 = userMapper2.selectByPrimaryKey(1);
    System.out.println("user3 => " + user3.toString());
    //第四次
    User user4 = userMapper2.selectByPrimaryKey(1);
    System.out.println("user4 => " + user4.toString());
    sqlSession2.commit();
}
```
来看下结果
```
DEBUG 2019-01-30 00:01:29791 Opening JDBC Connection
DEBUG 2019-01-30 00:01:34688 Created connection 1121453612.
DEBUG 2019-01-30 00:01:34689 Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@42d8062c]
DEBUG 2019-01-30 00:01:34691 ==>  Preparing: select id, name, age from user where id = ? 
DEBUG 2019-01-30 00:01:34737 ==> Parameters: 1(Integer)
DEBUG 2019-01-30 00:01:34757 <==      Total: 1
user1 => User{id=1, name='ayang', age=18}
DEBUG 2019-01-30 00:01:34757 Cache Hit Ratio [com.demo.mybatis.mapper.UserMapper]: 0.0
user2 => User{id=1, name='ayang', age=18}
DEBUG 2019-01-30 00:01:34818 Cache Hit Ratio [com.demo.mybatis.mapper.UserMapper]: 0.3333333333333333
user3 => User{id=1, name='ayang', age=18}
DEBUG 2019-01-30 00:01:34819 Cache Hit Ratio [com.demo.mybatis.mapper.UserMapper]: 0.5
user4 => User{id=1, name='ayang', age=18}
```
可以看到第一次和第二次走的是一级缓存，第三次和第四次走的是二级缓存。

# 总结
一级缓存是自动开启的，sqlSession级别的缓存，查询结果存放在BaseExecutor中的localCache中。

如果第一次做完查询，接着做一次update | insert | delete | commit | rollback操作，则会清除缓存，第二次查询则继续走数据库。

对于一级缓存不同的sqlSession之间的缓存是互相不影响的。

二级缓存是手动开启的，作用域为sessionfactory，也可以说MapperStatement级缓存，也就是一个namespace（mapper.xml）就会有一个缓存，不同的sqlSession之间的缓存是共享的。

因为二级缓存的数据不一定都是存储到内存中，它的存储介质多种多样，实现二级缓存的时候，MyBatis要求返回的POJO必须是可序列化的，也就是要求实现Serializable接口，如果存储在内存中的话，实测不序列化也可以的。

一般为了避免出现脏数据，所以我们可以在每一次的insert | update | delete操作后都进行缓存刷新，也就是在Statement配置中配置flushCache属性，如下：
```xml
<!--刷新二级缓存  flushCache="true"-->
  <update id="updateByPrimaryKey" parameterType="com.demo.mybatis.pojo.User" flushCache="true">
    update user
    set name = #{name,jdbcType=VARCHAR},
      age = #{age,jdbcType=INTEGER}
    where id = #{id,jdbcType=INTEGER}
</update>
```