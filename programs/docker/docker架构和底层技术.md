[TOC]

# Docker是一个Platform
Docker提供了一个开发，打包，运行app的平台
把app和底层infrastructure隔离开来

![](_v_images/20191208143937579_601024278.png)

![](_v_images/20191208144119785_869589766.png)

# 底层技术支持
1. Namespaces：做隔离pid,net,ipc,mnt,uts
2. Control groups：做资源限制
3. Union file systems：Contrainer和image的分层