# dns解析
www.google.com dns解析 ip地址

域名的解析
www.google.com.root 从右向左解析
.root 根域名（都是root，省略）
.com/.edu/.org 顶级域名
.google/.baidu/.youtube 次级域名
www 主机名

![](_v_images/20191106210437409_2077942516.png)

![](_v_images/20191106210617525_136265005.png)

# 网络协议
![](_v_images/20191106211219981_585778214.png)

应用层：用户的应用（HTTP、FTP、SMTP）
传输层：端口到端口的链接（TCP、UDP）
网络层：主机到主机（IP）
链路层：网卡与网卡（Ethernet）【MAC地址】
实体层：光缆、wifi、网线（电信号）【物理连接】

一帧最大1518byte

![](_v_images/20191106212712423_1506174201.png)

![](_v_images/20191106213151457_331022047.png)

![](_v_images/20191106213226183_1676811856.png)

![](_v_images/20191106213427159_233190066.png)



网络编程的本质是进程间通信

![](_v_images/20191112214248475_24866848.png)

![](_v_images/20191112214549288_785106910.png)

stream 
input
output

reader
write

![](_v_images/20191112215310904_479580990.png)

![](_v_images/20191112220342509_2045257945.png)

![](_v_images/20191112220630293_1260055554.png)

DataOutputStream\DataOutputStream 输出基本数据类型

java.io 设计模式（装饰器模式）

![](_v_images/20191112221712894_224494831.png)

异步同步调用结果

![](_v_images/20191113230215178_1329092770.png)

![](_v_images/20191113230328979_2132466374.png)

# BIO