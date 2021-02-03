# Redis入门笔记

## 介绍

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://www.redis.cn/commands/geoadd.html) 索引半径查询。Redis 内置了 [复制（replication）](http://www.redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://www.redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://www.redis.cn/topics/lru-cache.html)，[事务（transactions）](http://www.redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://www.redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://www.redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://www.redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。

## 数据类型

- string

- hash：哈希/散列

  hgetall、hget、hset、hmset、hmget、hlen、hdel、hexists、hkeys、hvals、hscan、hsetnx、hstrlen、hincrby、hincrbyfloat

- list

- sets

- sort sets

详细[命令](http://www.redis.cn/commands.html)

## 事务

Redis  [事务](http://www.redis.cn/topics/transactions.html)可以一次执行多个命令，且

- 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断
- 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。

相关命令：``MULTI``开启事务、``EXEC``执行、``DISCARD``中断、``WATCH``监控

- EXEC

### 用法

```shell
127.0.0.1:6379> SET foo 1
OK
127.0.0.1:6379> MULTI #开启事务
OK
127.0.0.1:6379> INCR foo
QUEUED
127.0.0.1:6379> INCRBY foo 2
QUEUED
127.0.0.1:6379> EXEC #执行事务
1) (integer) 2
2) (integer) 4
127.0.0.1:6379> GET foo
"4"
```

### 错误

使用事务时可能会遇上以下两种错误：

- 事务在执行EXEC入队之前，入队的命令可能会出错。如：语法参数错误
- EXEC调用之后失败。如：命令可能处理了错误类型的键

对于发生在EXEC执行之前的错误，客户端以前的做法是检查命令入队所得的返回值：如果命令入队时返回 `QUEUED` ，那么入队成功；否则，就是入队失败。如果有命令在入队时失败，那么大部分客户端都会停止并取消这个事务。

从 Redis 2.6.5 开始，服务器会对命令入队失败的情况进行记录，并在客户端调用EXEC命令时，拒绝执行并自动放弃这个事务。

至于那些在EXEC命令执行之后所产生的错误， 并没有对它们进行特别处理： 即使事务中有某个/某些命令在执行时产生了错误， 事务中的其他命令仍然会继续执行。

### 放弃事务

执行``DISCARD``命令时， 事务会被放弃， 事务队列会被清空， 并且客户端会从事务状态中退出

```shell
127.0.0.1:6379> SET foo 1
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> INCR foo
QUEUED
127.0.0.1:6379> INCR foo
QUEUED
127.0.0.1:6379> DISCARD
OK
127.0.0.1:6379> GET foo
"1"
```

### Redis 不支持回滚（roll back）

Redis 在事务失败时不进行回滚，而是继续执行余下的命令。

以下是这种做法的优点：

- Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：即失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应出现在生产环境中。
- 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

### WATCH

WATCH命令可以为 Redis 事务提供 check-and-set （CAS）行为实现**乐观锁**。

**被 WATCH 的键会被监视，并会发觉这些键是否被改动过了。 如果有至少一个被监视的键在 EXEC执行之前被修改了， 那么整个事务都会被取消， EXEC返回nil-reply来表示事务已经失败**。

WATCH 使得 EXEC 命令需要有条件地执行： 事务只能在所有被监视键都没有被修改的前提下执行， 如果这个前提不能满足的话，事务就不会被执行。WATCH可被调用多次，对key监视从WATCH被调用开始至执行EXEC为止。

## 持久化

redis提供了RDB及AOF两种方式的持久化。

### RDB

能够在指定的时间间隔能对你的数据进行快照存储,简单来说就是定时备份。保存RDB的方式：

- save：保存是阻塞主进程，客户端无法连接redis，等SAVE完成后，主进程才开始工作，客户端才可以连接，因此生产环境中很少使用save，可使用bgsave。
- bgsave：后台保存DB，会立即返回 OK 状态码。 Redis fork一个save子进程, 父进程继续提供服务以供客户端调用，子进程将DB数据保存到磁盘然后退出。因此在执行过程中，不影响主进程，客户端可以正常链接。

#### RDB优点

- 紧凑型文件，保存了过去某个时间点的数据集，非常适用于备份，如每小时保存下过去24小时数据，每天保存过去30天数据，这样可以根据需求恢复到不同版本。
- 一个紧凑的单一文件，方便传输至其它远端数据中心用户数据恢复
- 恢复大数据时候，RDB相对快一些。
- 异步保存（bgsave）时候父进程唯一需要做的就是fork出一个子进程，接下来的工作全部交给子进程，父进程无需进行IO操作，故RDB持久化可以最大化redis性能。

#### RDB缺点

- 意外宕机时候可能导致部分数据丢失
- RDB 需要经常fork子进程来保存数据集到硬盘上,当数据集比较大的时候,fork的过程是非常耗时的,可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续1秒,AOF也需要fork,但是你可以调节重写日志文件的频率来提高数据集的耐久度

### AOF

AOF:记录每次对服务器写的操作至日志中，服务器重启时候重新执行该命令以恢复数据，redis还能对aof文件进行后台重写，使其AOF体积不至于太大。

#### AOF优点

- 使用AOF 会让你的Redis更加耐久:使用不同的fsync策略：无fsync,每秒fsync,每次写的时候fsync.使用默认的每秒fsync策略,一旦出现故障最多丢失1秒数据。

- AOF文件是一个只进行追加的日志文件，即使由于某些原因(磁盘空间已满，写的过程中宕机等等)未执行完整的写入命令,你也也可使用redis-check-aof工具修复这些问题。

- Redis在AOF体积过大时候，自动地在后台对 AOF 进行重写。

  **整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作**。

- AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 

  **如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态**。

#### AOF缺点

- AOF文件体积大于RDB。

- 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。

  在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。



## 消息订阅Pub/Sub

订阅，取消订阅和发布实现了发布/订阅消息，发送者是将发布的消息分到不同的频道。订阅者对一个或多个频道感兴趣，只需接收感兴趣的消息，不需要知道什么样的发布者发布的。这种发布者和订阅者的解耦合可以带来更大的扩展性和更加动态的网络拓扑。

## 监控工具 Sentinel

## Redis与Memcache对比

## 中文手册地址

**[Redis中文社区](http://www.redis.cn/)**

