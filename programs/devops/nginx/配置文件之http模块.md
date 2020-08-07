[TOC]

```nginx
http {
    # 文件扩展名与文件类型映射表
    include       mime.types;
    # 默认文件类型
    default_type  application/octet-stream;

    # 日志文件输出格式 这个位置相于全局设置
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # 请求日志保存位置
    # access_log  logs/access.log  main;

    # 打开发送文件
    sendfile        on;
    # tcp_nopush     on;

    # 连接超时时间
    keepalive_timeout  65;

    # 打开gzip压缩
    # gzip  on;

    # 设定请求缓冲
    # client_header_buffer_size 1k;
    # large_client_header_buffers 4 4k;

    # 设定负载均衡的服务器列表
    upstream myproject {
        # weigth参数表示权值，权值越高被分配到的几率越大
        # max_fails 当有#max_fails个请求失败，就表示后端的服务器不可用，默认为1，将其设置为0可以关闭检查
        # fail_timeout 在以后的#fail_timeout时间内nginx不会再把请求发往已检查出标记为不可用的服务器
        server 192.168.122.133:8080 weight=1 max_fails=2 fail_timeout=30s;
        server 192.168.122.134:8080 weight=1 max_fails=2 fail_timeout=30s;
    }

    #配置虚拟主机，基于域名、ip和端口
    server {
        # 监听端口
        listen       80;
        # 监听域名
        server_name  localhost;

        # nginx访问日志放在logs/host.access.log下，并且使用main格式（还可以自定义格式）
        # access_log  logs/host.access.log  main;

        # 返回的相应文件地址
        location / {
            # 设置客户端真实ip地址
            # proxy_set_header X-real-ip $remote_addr;
            # 负载均衡反向代理
            # proxy_pass http://myapp;

            # 返回根路径地址（相对路径:相对于/usr/local/nginx/）
            root   html;
            # 默认访问文件
            index  index.html index.htm;
        }

        # 配置反向代理tomcat服务器：拦截.jsp结尾的请求转向到tomcat
        location ~ \.jsp$ {
            proxy_pass http://192.168.122.133:8080;
        }


        #错误页面及其返回地址
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

    }
}
```
可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。

# http

|      关键字       |                                    解释                                    |
| --------------- | ------------------------------------------------------------------------- |
| include          | 文件扩展名与文件类型映射表                                                     |
| default_type     | 属于HTTP核心模块指令，这里设定默认类型为二进制流。也就是当文件类型未定义时使用这种方式     |
| log_format       | 日志文件输出格式 这个位置相于全局设置                                            |
| access_log       | 请求日志保存位置                                                             |
| sendfile         | 打开发送文件                                                                |
| keepalive_timout | 连接超时时间                                                                |
| gzip             | 打开gzip压缩                                                                |
| unpstream        | 设定负载均衡的服务器列表                                                       |
| weigth           | 表示权重，值越高被分配到的几率越大                                               |
| max_fails        | 当有max_fails个请求失败，就表示后端服务器不可用，默认为1，将其设置为0可以关闭（服务熔断） |
| fail_timeout     | 在以后的fail_timeout时间内nginx不会再把请求发往已检查出标记为不可用的服务器（服务熔断）  |



# server 配置虚拟主机，基于域名、ip和端口
配置虚拟主机的相关参数，一个http中可以有多个server。
## listen 监听端口
```
listen 80;
```
## server_name 监听域名
```
server_name localhost;
```
## error_page 错误页面及其返回地址
```
error_page   500 502 503 504  /50x.html;
location = /50x.html {
    root   html;
}
```
# location 返回的相应文件地址
配置请求的路由，以及各种页面的处理情况。
```
location / {
    proxy_pass http://backend_server;
    proxy_set_header Host $http_host:$proxy_port;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```
## root 返回根路径地址（相对路径:相对于/usr/local/nginx/）
```
root html;
```
## index 默认访问文件
```
index index.html index.htm;
```
## proxy_pass 负载均衡反向代理
```
proxy_pass http://unpstream_name;
```
## proxy_set_header 设置客户端真实ip地址
`proxy_set_header field value;`
+ field：为要更改的项目，也可以理解为变量的名字，比如host
+ value：为变量的值
```
proxy_set_header Host $http_host:$proxy_port;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```
## alise
## access_log 配置访问日志
```
access_log logs/wolfcode.cn.access.log main;
```
