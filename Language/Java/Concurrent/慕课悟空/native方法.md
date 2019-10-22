[TOC]

# native方法

native方法（由c或cpp编写），如果想看源码，去网站看openjdk实际实现

# 如何分析native方法

进入github（也可以进入 openJDK网站 ）
点“搜索文件”，搜索对应的c代码类Thread.c
找到native方法对应的方法名
去src/hotspot/share/prims/jvm.cpp里看cpp代码

![20190905205555461_716593482](_v_images/20191022142643768_1313504779.png)