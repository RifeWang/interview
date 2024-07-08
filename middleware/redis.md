
# Redis

## 概述

Redis 是一个高性能的 key-value 内存数据库，常用于缓存、计数器、排行榜、社交网络、消息队列等多种场景。

QPS 可以达到 10万/秒，具体环境可以使用 redis-benchmark 进行测试。

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
- `BitMap`
- `Hyperloglog`：概率数据结构，用于估计集合的基数
- `Bloom Filter`：布隆过滤器
- `Geo`
- `Pub/Sub`
- `Stream`：append-only 只可追加的流

其它：
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

查询：
- 从缓存中读取数据；
- 如果缓存命中，则直接返回数据；
- 如果缓存不命中，则从数据库中查询数据；
- 查询到数据后，将数据写入到缓存中，并且返回给用户。

更新：
- 更新数据库中的记录；
- 删除缓存记录。


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