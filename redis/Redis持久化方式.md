### 持久化配置

Redis 提供了两种不同的持久化方法来将数据存储到硬盘里面。这两种方法可以单独使用，也可以同时使用。

一种方法叫快照，它可以将某一时刻 Redis 的所有数据写入硬盘。它的相关配置有：

```properties
# 多久执行一次自动快照操作
save 60 1000
# 在创建快照失败后是否仍然继续执行写命令
stop-writes-on-bgsave-error no
# 是否对快照文件进行压缩
rdbcompression yes
# 快照文件的名称
dbfilename dump.rdb
```

另一种方法叫只追加文件（append-only file，AOF），它会在执行写命令时，将命令复制到硬盘里面。它的相关配置有：

```properties
# 是否使用 AOF 持久化
appendonly yes
# 多久才将写入的内容同步到硬盘 always | everysec | no
appendfsync everysec
# 在对 AOF 进行压缩的时候能否执行同步操作
no-appendfsync-on-rewrite no
# 多久执行一次 AOF 压缩
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

两种方式共用的配置是：

```properties
# 快照文件和 AOF 文件保存的位置
dir ./
```

可以执行命令 `config get dir` 获取配置的目录。

在 Redis 重启后，可以通过在硬盘中保存的文件恢复数据。

### 快照持久化

创建快照的方法有以下几种：

- 客户端可以通过向 Redis 发送 `BGSAVE` 命令来创建一个快照。Redis 会调用 fork 来创建一个子进程，然后子进程负责将快照写入硬盘，而父进程继续处理命令请求。
- 客户端可以通过向 Redis 发送 `SAVE` 命令来创建一个快照。Redis 服务器在快照创建完毕之前将不再响应任何其他命令。
- 根据配置文件的配置，比如 `save 60 1000`，那么从 Redis 最近一次创建快照之后开始算起，当 “60 秒之内有 1000 次写入"这个条件被满足时，Redis 就会自动触发 `BGSAVE` 命令。可以配置多个 `save` 选项。
- 当 Redis 通过 SHUTDOWN 命令接受到关闭服务器的请求时，或者接收到标准 TERM 信号时，会执行一个 SAVE 命令，在 SAVE 命令执行完毕之后关闭服务器。
- 当一个 Redis 服务器连接另一个 Redis 服务器，并向对方发送 SYNC 命令来开始一次复制操作时，如果主服务器目前没有在执行 BGSAVE 操作，或者主服务器并非刚刚执行完 BGSAVE 操作，那么主服务器就会执行 BGSAVE 命令。

使用这种方式，**Redis 会丢失最近一次创建快照之后写入的所有数据**。

而且随着 Redis 占用的内存越来越多，`BGSAVE` 在创建子进程时耗费的时间也会越来越多。对于真实的硬件、VMWare虚拟机或者 KVM 虚拟机来说，Redis 进程每占用一个 GB 的内存，创建该进程的子进程所需的时间就要增加 10～20 毫秒。

在数据量非常大时，为了防止 Redis 因为创建子进程而出现停顿，可以考虑关闭自动保存，通过手动发送 `BGSAVE` 或 `SAVE` 来进行持久化，来控制停顿出现的时间。另外，使用 `SAVE` 虽然会阻塞 Redis 知道快照生成完毕，但是因为它需要创建子进程，所以不会像 `BGSAVE` 一样因为创建子进程而导致 Redis 停顿；并且因为没有子进程在争抢资源，所以 `SAVE` 创建快照的速度比较快一点。

### AOF 持久化

AOF 持久化会将被执行的写命令写到 AOF 文件的末尾，以此来记录数据的变化。恢复数据时，Redis 只要从头到尾重新执行一次 AOF 文件所包含的所有写命令即可。

写 AOF 文件的频率有 always 、everysec、no 三个选项：

- always：每个 redis 写命令都要同步写入硬盘，这样会严重降低 redis 的速度；
- everysec：每秒执行一次同步，显式地将多个写命令同步到硬盘；
- no：让操作系统来决定应该何时进行同步；

一般情况下，使用 `everysec` 配置。即使系统崩溃，也只是丢失一秒之内产生的数据。

随着 Redis 不断运行，AOF 文件的体积也会不断增长。而且如果 AOF 文件过大，Redis 在重启后，使用它还原数据的操作也非常耗时。

为了解决 AOF 文件体积不断增大的问题，用户可以向 Redis 发送 `BGREWRITEAOF` 命令，这个命令会通过移除 AOF 文件中的冗余命令来重写 AOF 文件，使 AOF 文件的体积变得尽可能地小。

`BGREWRITEAOF` 的工作原理：Redis 创建一个子进程，然后由子进程负责对 AOF 文件进行重写。

控制 `BGREWRITEAOF` 的配置：

```properties
# 可以大于 100
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

上面的配置表示当 AOF 文件的体积大于 64MB，并且文件的体积比上一次重写之后的体积大了至少一倍（100%）的时候，Redis 将执行 `BGREWRITEAOF` 命令。

> 本篇文章已保存到 [GitHub](https://github.com/jixiaocan/learn-notes/tree/master/redis) 上，欢迎下载。













