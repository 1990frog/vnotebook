[TOC]

# 概览
http1.0协议默认是不支持keepalive的，使用http1.1自动设置keepalive，并将header头文件置空

# 配置
```
upstream servers{
    server ip:port weight=1;
    keepalive 30;
}

location / {
    proxy_http_version 1.1;
    proxy_set_header Connection "";
}
```

# 查看linux端口监控
```
$ netstat -an | grep ip | grep ESTABLISHED
$ netstat -an | grep ip | grep TIME_WAIT
```