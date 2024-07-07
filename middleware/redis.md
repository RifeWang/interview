
# 分布式缓存

## Cache Aside Pattern 旁路缓存策略

查询：
- 从缓存中读取数据；
- 如果缓存命中，则直接返回数据；
- 如果缓存不命中，则从数据库中查询数据；
- 查询到数据后，将数据写入到缓存中，并且返回给用户。

更新：
- 更新数据库中的记录；
- 删除缓存记录。


## 常见问题

- `缓存穿透`: 数据库中是空数据，如果缓存也没有，会导致频繁查库。
  - 缓存 null
  - 布隆过滤器，过滤不可能存在的数据，( url 去重之类的，存储所有的ID，布隆的结果是 ID 没有就肯定没有，有但实际不一定有)

- `缓存击穿`: 热点数据过期，导致大量请求访问数据库
  - 自动刷新，而不是到期自动淘汰
  - 加分布式锁，比如锁住这个 key ，使用 redis 的 setNX 操作，获得锁的可以去查询数据库然后更新缓存，其他的采取重试策略
  - 一般实际很少遇到单个热点数据让数据库崩溃的情况

- `缓存雪崩`: 缓存不可用，或者大量缓存同一时间失效
  - 保证缓存系统的高可用
  - 过期时间取随机值
  - 多级缓存
  - 如果是集群的话，概率较低


## Memcached / Redis

- `Memcached`
  - 数据结构单一，简单 key-value
  - 单个 value 最大 1 MB
  - 不支持持久化
  - 不支持集群

- `Redis`
  - 基本数据类型：string | list | set | zset（ sorted set ）| hash
  - 高级数据类型：bitMap | Hyperloglog | 布隆过滤器 | GeoHash | Pub/Sub | Stream
  - 每秒 10 万次读写操作
  - 单个 value 最大 512 MB
  - 支持持久化，两种方式：RDB 内存快照、AOF 日志文件
  - 支持集群
  - `LRU`: 挑选最近最少使用的数据进行淘汰

  - 不常用的数据结构
    - `Bit arrays`: bit 数组
    - `HyperLogLogs`: 概率数据结构，用于估计集合的基数
    - `Streams`: append-only 只可追加的流

- `Redis` 实现分布式锁：
  - 加锁：set 多参数实现（setnx + expire）的原子性
  - 解锁：判断拥有者再解锁
  - 设置超时时间避免死锁：A 获取了锁之后挂掉了，一直无法释放锁，B 永远无法获取锁

- `Redis pipeline`:
  - 将多个命令打包到一起发送给 redis 然后逐个执行，最后返回
  - pipeline 的多个命令必须没有相互关联
  - 对集群支持不友好

- `Redis multi & exec & discard` 事务:
  - 弱事务


- `Redis Cluster data sharding` :
    - 没有使用 `consistent hashing` 算法
    - 使用的是 `hash slot` ，每个 key 都分布于 `hash slot` 中
    - 内置 `16384` 个哈希槽，`CRC16` 算法取模 `16384`
    - 集群节点增加和删除时，将哈希槽进行相应的转移
    - pipeline 等多 key 操作时，keys 必须同属于一个 slot 槽，可以使用 `{}` (hash tags) 强制将 key 分布于同一个 hash 槽中

- `Redis Cluster master-slave model`:
  - 集群的每个节点都可以作为 `master` 节点，拥有 N 个 `slave` 节点，从而保障高可用

- `Redis Cluster consistency guarantees`:
  - 不保证强一致性
  - `replication` 备份是异步进行的


- `LRU ( Least Recently Used )` : 最近最少使用

