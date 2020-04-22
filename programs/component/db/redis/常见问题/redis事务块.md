[TOC]

# mutil...exec
事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由`EXEC`命令原子性(atomic)地执行
```
$ multi
$ ...
$ exec
```

# watch
可以用watch监控一个key，后面执行mutil...exec事务块，如果watch监控的值发生了变化，那么事务回滚（CAS的思想）