[TOC]

# RocketMQTransactionListener
```java
@Target({ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface RocketMQTransactionListener {

    /**
     * Declare the txProducerGroup that is used to relate callback event to the listener, rocketMQTemplate must send a
     * transactional message with the declared txProducerGroup.
     * <p>
     * <p>It is suggested to use the default txProducerGroup if your system only needs to define a TransactionListener class.
     */
    String txProducerGroup() default RocketMQConfigUtils.ROCKETMQ_TRANSACTION_DEFAULT_GLOBAL_NAME;

    /**
     * Set ExecutorService params -- corePoolSize
     */
    int corePoolSize() default 1;

    /**
     * Set ExecutorService params -- maximumPoolSize
     */
    int maximumPoolSize() default 1;

    /**
     * Set ExecutorService params -- keepAliveTime
     */
    long keepAliveTime() default 1000 * 60; //60ms

    /**
     * Set ExecutorService params -- blockingQueueSize
     */
    int blockingQueueSize() default 2000;

    /**
     * The property of "access-key"
     */
    String accessKey() default "${rocketmq.producer.access-key}";

    /**
     * The property of "secret-key"
     */
    String secretKey() default "${rocketmq.producer.secret-key}";
}
```
可以设置为多线程并发的