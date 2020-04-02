taskset 查询或设置进程（线程）绑定CPU（亲和性）

# 运行时绑定
```
$ taskset -pc 29797
pid 29797's current affinity list: 0-11
$ taskset -pc 0 29797
pid 29797's current affinity list: 0-11
pid 29797's new affinity list: 0
$ taskset -pc 29797
pid 29797's current affinity list: 0
```

# 启动时绑定
```
$ taskset -c 0 /usr/local/bin/redis-server /etc/redis/redis.conf
```
