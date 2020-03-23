




二次确认




```sql
create table rocketmq_transaction_log
(
  id             int auto_increment comment 'id'
    primary key,
  transaction_Id varchar(45) not null comment '事务id',
  log            varchar(45) not null comment '日志'
)
comment 'RocketMQ事务日志表';
```

# 发送半消息
sendMessageInTransaction

![](_v_images/20200323130807563_1359789938.png)

idea抽出方法
idea的停止按钮的优雅停机，会将代码执行完成

# Spring Cloud Stream
+ 一个用于构建消息驱动的微服务框架

![](_v_images/20200323132628551_651343438.png)


![](_v_images/20200323132834248_1202764480.png)

![](_v_images/20200323133145976_44308719.png)

