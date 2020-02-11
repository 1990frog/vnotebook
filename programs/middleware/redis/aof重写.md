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


# 执行流程
子进程在执行AOF重写时，主进程需要执行以下三个工作：
1、处理命令请求；
2、将写命令追加到现有的AOF文件中；
3、将写命令追加到AOF重写缓存中。
如此可以保证：
1）、现有的AOF功能继续执行，即使AOF重写期间发生停机，也不会有任何数据丢失；
2）、所有对数据库进行修改的命令都会被记录到AOF重写缓存中。
当子进程完成对AOF文件重写之后，它会向父进程发送一个完成信号，父进程接到该完成信号之后，会调用一个信号处理函数，该函数完成以下工作：
1）、将AOF重写缓存中的内容全部写入到新的AOF文件中；
2）、对新的AOF文件进行改名，覆盖原有的AOF文件。

当“1）、将AOF重写缓存中的内容全部写入到新的AOF文件中；”执行完毕后，现有AOF文件、新的AOF文件和数据库三者的状态就完全一致了。
当“2）、对新的AOF文件进行改名，覆盖原有的AOF文件。”执行完毕后，程序就完成了新旧两个AOF文件的替换。
当这个信号处理函数执行完毕之后，主进程就可以继续像往常一样接收命令请求了。在整个AOF后台重写过程中，只有最后的“主进程写入命令到AOF缓存”和“对新的AOF文件进行改名，覆盖原有的AOF文件。”这两个步骤会造成主进程阻塞，在其他时候，AOF后台重写都不会对主进程造成阻塞，这将AOF重写对性能造成的影响降到最低。

以上阐述，就是AOF后台重写，也就是BGREWRITEAOF命令的工作原理。

触发AOF后台重写的条件
AOF重写可以由用户通过调用BGREWRITEAOF手动触发。
另外，服务器在AOF功能开启的情况下，会维持以下三个变量：
l 记录当前AOF文件大小的变量aof_current_size。
l 记录最后一次AOF重写之后，AOF文件大小的变量aof_rewrite_base_size。
l 增长百分比变量aof_rewrite_perc。

每次当serverCron（服务器常规操作）函数执行时，它会检查以下条件是否全部满足，如果全部满足的话，就触发自动的AOF重写操作：
1）、没有BGSAVE命令（RDB持久化）/AOF持久化在执行；
2）、没有BGREWRITEAOF在进行；
3）、当前AOF文件大小要大于server.aof_rewrite_min_size（默认为1MB），或者在redis.conf配置了auto-aof-rewrite-min-size大小；
4）、当前AOF文件大小和最后一次重写后的大小之间的比率等于或者等于指定的增长百分比（在配置文件设置了auto-aof-rewrite-percentage参数，不设置默认为100%）

如果前面三个条件都满足，并且当前AOF文件大小比最后一次AOF重写时的大小要大于指定的百分比，那么触发自动AOF重写。
