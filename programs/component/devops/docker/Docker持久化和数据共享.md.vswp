vnote_backup_file_826537664 /home/cai/Documents/vnotebook/programs/docker/Docker持久化和数据共享.md
![](_v_images/20200101192617296_518751380.png)

# Docker持久化数据的方案
+ 基于本地文件系统的Volume。可以在执行Docker create或Docker run时，通过-v参数将本机的目录作为容器的数据卷。这部分功能便基于本地文件系统的volume管理。
+ 基于plugin的Volume，支持第三方的存储方案，比如NAS，aws

# Volume的类型
+ 受管理的data Volume，由docker后台自动创建。
+ 绑定挂载的Volume，具体挂载位置可以由用户指定。

sudo docker logs xxx 查看日志
sudo docker volume ls

sudo docker run -d -v mysql:/var/lib/mysql --name mysql -e xxx
指定volume名字

mysql -u root
进入mysql的shell

# Bind Mouting

docker run -v /home/aaa:/root/aaa

docker rm -f xxx 强制删除container

docker run -d -v $(pwd):/usr/share/nginx/html -p 80:80 --name web xxxxx


# demo
![](_v_images/20200101205418663_44131556.png)


