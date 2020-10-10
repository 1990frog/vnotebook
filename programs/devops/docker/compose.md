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
# Services
+ 一个service代表一个container，这个container可以从dockerhub的image来创建，或者从本地的DockerFile build出来的image来创建
+ service的启动类似docker run，我们可以给其指定network和volume，所以可以给service指定network和volume的引用




```
docker-compose -f docker-compose.yml up
docker-compose up

docker-compose ps

docker-compose start
docker-compose stop/down

docker-compose exec mysql bash
```

# docker-compose scale
![](https://gitee.com/caijingquan/imagebed/raw/master/1602321549_20200112151017585_60943431.png)

# 部署一个复杂的投票应用
docker-compose build
docker-compose up

# docker-compose是一个用于本地开发的工具，不是 一个用于生产环境的工具

---

# docker-compose.yml
```docker
version: "3"
services:
  voting-app:
    build: ./voting-app/.
    volumes:
     - ./voting-app:/app
    ports:
      - "5000:80"
    links:
      - redis
    networks:
      - front-tier
      - back-tier

  result-app:
    build: ./result-app/.
    volumes:
      - ./result-app:/app
    ports:
      - "5001:80"
    links:
      - db
    networks:
      - front-tier
      - back-tier

  worker:
    build: ./worker
    links:
      - db
      - redis
    networks:
      - back-tier

  redis:
    image: redis
    ports: ["6379"]
    networks:
      - back-tier

  db:
    image: postgres:9.4
    volumes:
      - "db-data:/var/lib/postgresql/data"
    networks:
      - back-tier

volumes:
  db-data:

networks:
  front-tier:
  back-tier:
```