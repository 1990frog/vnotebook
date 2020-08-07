sudo docker run --name mysql_copy  --restart always --privileged=true -p 4306:3306 -v /etc/mysql/my.cnf:/etc/mysql/my.cnf -v /var/lib/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD="123456" -d mysql


–restart always：开机启动
–privileged=true：提升容器内权限
-v=/mysqltest/config/my.cnf:/etc/my.cnf：映射配置文件
-v=/mysqltest/data:/var/lib/mysql：映射数据目录

