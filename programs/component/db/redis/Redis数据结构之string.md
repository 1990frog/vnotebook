[TOC]

# string
## get 获取
`GET key`
Time complexity: O(1)
Get the value of key. If the key does not exist the special value nil is returned. An error is returned if the value stored at key is not a string, because GET only handles string values.
```
redis> GET nonexisting
(nil)
redis> SET mykey "Hello"
"OK"
redis> GET mykey
"Hello"
redis>
```
## set 赋值
`SET key value [expiration EX seconds|PX milliseconds] [NX|XX]`
Time complexity: O(1)

Set key to hold the string value. If key already holds a value, it is overwritten, regardless of its type. Any previous time to live associated with the key is discarded on successful SET operation.

Options
Starting with Redis 2.6.12 SET supports a set of options that modify its behavior:

+ EX seconds -- Set the specified expire time, in seconds.
+ PX milliseconds -- Set the specified expire time, in milliseconds.
+ NX -- Only set the key if it does not already exist.
+ XX -- Only set the key if it already exist.
```
redis> SET mykey "Hello"
"OK"
redis> GET mykey
"Hello"
redis>
```
## getset 获取旧值，添加新值
`GETSET key value`
Time complexity: O(1)
**Atomically** sets key to value and returns the old value stored at key. Returns an error when key exists but does not hold a string value.
```
redis> SET mykey "Hello"
"OK"
redis> GETSET mykey "World"
"Hello"
redis> GET mykey
"World"
redis>
```
## append 附加
`APPEND key value`
Time complexity: O(1)
If key already exists and is a string, this command appends the value at the end of the string. If key does not exist it is created and set as an empty string, so APPEND will be similar to SET in this special case.
```
redis> EXISTS mykey
(integer) 0
redis> APPEND mykey "Hello"
(integer) 5
redis> APPEND mykey " World"
(integer) 11
redis> GET mykey
"Hello World"
redis>
```
## mget 批量获取（原子操作）
`MGET key [key ...]`
Time complexity: O(N)
Returns the values of all specified keys. For every key that does not hold a string value or does not exist, the special value nil is returned. Because of this, the operation never fails.
n次get=n次网路时间+n次命令时间
1次mget=1次网路时间+n次命令时间
```
redis> SET key1 "Hello"
"OK"
redis> SET key2 "World"
"OK"
redis> MGET key1 key2 nonexisting
1) "Hello"
2) "World"
3) (nil)
```
## mset 批量赋值（原子操作）
`MSET key value [key value ...]`
Time complexity: O(N)
Sets the given keys to their respective values. MSET replaces existing values with new values, just as regular SET. See MSETNX if you don't want to overwrite existing values.
MSET is **atomic**, so all given keys are set at once. It is not possible for clients to see that some of the keys were updated while others are unchanged.
```
redis> MSET key1 "Hello" key2 "World"
"OK"
redis> GET key1
"Hello"
redis> GET key2
"World"
```
## decr 自减1
`DECR key`
Time complexity: O(1)
Decrements the number stored at key by one. If the key does not exist, it is set to 0 before performing the operation. An error is returned if the key contains a value of the wrong type or contains a string that can not be represented as integer. This operation is limited to 64 bit signed integers.
```
redis> SET mykey "10"
"OK"
redis> DECR mykey
(integer) 9
redis> SET mykey "234293482390480948029348230948"
"OK"
redis> DECR mykey
ERR ERR value is not an integer or out of range
redis>
```
## decrby 自减指定值
`DECRBY key decrement`
Time complexity: O(1)
Decrements the number stored at key by decrement. If the key does not exist, it is set to 0 before performing the operation. An error is returned if the key contains a value of the wrong type or contains a string that can not be represented as integer. This operation is limited to 64 bit signed integers.
```
redis> SET mykey "10"
"OK"
redis> DECRBY mykey 3
(integer) 7
redis>
```
## incr 自增1
`INCR key`
Time complexity: O(1)
Increments the number stored at key by one. If the key does not exist, it is set to 0 before performing the operation. An error is returned if the key contains a value of the wrong type or contains a string that can not be represented as integer. This operation is limited to 64 bit signed integers.
**Note**: this is a string operation because Redis does not have a dedicated integer type. The string stored at the key is interpreted as a base-10 **64 bit signed integer** to execute the operation.
Redis stores integers in their integer representation, so for string values that actually hold an integer, there is no overhead for storing the string representation of the integer.
```
redis> SET mykey "10"
"OK"
redis> INCR mykey
(integer) 11
redis> GET mykey
"11"
```
## incrby 自增指定值（可以设置负数）
`INCRBY key increment`
Time complexity: O(1)
```
redis> SET mykey "10"
"OK"
redis> INCRBY mykey 5
(integer) 15
```
## incrbyfloat
`INCRBYFLOAT key increment`
Time complexity: O(1)
```
redis> SET mykey 10.50
"OK"
redis> INCRBYFLOAT mykey 0.1
"10.6"
redis> INCRBYFLOAT mykey -5
"5.6"
redis> SET mykey 5.0e3
"OK"
redis> INCRBYFLOAT mykey 2.0e2
"5200"
```
## getrange 获取指定位置字符串
`GETRANGE key start end`
Time complexity: O(N)
Warning: this command was renamed to **GETRANGE**, it is called **SUBSTR** in Redis versions <= 2.0.
Returns the substring of the string value stored at key, determined by the offsets start and end (both are inclusive). Negative offsets can be used in order to provide an offset starting from the end of the string. So -1 means the last character, -2 the penultimate and so forth.
The function handles out of range requests by limiting the resulting range to the actual length of the string.
```
redis> SET mykey "This is a string"
"OK"
redis> GETRANGE mykey 0 3
"This"
redis> GETRANGE mykey -3 -1
"ing"
redis> GETRANGE mykey 0 -1
"This is a string"
redis> GETRANGE mykey 10 100
"string"
redis>
```
## setrange 修改指定位置字符串
`SETRANGE key offset value`
```
redis> SET key1 "Hello World"
"OK"
redis> SETRANGE key1 6 "Redis"
(integer) 11
redis> GET key1
"Hello Redis"
```
## strlen 获取string长度
`STRLEN key`
Time complexity: O(1)
Returns the length of the string value stored at key. An error is returned when key holds a non-string value.
```
redis> SET mykey "Hello world"
"OK"
redis> STRLEN mykey
(integer) 11
redis> STRLEN nonexisting
(integer) 0
```
## msetnx 原子设定多个值，如果存在回滚
`MSETNX key value [key value ...]`
Time complexity: O(N)
同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
即使只有一个给定 key 已存在， MSETNX 也会拒绝执行所有给定 key 的设置操作。
MSETNX 是原子性的，因此它可以用作设置多个不同 key 表示不同字段(field)的唯一性逻辑对象(unique logic object)，所有字段要么全被设置，要么全不被设置。
```
redis> MSETNX key1 "Hello" key2 "there"
(integer) 1
redis> MSETNX key2 "there" key3 "world"
(integer) 0
redis> MGET key1 key2 key3
1) "Hello"
2) "there"
3) (nil)
```
## psetex 赋值并设置毫秒级生存时间
`PSETEX key milliseconds value`
Time complexity: O(1)
这个命令和`SETEX`命令相似，但它以毫秒为单位设置`key`的生存时间，而不是像`SETEX`命令那样，以秒为单位。
```
redis> PSETEX mykey 1000 "Hello"
"OK"
redis> PTTL mykey
(integer) 1000
redis> GET mykey
"Hello"
```
## setex 赋值并设置秒级生存时间
`SETEX key seconds value`
Time complexity: O(1)
+ SET mykey value
+ EXPIRE mykey seconds
```
redis> SETEX mykey 10 "Hello"
"OK"
redis> TTL mykey
(integer) 10
redis> GET mykey
"Hello"
```
## setnx 仅当key不存在时赋值
将`key`的值设为`value`，当且仅当`key`不存在。
若给定的`key`已经存在，则`SETNX`不做任何动作。
`SETNX key value`
Time complexity: O(1)
Set key to hold string value if key does not exist. In that case, it is equal to SET. When key already holds a value, no operation is performed. SETNX is short for "SET if Not eXists".

Integer reply, specifically:

+ 1 if the key was set
+ 0 if the key was not set
```
redis> SETNX mykey "Hello"
(integer) 1
redis> SETNX mykey "World"
(integer) 0
redis> GET mykey
"Hello"
```
