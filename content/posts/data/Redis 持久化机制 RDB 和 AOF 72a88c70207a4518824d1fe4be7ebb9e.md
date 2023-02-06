---
title: "Redis 持久化机制: RDB 和 AOF"
date: 2023-02-01T18:02:04+11:00
draft: false
categories: ["Redis"]
---

# Redis 持久化机制: RDB 和 AOF

# Redis 持久化

### 为什么需要持久化?

Redis 是基于内存的数据库, 服务一旦宕机, 内存中的数据将全部丢失. 通常来说可以通过数据库来恢复这些数据, 但这会给数据库带来非常大的读压力, 并且这个过程会非常缓慢, 并导致程序响应慢, 因此 Redis 提供了把内存数据持久化到硬盘, 并通过备份文件来恢复数据的功能, 即持久化机制.

### 持久化的方式

目前 [Redis Documentation](https://redis.io/docs/management/persistence/) 上对持久化的支持有以下几种方案:

1. RDB (Redis Database): 将某个时间点上的数据生成快照 (snapshot) 并保存到硬盘上
2. AOF (Append Only File): 将每个接收到的写操作记录到硬盘上, 这些操作可以在 Redis 重启时被重放, 并用于重新构建 Redis 数据库
3. RDB + AOF: AOF 和 RDB 的混合模式

# RDB

RDB 指对整个数据集在特定时间点生成快照 (point-to-time snapshot), 可用于Redis的数据备份, 转移和恢复. 它是 Redis 默认使用的持久化方案.

## 工作原理

RDB 利用操作系统提供的[写时复制](https://en.wikipedia.org/wiki/Copy-on-write) (Copy-on-Write) 机制来进行持久化, 即当主进程 P fork 出子进程时 Q 时, Q 和 P 共享同一块内存空间, 当 P 准备对某块内存进行写操作时, P 会将这块内存页进行复制, 并在新的副本上对数据进行修改, 而 Q 仍然读取原先的内存页. 这样既能够保证 Redis 实例继续服务外部流量, 又能够以最小的成本完成数据的持久化. 但正因如此, 持久化过程中的写操作是不会被记录的.

![./Untitled.png](/Untitled.png)

![./Untitled1.png](/Untitled1.png)


## 触发方式

触发rdb持久化的方式有2种:

1. 手动触发. 包含两个命令:
    1. `save`: 阻塞 Redis 进程, 并进行 RDB 持久化, 直到其完成为止, 对于内存占用大的实例会造成长时间阻塞.
    2. `bgsave`: background save, 让 Redis 进程通过 fork 操作创建子进程, 并在子进程进行 RDB 持久化, 只在 fork 阶段阻塞 Redis 进程
2. 自动触发, 通过配置中的 save 命令实现. Redis 服务有一个周期性维护函数 `serverCron`, 默认每 100 ms 执行一次, 它的其中一项功能就是检查所有 save 命令的条件里是否有任意一条被满足. 如果不想使用自动触发, 把所有的 save 命令注释即可.

```
save x y # 在 x 秒内如果至少有 y 个 key 值发生变化, 则触发RDB
save 60 900 # 在 60 秒内如果至少有 900 个 key 值发生变化, 则触发RDB

```

## 总结

### 是否应该以尽可能高的频率来触发 RDB?

为了保证宕机时丢失的数据尽量少, 我们也许可以每分钟出发一次 RDB 进行数据备份. 虽然 bgsave 在子进程中执行, 不会阻塞主线程, 但仍然有一些问题：

1. bgsave 需要通过 fork 操作来创建子进程, fork 操作本身是会阻塞主进程的, 并且主线程占用内存越多, fork 操作的阻塞时间越长
2. 将全量数据写入硬盘的操作会占用大量的带宽, 给硬盘带来很大的压力, 从而影响 Redis 实例的性能, 并且如果上一次的写入操作尚未完成, 就开始了下一次的写入操作, 更有可能会造成恶性循环

从这两点出发可以认为触发 RDB 的频率并不是越高越好, 我们需要考虑 Redis 实例占用内存的大小以及全量数据写入硬盘的速度.

### 优点

- RDB文件是某个时间节点的快照, 默认使用 LZF 算法进行压缩, 压缩后的文件体积远远小于内存大小, 适用于定期执行（例如每一小时进行一次）, 并将 RDB 文件上传到数据中心进行容灾备份
- 与AOF相比, 使用 RDB 恢复大型数据集更快

### 缺点

- RDB 方式实时性不够, 无法做到秒级的持久化；
- RDB 需要 fork 子进程, 而 fork 进程执行成本非常高；
- RDB 文件是二进制编码的, 没有可读性

# AOF

AOF (Append Only File) 通过写日志的方式, 在 Redis 每次写操作完成后在日志里记录下此次执行的命令, 当服务器重启的时候通过顺序地重放这些日志来恢复数据.

**配置**

AOF 功能默认是关闭的, 需要通过修改 redis.conf 并重启 Redis 来开启.

```
# no by default
appendonly yes
appendfilename appendonly.aof

```

**写后日志**

和 MySQL 的写前日志 (Write-Ahead Logging) 不同, AOF 会在写操作完成后记录日志, 这样既能够保证 Redis 不阻塞并及时响应写操作, 还可以避免运行时检查出写操作命令不合法再回滚这条日志. 但如果在命令执行完之后, 写日志完成之前, 服务器发生了宕机, 也有可能会丢失数据.

**工作流程**

AOF的工作原理可以概括为几个步骤：命令追加（append）、文件写入与同步（fsync）、文件重写（rewrite）、重启加载（load）.

## 1 追加命令 append

当 AOF 持久化功能开启时, Redis 执行完一个写命令后, 会按照 **[RESP (Redis Serialization Protocol)](https://redis.io/docs/reference/protocol-spec/)** 协议规定的格式把这条写命令**追加**到其维护的 **AOF 缓冲区**末尾.

**AOF缓冲区** (aof_buf) 采用 Redis 特有的数据结构 [SDS (Simple Dynamic String)](https://github.com/antirez/sds), 根据命令的类型, 使用不同的方法（`catAppendOnlyGenericCommand`, `catAppendOnlyExpireAtCommand`等）, 来对命令进行处理, 最后写入缓冲区.

如果命令追加时正在进行 AOF **重写**, 这些命令还会追加到**重写缓冲区** `aof_rewrite_buffer`.

## 2 写入文件以及同步 fsync

由于硬盘的 I/O 性能较差, 文件读写速度远远比不上 CPU 的处理速度, 那么如果每次文件写入都等待数据写入硬盘, 会整体拉低操作系统的性能. 为了解决这个问题, 操作系统提供了**延迟写（delayed write）**机制来提高硬盘的I/O性能.

Redis 每次事件轮询结束前（`beforeSleep`）都会调用函数 `flushAppendOnlyFile`, 它会把 **AOF 缓冲区**中的数据写入**内核缓冲区**, 并且根据 **appendfsync** 的配置来决定采用何种策略把**内核缓冲区**中的数据写入**磁盘**, 即调用 `fsync()` , 有三个可选项：

- always：每次都调用`fsync()`, 安全性最高, 但性能最差
- no：不会调用`fsync()`. 性能最好, 安全性最差.
- everysec：仅在满足同步条件时调用`fsync()`. 这是官方推荐的策略, 也是默认配置, 能够兼顾性能和数据安全性, 只有在系统突然宕机的情况下会丢失 1 秒的数据.

## 3 重写 rewrite

随着时间的增加, AOF 文件体积会越来越大, 导致磁盘占用空间更多, 数据恢复时间更长. 为了解决这个问题, Redis 引入了 **AOF 重写 (AOF Rewrite)** 机制, 通过创建新的 AOF 文件, 将旧文件中的多条命令整合成为新文件中的单条命令, 并替换旧文件, 来减少 AOF 文件的体积.

### **重写在何时发生?**

和 RDB 的触发方式类似, AOF重写可以通过手动或自动触发.

1. 手动触发：调用`bgrewriteaof`命令, 如果当前不存在正在执行的 bgsave 或 bgrewriteaof 子进程, 那么重写会立即执行, 否则会等待子进程操作结束后再执行.
2. 自动触发由两个配置项控制, 只有这两个指标同时满足的时候才会发生重写：
    - `auto-aof-rewrite-percentage`: 当前AOF文件（aof_current_size）和上一次重写发生后AOF文件大小（aof_base_size）相比, 其增加的比例, 默认为100, 即当 aof_current_size == 2 * aof_base_size 时触发
    - `auto-aof-rewrite-min-size`: 运行`BGREWRITEAOF`时AOF文件占用空间最小值, 默认为64MB

### **重写的流程是怎么样的?**

1. bgrewriteaof 触发重写, 判断是否存在 bgsave 或者 bgrewriteaof 正在执行, 如果存在则等待其执行结束再执行
2. 主进程fork子进程, 防止主进程阻塞无法提供服务
3. 子进程遍历 Redis 内存快照中数据写入临时 AOF 文件, 同时会将新的写指令写入 aof_buf 和 aof_rewrite_buf 两个重写缓冲区, 前者是为了写回旧的 AOF 文件, 后者是为了后续刷新到临时 AOF 文件中, 防止快照内存遍历时新的写入操作丢失
4. 子进程结束临时AOF文件写入后, 通知主进程
5. 主进程会将 aof_rewirte_buf 中的数据写到子进程生成的临时 AOF log 中
6. 主进程使用临时AOF文件替换旧AOF文件, 完成整个重写过程

整个过程可以参考下图：

Redis启动时把`aof_base_size`初始化为当时aof文件的大小, Redis运行过程中, 当AOF文件重写操作完成时, 会对其进行更新；`aof_current_size`为`serverCron`执行时AOF文件的实时大小. 当满足以下两个条件时, AOF文件重写就会触发：

### **AOF重写会阻塞吗?**

AOF 的重写过程是由后台进程 bgrewriteaof 来完成的. 主线程 fork 出后台的 bgrewriteaof 子进程, fork 操作会把主线程的内存拷贝一份给 bgrewriteaof 子进程, 这里面就包含了数据库的最新数据. 然后, bgrewriteaof 子进程逐一把拷贝的数据写成操作, 并记入重写日志, 因此在重写过程中, 只有当 fork 操作发生时会阻塞主线程.

## 4 重启并加载 load

Redis启动后通过`loadDataFromDisk`函数执行数据加载, 流程大致如下：

1. 未开启 AOF 的情况下, 只使用 RDB 文件加载数据
2. 开启 AOF 的情况下, 如果 AOF 文件使用 RDB 头, 那么先使用 RDB, 再使用 AOF , 否则只使用 AOF 加载数据

## 总结

### **AOF能保证数据完整性么?**

如果在对AOF文件进行写操作时发生了宕机, 或磁盘满了, 由于延迟写的特点, AOF的RESP命令可能会因为被截断而不完整. 发生这种情况时, Redis会按照配置项`aof-load-truncated` 的值来进行不同的操作：

- yes：尽可能多的加载数据, 并以日志的方式通知用户；
- no：以系统错误的方式产生崩溃, 并禁止重启, 需要用户手动修复文件

### 优点

- AOF持久化有更好的实时性, 因为使用 every second 作为 fsyn从的默认策略, 极端情况下可能只会丢失一秒的数据
- 对 AOF log 的操作只有 append, 不会导致文件损坏；即使最后写入数据被截断, 也很容易使用`redis-check-aof`工具修复
- 重写机制可以保证 AOF log 不占用太大空间, 并且重写过程中新的写操作也会记录到旧的 log 中, 防止数据丢失
- AOF log 具有更高的可读性, 并且可以轻易导出

### 缺点

- 对于相同的数据集, AOF 文件通常会比 RDB 文件大
- 在写操作较多时, AOF 的延迟会更高

# Reference

[https://redis.io/docs/management/persistence/](https://redis.io/docs/management/persistence/)