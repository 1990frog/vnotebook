[TOC]

# 准备
```docker
docker run -d --name mysql -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=test
docker run -d -e WORDPRESS_DB_HOST=mysql:3306 --link mysql -p 8080:80 wordpress
```

# Docker Compose
+ Docker Compose是一个工具
+ 这个工具可以通过一个yml文件定义多容器的docker应用
+ 通过一条命令就可以根据yml文件的定义去创建或者管理这多个容器

# docker-compose.yml
+ Services
+ Networks
+ Volumes

# Docker Compose版本差异

# Services
+ 一个service代表一个container，这个container可以从dockerhub的image来创建，或者从本地的DockerFile build出来的image来创建
+ service的启动类似docker run，我们可以给其指定network和volume，所以可以给service指定network和volume的引用

```docker
services:
    db:
        image:postgres:9.4
    volumes:
        - "db-data:/var/lib/postgresql/data"
    networks:
        - back-tier
```

```docker
services:
    worker:
        build:./worker
        links:
            - db
            - redis
        networks:
            - back-tier
```

![](_v_images/20200109221104403_721488217.png)

# 安装
```
┌─[cai@frog] - [~] - [四 1月 09, 22:13]
└─[$] <> sudo pacman -S docker-compose
[sudo] cai 的密码：
正在解析依赖关系...
正在查找软件包冲突...

软件包 (18) python-asn1crypto-1.2.0-3  python-bcrypt-3.1.7-3
            python-cached-property-1.5.1-4  python-cffi-1.13.2-2
            python-cryptography-2.8-1  python-docker-4.1.0-3
            python-docker-pycreds-0.4.0-5  python-dockerpty-0.4.1-6
            python-jsonschema-3.2.0-1  python-paramiko-2.6.0-3
            python-ply-3.11-4  python-pyasn1-0.4.8-1  python-pycparser-2.19-3
            python-pynacl-1.3.0-3  python-pyrsistent-0.15.6-1
            python-texttable-1.6.2-3  python-websocket-client-0.57.0-1
            docker-compose-1.25.0-1

下载大小:    2.03 MiB
全部安装大小：  13.11 MiB
```

docker-compose -f docker-compose.yml up
docker-compose up

docker-compose ps

docker-compose start
docker-compose stop/down

docker-compose exec mysql bash