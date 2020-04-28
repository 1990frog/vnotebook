Redis处理指令的时候是单线程的，可以为Redis的进程与指定CPU核绑定，这样就避免了CPU时间片切换带来的损耗。
每个服务器至少有一个CPU，每个CPU最少有多个核。所以为了更充分的利用CPU，启动对应总核数的Redis实例并都绑定一个核上即可。

```
taskset -pc 0 redis-server redis.conf
```