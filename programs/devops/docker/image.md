[TOC]

# 常用最小镜像
## BusyBox
Busybox是一个集成了一百多个最常用Linux命令和工具的软件工具箱，它在单一的可执行文件中提供了精简的Unix工具集。BusyBox可运行于多款POSIX环境操作系统中，如Linux（包括Andoroid）、Hurd、FreeBSD等。
Busybox既包含了一些简单实用的工具，如cat和echo，也包含了一些更大，更复杂的工具，如grep、find、mount以及telnet。可以说BusyBox是Linux系统的瑞士军刀。
## alpine
## scratch

# 什么是Image
1. 文件和meta data的集合（root filesystem）
2. 分层的，并且每一层都可以添加改变删除文件，成为一个新的image
3. 不同的image可以共享相同的layer
4. image本身是read-only的
![未命名文件](https://gitee.com/caijingquan/imagebed/raw/master/1602321486_20191209104945691_1607632261.png)
+ 所有image共用Linux Kernel(bootfs)
+ base image 也可以共享
+ base image 只包含rootfs，所以image很小

# image与container之间的联系
镜像的概念更多偏向于一个环境包，这个环境包可以移动到任意的Docker平台中去运行；而容器就是你运行环境包的实例。你可以针对这个环境包运行N个实例。换句话说container是images的一种具体表现形式。你也可以认为镜像与你装载操作系统iso镜像是一个概念，容器则可理解为镜像启动的操作系统。一个镜像可以启动任意多个容器，即可以装载多个操作系统。
也可以形容成：类与实例

# dockerhub获取image
```docker
$ docker pull hello-world
$ docker run hello-world
```

# 通过container生成image
```docker
$ docker commit container_id
```

# 通过dockerfile构建image
```docker
$ docker build -t dockerid/dockername .
```
