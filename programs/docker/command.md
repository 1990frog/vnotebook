# 非sudo执行docker
```
sudo groupadd docker    #创建docker用户组
sudo gpasswd -a {user} docker    #添加用户到docker组
重启docker,shell生效
```