# redis生命周期

![](_v_images/20191125205349355_764546666.png)

# 慢查询
两个配置

slowlog-max-len
1.先进先出队列
2.固定长度
3.保存在内存中

slowlog-log-slower-than
1.慢查询阈值（单位：微秒）
2.slowlog-log-slower-than=0，记录所有命令
3.slowlog-log-slower-than<0，不记录任何命令

默认值
config get slow-max-len=128
config get slowlog-log-slower-than=10000
修改配置文件重启
动态配置
config set slowlog-max-len 1000
config set slowlog-log-slower-than 1000

慢查询命令
1.slowlog get[n] ：获取慢查询队列
2.slowlog len：获取慢查询队列长度
3.slowlog reset：清空慢查询

慢查询运维经验
1.slow-max-len不要设置过大，默认10ms，通常设置1ms
2.slow-log-slower-than不要设置过小，通常设置1000左右
3.理解命令生命周期
4.定期持久化慢查询

# pipeline
