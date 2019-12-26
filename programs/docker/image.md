[TOC]

# 什么是Image
1. 文件和meta data的集合（root filesystem）
2. 分层的，并且每一层都可以添加改变删除文件，成为一个新的image
3. 不同的image可以共享相同的layer
4. image本身是read-only的
![未命名文件](_v_images/20191209104945691_1607632261.png)
+ 所有image共用Linux Kernel(bootfs)
+ base image 也可以共享
+ base image 只包含rootfs，所以image很小



# diy Base Image
```
docker pull hello-world
docker run hello-world
```