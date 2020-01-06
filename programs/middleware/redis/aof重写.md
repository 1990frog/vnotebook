[TOC]

![](_v_images/20191219151956437_69219163.png)
# 个人总结
AOF重写机制存在的目的就是为了减小AOF文件体积
# AOF 重写机制
+ AOF持久化是通过保存被执行的写命令来记录数据库状态的，所以AOF文件的大小随着时间的流逝一定会越来越大；影响包括但不限于：对于Redis服务器，计算机的存储压力；AOF还原出数据库状态的时间增加（缺陷：还原慢，体积大）
+ 为了**解决AOF文件体积膨胀的问题**，Redis提供了AOF重写功能：Redis服务器可以创建一个新的AOF文件来替代现有的AOF文件，新旧两个文件所保存的数据库状态是相同的，但是新的AOF文件不会包含任何浪费空间的冗余命令，通常体积会较旧AOF文件小很多。
# AOF重写实现两种方式
+ bgrewriteaof命令
+ redis.conf文件中配置AOF重写
## bgrewriteaof命令
AOF重写是将redis内存中的数据进行回朔并写入新的AOF文件
## AOF重写配置
四个参数：
1. auto-aof-rewrite-min-size触发重写文件大小
2. auto-aof-rewrite-percentage触发重写文件增长率
3. aof_current_size当前aof文件大小
4. aof_base_size上次触发重写时文件大小
```
配置:
auto-aof-rewrite-min-size    #AOF文件重写需要的尺寸
auto-aof-rewrite-percentage    #AOF文件增长率


统计
aof_current_size    #AOF当前尺寸（单位：字节）
aof_base_size    #AOF上次启动和重写的尺寸（单位：字节）


自动触发时机
aof_current_size>auto-aof-rewrite-min-size
【当前AOF文件尺寸】大于【AOF文件重写需要的尺寸】
(aof_current_size-aof_base_size)/aof_base_size>auto-aof-rewrite-percentage
当前size减去上次重写size之差除以上次重写size大于AOF文件增长率
```

![](_v_images/20191207144237476_1349180936.png)

# 总结
+ AOF重写的目的是为了解决AOF文件体积膨胀的问题，使用更小的体积来保存数据库状态，整个重写过程基本上不影响Redis主进程处理命令请求
+ **AOF重写其实是一个有歧义的名字，实际上重写工作是针对数据库的当前状态来进行的，重写过程中不会读写、也不适用原来的AOF文件**
+ AOF可以由用户手动触发，也可以由服务器自动触发
