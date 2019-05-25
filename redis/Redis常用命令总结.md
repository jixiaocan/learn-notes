[TOC]

> 本篇文章已保存到 [GitHub](https://github.com/jixiaocan/learn-notes/tree/master/redis) 上，欢迎下载。

## Redis 介绍

> Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs and geospatial indexes with radius queries. Redis has built-in replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster. 

Redis（即 Remote Dictionary Server）是一个由Salvatore Sanfilippo写的key-value存储系统，它是一个性能强劲的、使用内存存储的非关系数据库，它可以存储键和 5 种（字符串、列表、集合、散列表、有序集合）不同类型的值之间的映射，可以将存储在内存的数据持久化到硬盘，可以使用复制特性来扩展读性能，还可以使用客户端分片（client-side sharding）来扩展写性能。

> [这里](http://try.redis.io/)有一个在线教程，帮你快速熟悉 Redis 的常用数据类型和命令。

## Redis 特点
Redis 与其他 key-value 缓存产品有以下三个特点：

* Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
* Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
* Redis支持数据的备份，即master-slave模式的数据备份。

## Redis 优势
* 性能极高 – Redis读的速度是110000次/s,写的速度是81000次/s 。
* 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
* 原子 – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
* 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。

## Redis 连接
登录：`$ redis-cli -h host -p port -a password`
`select index` 切换 redis 数据库
`flushdb` 删除当前数据库所有的 key
`flushall` 删除所有数据
`auth password` 验证密码
`echo message`
`ping` 查看服务是否在运行
`quit` 退出连接

## Redis 服务
`info` 返回redis 服务信息
[Redis 服务器 | 菜鸟教程](http://www.runoob.com/redis/redis-server.html)

## 数据结构
### 字符串（String）

Redis 字符串可以存储字符串、整数、浮点数 3 种类型的值，常用的命令有：

- `set key value [EX seconds] [PX milliseconds] [NX|XX]`
- `get key`
- `mget key [key …]` 获取多个 key 的值
- `getset key value` 设置新值并返回旧值

对于存储整数或者浮点数的键，可以执行自增或增减操作：

- `incr key-name` 将键存储的值加1
- `decr key-name`：将键存储的值减1
- `incrby key-name amount`：将键存储的值加上整数 amount
- `decrby key-name amount`：将键存储的值减去整数 mount
- `incrbyfloat key-name amount`：将键存储的值加上浮点数 amount

如果对一个不存在的键或者一个保存了空字符串的键执行自增或者自减操作，那么 Redis 在执行操作时会将这个键的值当作是 0 来处理。

其他字符串命令：

- `append key-name value`：将 value 追加到 key-name 的值末尾
- `getrange key-name start end`：返回键 key-name 对应的 从 start 到 end 应对的字符串（包括 start 和 end）
- `setrange key-name offset value`：将从 offset 偏移量开始的子串设置为给定值

### 散列（Hashes）

散列可以将多个键值对存储到一个 Redis 键里面，适合存储对象，常用的命令有：

- `hset key field value`
- `hsetnx key field value` 只有在字段不存在时，设置字段的值
- `hget key field`
- `hgetall key` 获取散列包含的所有键值对
- `hdel key field [field …]`
- `hmset key field value [field value …]` 添加多个键值对到散列表
- `hmget key field [field …]` 获取 key 中多个键的值
- `hlen key` 返回 key 包含的键值对数量
- `hexists key field` 检查给定键是否存在与散列中
- `hkeys key` 获取散列包含的所有键
- `hvals key` 获取散列包含的所有值
- `hincrby key field increment` 将 field 存储的值加上整数 increment
- `hincrbyfloat key field increment` 将 field 存储的值加上浮点数 increment

如果散列包含的值非常大，那么可以先使用 hkeys 取出散列包含的所有键，然后再使用 hget 一个个地取出键的值，从而避免使用 hgetall 一次获取而知道的服务阻塞。

### 列表（List）

列表使用的数据结构是双向链表，可以从两端添加或者删除元素，常用的命令有：

- `lpush/rpush key-name value [value …]` 将一个或多个元素添加到列表的左端/右端
- `lpop/rpop key-name` 移除并返回列表最左端/右端的元素
- `lindex key-name offset` 返回列表中偏移量为 offset 的元素
- `lrange key-name start end` 返回列表从 start 到 end（包括）范围内所有元素（start =0，end=-1 时返回所有元素）
- `ltrim key-name start end` 列表进行修剪，只保留从 start 到 end（包括）范围内的元素
- `llen key` 返回列表的长度
- `lset key index value` 通过索引设置列表元素的值

- `lrem key count value` 根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。

最后一条命令中，COUNT 的值可以是以下几种：

- count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。
- count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。
- count = 0 : 移除表中所有与 VALUE 相等的值。

有几个列表命令可以将元素从一个列表移动到另一个列表，或者阻塞（block）执行命令的客户端知道有其他客户端给列表添加元素位置：

- `blpop/brpop key-name [key-name …] timeout` 从第一个非空列表中弹出位于最左端/右端的元素，或者在timeout 秒之内阻塞并等待可弹出的元素出现；
- `rpoplpush source-key dest-key` 从 source-key 列表中弹出右侧的元素，然后放在 dest-key 列表的左侧，并向用户返回这个元素；
- `brpoplpush source-key dest-key timeout` 与 rpoplpush 相似，只是如果 source-key 为空，那么在 timeout 秒之内阻塞并等待可弹出的元素出现

使用上面的命令，就可以实现队列的功能。

### 集合（Set）

集合以无序的方式存储多个不重复的元素，常用的命令有：

- `sadd key member [member ...]` 添加元素，返回添加成功的数量
- `srem key member [member ...]` 移除多个元素，返回移除元素的数量
- `sismember key member`  判断 member 是否是集合 key 的成员，1 包含，0 不包含
- `scard key` 返回集合包含的元素数量
- `smembers key` 返回集合包含的所有元素
- `spop key [count]` 随机移除 count 个元素
- `srandmember key [count]` 随机返回 count 个元素
- `smove source-key dest-key item` 如果集合 source-key 包含元素 item，那么从集合 source-key 里面移除元素 item，并将元素 item 添加到集合 dest-key 中；如果 item 被成功移除，那么命令返回 1，否则返回 0

集合还支持交集、并集、差集运算：

- `sdiff key [key ...]` 返回存在于第一个集合、但不存在于其他集合中的元素（差集）
- `sdiffstore destination key [key ...]` 同 sdiff 命令，并且把差集元素保存到 destination 里面
- `sinter key [key ...]` 返回所有集合的交集
- `sinterstore destination key [key ...]` 同 sinter 命令，并且把交集元素保存到 destination 里面
- `sunion key [key ...]` 返回所有集合的并集
- `sunionstore destination key [key ...]` 同 sunion 命令，并且包并集元素保存到 destination 里面


### 有序集合（Zset）
有序集合通过使用为每个元素指定 score 来实现集合的有序性，常用的命令有：

- `zadd key [NX|XX] [CH] [INCR] score member [score member ...]`
- `zrem key member [member …]` 移除指定的元素，返回被移除元素的个数
- `zremrangebyscore key min max` 移除集合中排名在 min 和 max 之间的成员
- `zcard key` 返回集合包含的元素个数
- `zincrby key increment member` 将 member 成员的分值加上 increment
- `zcount key min max` 返回分值在 min 和 max 之间的成员数量
- `zrank key member` 返回成员 member 在集合中的排名
- `zrevrank key member` 同 zrank 命令一样，并且成员按照分值从大到小排列
- `zscore key member` 返回成员 member 的分值
- `zrange key start stop [WITHSCORES]` 返回集合中排名在 start 和 stop 之间的成员，如果有 WITHSCORES 选项，会将成员的分值一起返回
- `zrevrange key start stop [WITHSCORES]`  同 zrange 命令一样，并且成员按照分值从到大到小排列
- `zrangebyscore key min max [WITHSCORES] [LIMIT offset count]` 获取集合中分值在 min 和 max 之间的成员
- `zrevrangebyscore key max min [WITHSCORES] [LIMIT offset count]` 同 zrangebyscore 命令一样，并且按照分值从大到小排列
- `zinterstore destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]` 有序集合的交集，默认使用 sum 函数（比如说一个集合里面的 a 的值是 1，另外一个集合里面 a 的值是 2，交集之后，a 的值是 3 ）
- `zunionstore destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]` 有序集合的并集



## Key 相关命令

Redis 可以设置键的过期时间，常用的命令有：

- `persist key` 移除键的过期时间
- `ttl key`查看给定键距离过期还有多少秒。key 不存在，返回 -2；没有设超期时间，返回 -1。
- `pttl key` 查看给定键距离过期还有多少毫秒
- `expire key seconds` 让给定键在指定的秒数之后过期
- `pexpire key milliseconds`  让给定键在指定的毫秒数之后过期
- `expireat key timestamp` 让给定键的过期时间设置为 unix 时间戳
- `pexpireat key milliseconds-timestamp` 让给定键的过期时间设置为 unix 毫秒级的时间戳

其他 key 相关的命令有：

- `keys pattern` 查看符合指定模式的 key
- `rename key newkey` 如果 newkey 已经存在，会覆盖它的值
- `renamenx key newkey` 仅当 newkey 不存在时，将 key 改名为 newkey
- `type key` 返回 key 所存储的值的类型：none | string | list | set | zset | hash
- `del key [key ...]` 删除成功返回1，多个累加计数。
- `exists key [key ...]` 存在返回1，多个累加计数。
- `move key db` 把 key 移到另一个数据库，如果它也有该 key，move 失败。
- `sort key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC|DESC] [ALPHA] [STORE destination]`  根据给定的选项，对输入列表、集合或者有序集合进行排序，然后返回或者存储排序的结果

## Redis 发布订阅

- `subscribe channel [channel ...]` 订阅频道
- `unsubscribe [channel [channel ...]]`  退订一个或多个频道，如果该命令没有任何频道，则退订所有的频道
- `publish channel message` 发布消息，返回接收到消息的订阅者的个数
- `psubscribe pattern [pattern …]` 订阅与给定模式相匹配的所有频道
- `punsubscribe [pattern [pattern ...]]` 退订与给定模式相匹配的所有频道

如果客户端在执行订阅操作的过程中断线，那么客户端将丢失在断线期间发送的所有消息。

## Redis 事务
Redis 有 5个命令可以让用户在不被打断的情况下对多个键执行操作，它们分别是 watch、multi、exec、unwatch、discard。

当 Redis 从一个客户端接收到 multi 命令时，Redis 会将这个客户端之前发送的所有命令都放入一个队列里面，知道正常客户端发送 exec 命令为止，然后 Redis 就会在不被打断的情况下，一个接一个地执行存储在队列里面的命令。

一个事务从开始到执行会经历以下三个阶段：

1. 开始事务。
2. 命令入队。
3. 执行事务。

单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。

事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。

> It’s important to note that even when a command fails, all the other commands in the queue are processed – Redis will not stop the processing of commands.
> `multi` 开始事务，`exec` 执行事务，`discard`取消事务：
```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set a aa
QUEUED
127.0.0.1:6379> set b bb
QUEUED
127.0.0.1:6379> set c cc
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) OK
```

## Redis 配置文件 redis.conf
`config get *` 获取所有的配置项
`config get [key]` 获取指定的配置项，比如获取 Redis 数据持久化保存的目录：`CONFIG GET dir`
`config set [key] [value]`

了解每项配置的具体含义：[Redis 配置 | 菜鸟教程](http://www.runoob.com/redis/redis-conf.html)

## Redis 密码设置

打开 redis 配置文件 redis.conf，Mac 下使用 brew 安装 redis，配置文件在 `/usr/local/etc/` 目录下。修改 requiredpass 配置：

```
requirepass xiaocan
```

保存文件，重启 redis，即可生效。
执行 redis-cli 进入 redis 控制台，此时还没有权限操作，输入密码 `auth xiaocan` 即可。
或者直接使用 `redis -a xiaocan` 进行登录。

查看密码

```
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "xiaocan"
```

设置密码。这个是临时密码，重启 redis 后不可用，会读取配置文件的密码。

```
127.0.0.1:6379> CONFIG set requirepass ling
OK
```

完整的 Redis 命令列表可以在 http://redis.io/commands 找到。

> 提示：可以将这个网站添加到 Alfred 中，以后就可以使用 Alfred 查看 Redis 命令的使用方法了。