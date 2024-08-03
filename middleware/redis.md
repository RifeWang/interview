
# Redis

## 概述

Redis 是一个高性能的 key-value 内存数据库，常用于缓存、计数器、排行榜、社交网络、消息队列等多种场景。

QPS 可以达到 10万/秒（这意味着单条指令处理速度 ~10 微妙），具体环境可以使用 redis-benchmark 进行测试。

Redis 支持的最大连接数默认为 10000，取决于 [redis.conf 配置文件中的 maxclients](https://redis.io/docs/latest/operate/oss_and_stack/management/config-file/)。

## 数据类型

基本数据类型及内部编码：
- `string`: 最大 512MB
    - int: 8 个字节的长整型
    - embstr: 小于等于 39 个字节的字符串
    - raw: 大于 39 个字节的字符串
- `hash`:
    - ziplist: 压缩列表，field 个数小于 hash-max-ziplist-entries（默认 512 个）且所有值都小于 hash-max-ziplist-value（默认 64 字节）时使用，内存更加紧凑
    - hashtable: 不满足 ziplist 条件时使用，因为此时 ziplist 的读写效率会下降，会消耗更多内存
- `list`:
    - ziplist: 压缩列表，field 个数小于 list-max-ziplist-entries（默认 512 个）且所有值都小于 list-max-ziplist-value（默认 64 字节）时使用
    - linkedlist: 链表
    - quicklist
- `set`:
    - intset: 整数集合，元素都是整数且元素个数小于 set-max-intset-entries（默认 512 个）时使用
    - hashtable
- `zset`:
    - ziplist: 元素个数小于 zset-max-ziplist-entries（默认 128 个）且所有值都小于 zset-max-ziplist-value（默认 64 字节）时使用
    - skiplist: 跳表

每种数据类型内部有多种编码实现方式，改进内部编码对外部的数据类型和命令没有影响。

高级数据类型：
- `JSON`
- `Streams`：append-only 只可追加的流
- `Geospatial`
- `Bitmaps`
- `Bitfields`
- `Probabilistic`
  - `Hyperloglog`：概率数据结构，用于估计集合的基数
  - `Bloom Filter`：布隆过滤器，插入性能更好，不能删除元素
  - `Cuckoo filter`：检查性能更好，允许删除
  - `t-digest`：估算数据流中的百分位数
  - `Top-K`：估计数据流中排名最靠前的 K 个元素
  - `Count-min sketch`：估计数据流中元素的频率
- `Time series`
- `Pub/Sub`

### 高级数据类型

通过四大模块提供了除了缓存之外的更多特性：
- `RedisJSON`: 操作 JSON 数据
- `RediSearch`: 查询与搜索（包括全文检索、向量搜索、GEO 地理位置等）
- `RedisTimeSeries`: 时序数据
- `RedisBloom`: 概率计算

里程碑版本：
- 3.0（2015 年 4 月）：引入集群模式
- 4.0（2017 年 7 月）：引入了 modules 模块系统
- 6.0（2020 年 5 月）：多线程 I/O

### 多线程 I/O
#### 背景
Redis 一直以来以高性能著称，主要原因有：
- 基于内存的操作
- 单线程模型，避免了多线程上下文切换和锁的开销
- 非阻塞 I/O 多路复用机制

然而，随着网络硬件性能的提升（特别是在 10Gb 网卡成为标准的情况下），Redis 的单线程模型在某些场景下成为了性能瓶颈。特别是在处理大量小型查询时，单个线程难以充分利用现代多核处理器和高速网络硬件的能力。网络 I/O 和连接管理成为主要瓶颈，而不是实际的数据处理。

#### 多线程 I/O 实现
- 主线程: 负责接受客户端连接，读取命令，解析命令，执行命令，生成回复。
- I/O线程: 负责读取客户端请求数据和发送回复数据。

工作流程:
1. 主线程接受连接，创建客户端对象。
1. I/O 线程负责从 socket 读取客户端请求数据。
1. 主线程解析命令，执行命令，生成回复。
1. I/O 线程负责将回复数据写回给客户端。

#### 配置和使用
- `io-threads`：设置I/O线程数，建议设置为CPU核心数-1
- `io-threads-do-reads`：设置为 yes 时，I/O线程会处理读操作。默认情况下 I/O 线程只处理写操作。

### 其它

- `pipeline`:
  - 将多个命令打包到一起发送给 redis 然后逐个执行，最后返回
  - pipeline 的多个命令必须没有相互关联
  - 对集群支持不友好

- 事务：
    - `multi & exec & discard` 弱事务
    - `Lua`

- 持久化，两种方式：
    - `RDB` 内存快照，全量备份，比较耗时
    - `AOF` 日志文件，实时备份，注意刷盘策略

- [分布式锁](https://redis.io/docs/latest/develop/use/patterns/distributed-locks/)
    - 加锁：set 多参数实现（setnx + expire）的原子性
    - 解锁：判断拥有者再解锁
    - 设置超时时间避免死锁：A 获取了锁之后挂掉了，一直无法释放锁，B 永远无法获取锁

- 内存溢出控制策略，maxmemory-policy:
    - noeviction：不会删除数据，拒绝新的写入操作，3.0之后默认的内存淘汰策略
    - volatile-lru：根据 LRU 算法删除设置了 expire 的键
    - allkeys-lru：根据 LRU 算法删除键
    - allkeys-random：随机删除键
    - volatile-random：随机删除过期键
    - volatile-ttl：根据 ttl 删除最近将要过期的数据
    - volatile-lfu：淘汰所有设置了过期时间的键值中最少使用的键值
    - allkeys-lfu：淘汰所有键值中最少使用的键值

- 高可用：哨兵、集群


Redis Cluster：
- `Redis Cluster data sharding` :
    - 没有使用 `consistent hashing` 算法
    - 使用的是 `hash slot` ，每个 key 通过 `CRC16` 算法取模确定 `hash slot`
    - 集群使用 `16384` 个哈希槽，每个节点分配一部分槽
    - 集群节点增加和删除时，将哈希槽进行相应的转移
    - pipeline 等多 key 操作时，keys 必须同属于一个 slot 槽，可以使用 `{}` (hash tags) 强制将 key 分布于同一个 hash 槽中

- `Redis Cluster master-slave model`:
  - 集群的每个节点都可以作为 `master` 节点，拥有 N 个 `slave` 节点，从而保障高可用

- `Redis Cluster consistency guarantees`:
  - 不保证强一致性
  - `replication` 备份是异步进行的


## 数据一致性（数据同步策略）

### Cache Aside Pattern 旁路缓存策略

应用程序负责管理缓存。

查询：
- 从缓存中读取数据；
- 如果缓存命中，则直接返回数据；
- 如果缓存不命中，则从数据库中查询数据；
- 查询到数据后，将数据写入到缓存中，并且返回给用户。

更新/删除：
- 更新数据库中的记录；
- 删除缓存记录。

### 其它

- Read-Through: 读时由 cache 从 database 取数据
- Write-Through: 写时由 cache 写入 database
- Write-Behind: 写时由 cache 异步写入 database

aside & through: aside 是应用程序操作数据同步，through 是缓存系统自身进行操作。

## 缓存常见问题

- `缓存穿透`: 数据库中是空数据，如果缓存也没有，会导致频繁查库。
  - 缓存 null
  - 布隆过滤器，过滤不可能存在的数据

- `缓存击穿`: 热点数据过期，导致大量请求访问数据库
  - 自动刷新，而不是到期自动淘汰
  - 加分布式锁，比如锁住这个 key ，使用 redis 的 setNX 操作，获得锁的可以去查询数据库然后更新缓存，其他的采取重试策略
  - 一般实际很少遇到单个热点数据让数据库崩溃的情况

- `缓存雪崩`: 缓存不可用，或者大量缓存同一时间失效
  - 保证缓存系统的高可用，主从 + sentinel、cluster
  - 过期时间取随机值
  - 多级缓存
  - 限流


## 参考文章

- https://xiaolincoding.com/redis/base/redis_interview.html
- https://javaguide.cn/database/redis/redis-questions-01.html
- https://juejin.cn/post/6844904102850199566