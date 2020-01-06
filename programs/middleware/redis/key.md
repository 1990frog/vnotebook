[TOC]
# del 删除key
```
127.0.0.1:6379> del 20589
(integer) 1
127.0.0.1:6379> get 20589
(nil)
127.0.0.1:6379> exists 20589
(integer) 0
```
# dump 序列化key的值
```
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> dump hello
"\x00\x05world\t\x00\xc9#mH\x84/\x11s"
```
# restore 反序列化并赋值，搭配dump使用
`RESTORE key ttl serialized-value`
+ 反序列化给定的序列化值，并将它和给定的 key 关联。
+ 参数 ttl 以毫秒为单位为 key 设置生存时间；如果 ttl 为 0 ，那么不设置生存时间。
+ RESTORE 在执行反序列化之前会先对序列化值的 RDB 版本和数据校验和进行检查，如果 RDB 版本不相同或者数据不完整的话，那么 RESTORE 会拒绝进行反序列化，并返回一个错误。
```
127.0.0.1:6379> restore newhello 0 "\x00\x05world\t\x00\xc9#mH\x84/\x11s"
OK
127.0.0.1:6379> get newhello
"world"
```
# exists 检查给定 key 是否存在
```
redis> SET db "redis"
OK
redis> EXISTS db
(integer) 1
redis> DEL db
(integer) 1
redis> EXISTS db
(integer) 0
```
# expire 给指定 key 设置生存时间
`EXPIRE key seconds`

为给定 key 设置生存时间，当 key 过期时(生存时间为 0 )，它会被自动删除。
在 Redis 中，带有生存时间的 key 被称为volatile。
生存时间可以通过使用 DEL 命令来删除整个 key 来移除，或者被 SET 和 GETSET 命令覆写(overwrite)，这意味着，如果一个命令只是修改(alter)一个带生存时间的 key 的值而不是用一个新的 key 值来代替(replace)它的话，那么生存时间不会被改变。
比如说，对一个 key 执行 INCR 命令，对一个列表进行 LPUSH 命令，或者对一个哈希表执行 HSET 命令，这类操作都不会修改 key 本身的生存时间。
另一方面，如果使用 RENAME 对一个 key 进行改名，那么改名后的 key 的生存时间和改名前一样。
RENAME 命令的另一种可能是，尝试将一个带生存时间的 key 改名成另一个带生存时间的 another_key ，这时旧的 another_key (以及它的生存时间)会被删除，然后旧的 key 会改名为 another_key ，因此，新的 another_key 的生存时间也和原本的 key 一样。
使用 PERSIST 命令可以在不删除 key 的情况下，移除 key 的生存时间，让 key 重新成为一个『持久的』(persistent) key 。

更新生存时间
可以对一个已经带有生存时间的 key 执行 EXPIRE 命令，新指定的生存时间会取代旧的生存时间。
```
redis> SET cache_page "www.google.com"
OK
redis> EXPIRE cache_page 30  # 设置过期时间为 30 秒
(integer) 1
redis> TTL cache_page    # 查看剩余生存时间
(integer) 23
redis> EXPIRE cache_page 30000   # 更新过期时间
(integer) 1
redis> TTL cache_page
(integer) 29996
```
# pexpire 设置毫秒生存时间
`PEXPIRE key milliseconds`
这个命令和 EXPIRE 命令的作用类似，但是它以毫秒为单位设置 key 的生存时间，而不像 EXPIRE 命令那样，以秒为单位
# persist 移除给定 key 的生存时间
# expireat 类似expire，时间参数为unix时间戳
EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置生存时间。
不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。
# pexpireat 为expireat衍生，以毫秒为单位设置 key 的过期 unix 时间戳
# keys 查找所有符合给定模式 pattern 的 key
`KEYS pattern`
时间复杂度O(n)，慎用
KEYS 的速度非常快，但在一个大的数据库中使用它仍然可能造成性能问题，如果你需要从一个数据集中查找特定的 key ，你最好还是用 Redis 的集合结构(set)来代替。
```
特殊符号用 \ 隔开
KEYS * 匹配数据库中所有 key 。
KEYS h?llo 匹配 hello ， hallo 和 hxllo 等。
KEYS h*llo 匹配 hllo 和 heeeeello 等。
KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo 。
```
# migrate 将 key 原子性地从当前实例传送到目标实例的指定数据库上
`MIGRATE host port key destination-db timeout [COPY] [REPLACE]`
# move 将当前数据库的 key 移动到给定的数据库 db 当中
`MOVE key db`
如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 key ，或者 key 不存在于当前数据库，那么 MOVE 没有任何效果。
因此，也可以利用这一特性，将 MOVE 当作锁(locking)原语(primitive)。
# object 允许从内部察看给定 key 的 Redis 对象
`OBJECT subcommand [arguments [arguments]]`
OBJECT 命令允许从内部察看给定 key 的 Redis 对象。
它通常用在除错(debugging)或者了解为了节省空间而对 key 使用特殊编码的情况。
当将Redis用作缓存程序时，你也可以通过 OBJECT 命令中的信息，决定 key 的驱逐策略(eviction policies)。
OBJECT 命令有多个子命令：
+ OBJECT REFCOUNT <key> 返回给定 key 引用所储存的值的次数。此命令主要用于除错。
+ OBJECT ENCODING <key> 返回给定 key 锁储存的值所使用的内部表示(representation)。
+ OBJECT IDLETIME <key> 返回给定 key 自储存以来的空转时间(idle， 没有被读取也没有被写入)，以秒为单位。
```
redis> SET game "COD"           # 设置一个字符串
OK
redis> OBJECT REFCOUNT game     # 只有一个引用
(integer) 1
redis> OBJECT IDLETIME game     # 等待一阵。。。然后查看空转时间
(integer) 90
redis> GET game                 # 提取game， 让它处于活跃(active)状态
"COD"
redis> OBJECT IDLETIME game     # 不再处于空转
(integer) 0
redis> OBJECT ENCODING game     # 字符串的编码方式
"raw"
redis> SET phone 15820123123    # 大的数字也被编码为字符串
OK
redis> OBJECT ENCODING phone
"raw"
redis> SET age 20               # 短数字被编码为 int
OK
redis> OBJECT ENCODING age
"int"
```
# pttl 以毫秒为单位返回 key 的剩余生存时间
`PTTL key`
# ttl 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)
# randomkey 从当前数据库中随机返回(不删除)一个 key
# rename 重命名，不存在覆盖
`RENAME key newkey`
将 key 改名为 newkey 。
当 key 和 newkey 相同，或者 key 不存在时，返回一个错误。
当 newkey 已经存在时， RENAME 命令将覆盖旧值。
# renamenx 重命名，不存在返回错误
`RENAMENX key newkey`
当且仅当 newkey 不存在时，将 key 改名为 newkey 。
当 key 不存在时，返回一个错误。
# scan 基于游标的迭代器
`SCAN cursor [MATCH pattern] [COUNT count]`
[scan命令](./scan命令.md)
# sort 返回key中经过排序的元素
`SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination]`
返回或保存给定列表、集合、有序集合 key 中经过排序的元素。
排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较。
```
# 开销金额列表
redis> LPUSH today_cost 30 1.5 10 8
(integer) 4
# 排序
redis> SORT today_cost
1) "1.5"
2) "8"
3) "10"
4) "30"
# 逆序排序
redis 127.0.0.1:6379> SORT today_cost DESC
1) "30"
2) "10"
3) "8"
4) "1.5"
```
# touch
# type 返回 key 所储存的值的类型
```
none (key不存在)
string (字符串)
list (列表)
set (集合)
zset (有序集)
hash (哈希表)
```
# unlink
# wait