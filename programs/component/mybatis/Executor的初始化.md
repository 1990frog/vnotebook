[TOC]

# mybatis中几个核心的概念名词

|        名称         |                                 意义                                  |
| ------------------ | -------------------------------------------------------------------- |
| Configuration       | 管理 mybatis-config.xml 全局配置关                                       |
| SqlSessionFactor    | Session 管理工厂                                                       |
| Session             | SqlSession 是一个面向用户（程序员）的接口。SqlSession中提供了很多操作数据库的方法 |
| Executor            | 执行器是一个接口（基本执行器、缓存执行器）作用：SqlSession内部通过执行器操作数据库  |
| MappedStatement	 | 底层封装对象作用：对操作数据库存储封装，包括sql语句、输入输出参数                 |
| StatementHandle     | 具体操作数据库相关的 handler 接口                                         |
| ResultSetHandler    | 具体操作数据库返回结果的 handler                                          |

# 看下mybatis的源码结构：
```
├─annotations ->注解相关 比如 select insert
├─binding -> mapper 相关
├─builder ->解析 xml 相关
├─cache ->缓存
├─cursor -> 返回结果 resultset
├─datasourcer ->数据管理
├─exceptionsr -> 异常
├─executorr -> 执行器
├─io ->classloader
├─jdbc ->jdbc
├─lang ->jdk7 jdk8
├─logging ->日志相关
├─mapping ->mapper 相关的封装
├─parsing ->xml 相关解析
├─plugin ->拦截器
├─reflection ->反射相关
├─scripting ->数据厂家
├─session ->sessiomn
├─transaction ->事务
└─type ->返回类型
```

拿我们之前写的demo，查看源码里面是怎么写的，如下面这个。
```java
@Test
public void test01() throws IOException {
    String resource = "mybatis-config.xml";
    Reader reader = Resources.getResourceAsReader(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
    SqlSession session = sqlSessionFactory.openSession();
    User user = session.selectOne("selectByPrimaryKey", 1);
    session.commit();
    logger.info(user.toString());
}
```
这个很好理解，以流的方式读取我们的config文件。
```java
Reader reader = Resources.getResourceAsReader(resource);
```
然后来看SqlSessionFactory是怎么build创建的。（下面只展示核心代码）
```java
new SqlSessionFactoryBuilder().build(reader);
```
进入看一下。
```java
public SqlSessionFactory build(Reader reader) {
    return this.build((Reader)reader, (String)null, (Properties)null);
}
```
继续进入
```java
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
   XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
   SqlSessionFactory  var5 = this.build(parser.parse());
   return var5;
}
```
这时候可以看到实例化了一个核心的类XMLConfigBuilder，然后parser.parse()，进行解析，我们点进去。
```java
public Configuration parse() {
    if (this.parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    } else {
        this.parsed = true;
        this.parseConfiguration(this.parser.evalNode("/configuration"));
        return this.configuration;
    }
}
```
重点看这个
```java
this.parseConfiguration(this.parser.evalNode("/configuration"));
```
继续点进去
```java
private void parseConfiguration(XNode root) {
    try {
        this.propertiesElement(root.evalNode("properties"));
        this.typeAliasesElement(root.evalNode("typeAliases"));
        this.pluginElement(root.evalNode("plugins"));
        this.objectFactoryElement(root.evalNode("objectFactory"));
        this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
        this.reflectionFactoryElement(root.evalNode("reflectionFactory"));
        this.settingsElement(root.evalNode("settings"));
        this.environmentsElement(root.evalNode("environments"));
        this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
        this.typeHandlerElement(root.evalNode("typeHandlers"));
        this.mapperElement(root.evalNode("mappers"));
    } catch (Exception var3) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
    }
}
```
OK，就是我们之前说的mybatis-config.xml里面的属性。

mybatis-config.xml

|               属性名               |                                    	作用                                    |
| --------------------------------- | -------------------------------------------------------------------------- |
| 属性（peoperties）	               | 系统属性占用配置                                                              |
| 设置（settings）                    | 	用于修改Mybatis的运行时行为                                                 |
| 类型别名（typeAliases）	           | 为类型建立别名，一般使用更短的名称代替                                             |
| 类型处理器（typeHanders）	           | 用于将预编译语句（PreparedStatement）或者结果集（ResultSet）中的JDBC类型转换为Java类型 |
| 对象工厂（ObjectFactory）	           | 提供默认构造器或执行构造器参数初始化目标类型的对象                                   |
| 插件（plugins）	                   | Mybatis提供插件的方式来拦截映射（可以根据自己的需求进行编写插件）                      |
| 环境（environments）                | Mybatis运行配置多个环境                                                        |
| 数据库标识提供商（databaseIdProvider） | 	数据库标识提供商                                                              |
| SQL映射文件（mappers）	           | SQL映射文件                                                                  |
中间有将我们的配置封装Node对象
```java
public XNode evalNode(String expression) {
    return this.evalNode(this.document, expression);
}

public XNode evalNode(Object root, String expression) {
    Node node = (Node)this.evaluate(expression, root, XPathConstants.NODE);
    return node == null ? null : new XNode(this, node, this.variables);
}
```
在之后，返回一个DefaultSqlSessionFactory
```java
public SqlSessionFactory build(Configuration config) {
        return new DefaultSqlSessionFactory(config);
}
```
整体的一个调用链是这样的
```java
org.apache.ibatis.session.SqlSessionFactoryBuilder.build(java.io.InputStream)
 >org.apache.ibatis.builder.xml.XMLConfigBuilder 实例构造函数
   >org.apache.ibatis.builder.xml.XMLConfigBuilder.parse
     >org.apache.ibatis.builder.xml.XMLConfigBuilder.parseConfiguration 解析mybatis-config.xml中的内容
       >org.apache.ibatis.parsing.XPathParser.evaluate
        >org.apache.ibatis.builder.xml.XMLConfigBuilder.mapperElement
         >org.apache.ibatis.session.SqlSessionFactoryBuilder.build(org.apache.ibatis.session.Configuration)
          >org.apache.ibatis.session.defaults.DefaultSqlSessionFactory.DefaultSqlSessionFactory 返回一个DefaultSqlSessionFactory
```
整个流程就是返回一个我们下一步需要的DefaultSqlSessionFactory。

下一段代码

```java
SqlSession session = sqlSessionFactory.openSession();
```
同样的，一步步点进去
```java
public SqlSession openSession() {
    return this.openSessionFromDataSource(this.configuration.getDefaultExecutorType(), (TransactionIsolationLevel)null, false);
}
```
继续
```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;

    DefaultSqlSession var8;
    try {
        Environment environment = this.configuration.getEnvironment();
        TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(environment);
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
        Executor executor = this.configuration.newExecutor(tx, execType);
        var8 = new DefaultSqlSession(this.configuration, executor, autoCommit);
    } catch (Exception var12) {
        this.closeTransaction(tx);
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
    } finally {
        ErrorContext.instance().reset();
    }

    return var8;
}
```
其中 transactionFactory.newTransaction 返回一个TransactionFactory
```java
tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
```
然后
```java
Executor executor = this.configuration.newExecutor(tx, execType);
```
返回一个执行器 Executor

Executor：执行器是一个接口（基本执行器、缓存执行器）

作用：SqlSession内部通过执行器操作数据库

然后继续进去
```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? this.defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Object executor;
    //批处理执行器
    if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
    //可重复用的执行器
    } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
    } else {
        executor = new SimpleExecutor(this, transaction);
    }

    //cacheEnabled默认是true，传入的参数是SimpleExecutor，装饰成CachingExecutor
    if (this.cacheEnabled) {
        executor = new CachingExecutor((Executor)executor);
    }

    //这个是拦截器链，责任链模式拦截器，不过需要我们自己实现
    Executor executor = (Executor)this.interceptorChain.pluginAll(executor);
    return executor;
}
```
最后返回给我们一个executor，因为mybatis默认开启缓存，所以返回是CachingExecutor，但是如果没有在mybatis-config.xml中开启二级缓存，在后面的query查询中MappedStatement.getCache()时，返回的还是空的，在之后用delegate.query（...）进行查询，那时executor是SimpleExecutor。

Executor类图



