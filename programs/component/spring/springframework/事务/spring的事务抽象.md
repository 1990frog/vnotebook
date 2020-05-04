[TOC]

结论是：在事务属性为REQUIRED时，在相同线程中进行相互嵌套调用的事务方法工作于相同的事务中。如果互相嵌套调用的事务方法工作在不同线程中，则不同线程下的事务方法工作在独立的事务中。



# 事务抽象的核心接口
```java
public interface PlatformTransactionManager extends TransactionManager {
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
    			throws TransactionException;
    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
```
# 编程式事务
使用编程式事务模板`TransactionTemplate`
```java
@Slf4j
@Controller
public class ProgrammaticTransaction {

    @Autowired
    private TransactionTemplate transactionTemplate;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    /**
     * sql无返回结果
     */
    @GetMapping("/transaction/execute/program")
    @ResponseBody
    public Result execute(){
        log.info("dbsize is {}",getCount());
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
                jdbcTemplate.execute("INSERT INTO transaction (id, code, name) VALUES (1, 1, 'aaa')");
                log.info("dbsize is {}",getCount());
                transactionStatus.setRollbackOnly();
            }
        });

        log.info("dbsize is {}",getCount());
        return Result.builder(null);
    }

    /**
     * sql有返回结果
     */
    @GetMapping("/transaction/query/program")
    @ResponseBody
    public Result query(){
        // 有返回
        return Result.builder(transactionTemplate.execute((TransactionCallback<List>) status -> jdbcTemplate.queryForList("select * from transaction")));
    }

    private long getCount() {
        return (long) jdbcTemplate.queryForList("SELECT COUNT(*) AS CNT FROM transaction")
                .get(0).get("CNT");
    }
}
```
# 声明式事务
## 开启事务注解的方式
`@EnableTransactionManagement`加在启动类上
## @Transactional注解
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {

	@AliasFor("transactionManager")
	String value() default "";
	@AliasFor("value")
	String transactionManager() default "";
    // 事务的传播行为，默认required，如果有事务就复用，没有就new个新的
	Propagation propagation() default Propagation.REQUIRED;
	// 事务的隔离级别，默认使用数据库的，不需要进行设置，数据库底层不支持设置了也没毛用
	Isolation isolation() default Isolation.DEFAULT;
	// 事务的超时时间，默认-1永不超时
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
	// 该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false
	boolean readOnly() default false;
	// 根据指定异常触发回滚，这有个神坑，默认设置为RuntimeException
	// 指定多个异常类名称：@Transactional(rollbackForClassName={"RuntimeException","Exception"})
	Class<? extends Throwable>[] rollbackFor() default {};
	// rollbackForClassName={"RuntimeException","Exception"}设置根据异常名回滚
	String[] rollbackForClassName() default {};
	// 取反
	Class<? extends Throwable>[] noRollbackFor() default {};
	// 取反
	String[] noRollbackForClassName() default {};
}
```
## 声明式事务大坑一：默认rollbackFor
<font color="red">最常见的坑，就是@Transactional注解中如果不配置rollbackFor属性,那么事物只会在遇到RuntimeException的时候才会回滚</font>
## 声明式事务大坑二：aop内部失效问题
[aop内部调用失效问题](aop内部调用失效问题.md)
# 事务的传播特性
1. required（常用）：如果有事务, 那么加入事务, 没有的话新建一个(默认情况下)
2. not_supported（常用）：容器不为这个方法开启事务
3. requires_new（常用）：不管是否存在事务,都创建一个新的事务,原来的挂起,新的执行完毕,继续执行老的事务
4. mandatory：必须在一个已有的事务中执行,否则抛出异常
5. never：必须在一个没有的事务中执行,否则抛出异常(与Propagation.MANDATORY相反)
6. supports：如果其他bean调用这个方法,在其他bean中声明事务,那就用事务.如果其他bean没有声明事务,那就不用事务
# 事务嵌套的问题
```java
/**
 * 这个类用于测试事务的传播
 *
 * 事务嵌套调用：
 * 1.required调用not_supported，在方法里触发异常：都未插入数据，说明required的事务传递给了not_supported
 * 2.not_supported调用required，在方法里触发异常：有一条数据not_supported，required未插入到数据库
 *
 * 结论：
 * 事务可以从上游传递给下游？
 */
@Component
public class PropagationTr {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    /**
     * 事务传播等级：
     * required（默认事务传播）
     *
     * 自我介绍：
     * “能用就凑合，没有在弄个新的”
     */
    @Transactional(propagation = Propagation.REQUIRED)
    public void required() {
        jdbcTemplate.execute("INSERT INTO transaction (id, code, name) VALUES (1, 1, 'REQUIRED')");
//        ((PropagationTr) AopContext.currentProxy()).notSupported();//通过required去调用notSupported
        throw new RuntimeException();
    }

    /**
     * 事务传播等级：
     * not_supported
     *
     * 自我介绍：
     * “就是不用，有也不用”
     */
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void notSupported(){
        jdbcTemplate.execute("INSERT INTO transaction (id, code, name) VALUES (2, 2, 'NOT_SUPPORTED')");
//        ((PropagationTr) AopContext.currentProxy()).required();
        throw new RuntimeException();
    }

    /**
     * 事务传播等级requires_new
     *
     * 自我介绍：
     * “不管有没有，我都要新的事务”
     */
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void requiresNew(){
        jdbcTemplate.execute("INSERT INTO transaction (id, code, name) VALUES (3, 3, 'REQUIRES_NEW')");
    }

}
```
## 
# 事务隔离特性
|           隔离值           | 值  |  脏读  | 不可重复读 | 幻读 |
| ------------------------- | --- | ----- | -------- | --- |
| ISOLATION_READ_UNCOMMITTED | 1   | 允许   | 允许      | 允许 |
| ISOLATION_READ_COMMITTED   | 2   | 不允许 | 允许      | 允许 |
| ISOLATION_REPEATABLE_READ  | 3   |    不允许   |    不允许      | 允许 |
| ISOLATION_SERIALIZABLE     | 4   |    不允许   |   不允许       |  不允许   |

默认为：-1由数据库决定
ProgrammaticTransaction


