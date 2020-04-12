vnote_backup_file_826537664 /home/cai/Documents/vnotebook/programs/component/javafamily/springcloud/rocketmq/事务消息.md
[TOC]

# 流程
1. producer发送半消息给mq
2. mq接受成功发送回执给producer（确认接收到半消息）
3. producer执行本地事务，成功发送commit给mq，失败发送fallback给mq
4. mq收到producer发送的commit或fallback就执行响应的操作，向consumer发送消息，或者退回消息
5. 如果上一步在producer发送确认消息时发生异常，消息发送失败，mq在长时间未收到二次确认就会回查
6. producer检查本地事务状态

# 生产者
第一步与第二步发生异常都会触发回滚，但是第三步发生异常虽然也会触发回滚，但是第二步的消息已经发生出去了
```
@Transactional(rollbackFor = Exception.class)
public void save(){
    // step-1
    db.save();
    // step-2
    mqTemplate.convertAndSend();
    // step-3
    cache();
}
```
# 事务消息

![](_v_images/20200323200909734_523683364.png)
![164e91cb94fab729](_v_images/20200323200142393_2066064305.png)

如果步骤4断网了或者重启了，就有可能导致mqserver没有拿到二次确认的结果