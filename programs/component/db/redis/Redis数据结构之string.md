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
## decr
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
## decrby
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
## incr
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
## incrby
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
## getrange
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
## setrange
`SETRANGE key offset value`
```
redis> SET key1 "Hello World"
"OK"
redis> SETRANGE key1 6 "Redis"
(integer) 11
redis> GET key1
"Hello Redis"
```
## strlen
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
## msetnx
`MSETNX key value [key value ...]`
Time complexity: O(N)
Sets the given keys to their respective values. MSETNX will not perform any operation at all even if just a single key already exists.
Because of this semantic MSETNX can be used in order to set different keys representing different fields of an unique logic object in a way that ensures that either all the fields or none at all are set.
MSETNX is atomic, so all given keys are set at once. It is not possible for clients to see that some of the keys were updated while others are unchanged.
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
## psetex
`PSETEX key milliseconds value`
Time complexity: O(1)
PSETEX works exactly like SETEX with the sole difference that the expire time is specified in milliseconds instead of seconds.
```
redis> PSETEX mykey 1000 "Hello"
"OK"
redis> PTTL mykey
(integer) 1000
redis> GET mykey
"Hello"
```
## setex
`SETEX key seconds value`
Time complexity: O(1)
Set key to hold the string value and set key to timeout after a given number of seconds. This command is equivalent to executing the following commands:
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
## setnx
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
# bit(string)
## bitfield
`BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]`
## bitop
`BITOP operation destkey key [key ...]`
Time complexity: O(N)
Perform a bitwise operation between multiple keys (containing string values) and store the result in the destination key.
## bitpos
`BITPOS key bit [start] [end]`
Time complexity: O(N)
## getbit
`GETBIT key offset`
Time complexity: O(1)
Returns the bit value at offset in the string value stored at key.
When offset is beyond the string length, the string is assumed to be a contiguous space with 0 bits. When key does not exist it is assumed to be an empty string, so offset is always out of range and the value is also assumed to be a contiguous space with 0 bits.
```
redis> SETBIT mykey 7 1
(integer) 0
redis> GETBIT mykey 0
(integer) 0
redis> GETBIT mykey 7
(integer) 1
redis> GETBIT mykey 100
(integer) 0
```
## bitcount 位宽
`BITCOUNT key [start end]`
Time complexity: O(1)
Count the number of set bits (population counting) in a string.
```
redis> SET mykey "foobar"
"OK"
redis> BITCOUNT mykey
(integer) 26
redis> BITCOUNT mykey 0 0
(integer) 4
redis> BITCOUNT mykey 1 1
(integer) 6
redis> SET name 李雷
redis>BITCOUNT name
(integer) 30
```
## setbit