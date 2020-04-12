[TOC]

# 配置
```yml
spring:
  application:
    # 服务名称尽量用-，不要用_，不要用特殊字符
    name: customer
  cloud:
    nacos:
      discovery:
        # 指定nacos server的地址
        server-addr: localhost:8848
    # 向sentinel控制台注册服务
    sentinel:
      filter:
        enabled: false
      transport:
        # port端口配置会在应用对应的机器上启动一个 Http Server，该 Server 会与 Sentinel 控制台做交互。
        # 比如 Sentinel 控制台添加了一个限流规则，会把规则数据 push 给这个 Http Server 接收，Http Server 再将规则注册到 Sentinel 中。
        # 如果不设置，会自动从8719开始扫描，依次+1，直到找到未被占用的端口
        port: 8720
        # 指定控制台地址
        dashboard: localhost:8080
        # 心跳发送周期，默认值null
        # 但在simpleHttpHeartbeatSender会默认值10秒
        heartbeat-interval-ms: 10000

    stream:
      rocketmq:
        binder:
          name-server: localhost:9876
        bindings:
          # 指定channel，这里可以设置多个channel
          output1:
            producer:
              transactional: true
              group: tx-group
          output2:
            producer:
              transactional: true
              group: tx-group
      bindings:
        output:
          # 用来指定topic
          destination: stream-test-topic
        # 自定义输出接口，这个接口我定义了3个接口，分别发布消息到不同的topic
        output1:
          destination: topic1
        output2:
          destination: topic2
        output3:
          destination: topic3
        input:
          destination: stream-test-topic
          # 这里group一定要设置
          group: bind-group
        input1:
          destination: topic1
          group: group1
        input2:
          destination: topic2
          group: group2
        input3:
          destination: topic3
          group: group3
```
stream发送事务消息，与发送正常消息一样
rocketmqTemplate有一个参数提供给我们设置流水号之类的东东，stream不支持，但是stream可以在setHeader中设置
```java
/**
 * productor事务
 */
@PutMapping("/stream/tx")
public void tx() {
    mySource.output1().send(MessageBuilder
            .withPayload("this is my transational message!")
            .setHeader("SerialNumber", "001")
            .build());
}
```
实现RocketMQLocalTransactionListener接口
该接口有两个方法：
+ executeLocalTransaction：执行本地事务
+ checkLocalTransaction：检查本地事务
```java
@Slf4j
@RocketMQTransactionListener(txProducerGroup = "tx-group")
public class StreamRocketMQLocalTransationalListener implements RocketMQLocalTransactionListener {

    /**
     * 执行本地事务，如果执行成功发送COMMIT给MQ完成二次确认，让消息投递，反之，ROLLBACK
     * @param message
     * @param o
     * @return
     */
    @Override
    public RocketMQLocalTransactionState executeLocalTransaction(Message message, Object o) {
        try{
            // 这做个事务
            log.info("执行事务！");
            return RocketMQLocalTransactionState.COMMIT;
        }catch (Exception e){
            return RocketMQLocalTransactionState.ROLLBACK;
        }
    }

    @Override
    public RocketMQLocalTransactionState checkLocalTransaction(Message message) {
        MessageHeaders messageHeaders = message.getHeaders();
        String serialNumber = (String)messageHeaders.get("SerialNumber");
        log.info("通过业务流水号：{}，清除之前操作的数据",serialNumber);
        return RocketMQLocalTransactionState.ROLLBACK;
    }
}
```