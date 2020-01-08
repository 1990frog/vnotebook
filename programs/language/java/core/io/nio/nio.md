[TOC]

![BIO_NIO示意图](_v_images/20200103144600961_966811236.png)

# 非阻塞式NIO
1.使用Channel代替Stream

流是单向的：写入、写出
通道是双向的
Channel可阻塞、也可非阻塞

2.使用Selector监控多条Channel
3.可以在一个线程里处理多个Channel I/O

# Channel与Buffer

![20200102213144377_250675781](_v_images/20200103143742834_1977533972.png)

flip()
clear()
compact()

# Channel基本操作
![20200102214415397_393942958](_v_images/20200103143932406_460900779.png)
![20200102214455428_1220963344](_v_images/20200103143943043_645777050.png)