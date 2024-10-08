# 架构基础

## 软件工程
### 信息系统生命周期
- 产生：一是概念的产生过程，根据企业需要，提出建设信息系统的初步想法；二是需求分析过程，对需求进行深入调研和分析，形成需求分析报告。
- 开发：分为 5 个阶段
    - 总体规划：可行性分析、项目开发计划
    - 系统分析：需求分析，为系统设计提供逻辑模型。
    - 系统设计：概要设计、详细设计
    - 系统实施：编码、测试
    - 系统验收
- 运行
- 消亡

### 软件生命周期模型
- 瀑布模型（`Waterfall Model`）：严格串行化的过程，难以应对改变。
    1. 需求分析
    2. 系统设计
    3. 程序设计
    4. 编码实现
    5. 单元测试
    6. 集成测试
    7. 系统测试
    8. 运行维护

- 原型化模型（`Prototype Model`）：又称快速原型。对于大型软件，原型可能非常复杂而难以快速形成。
    - 抛弃型原型：仅仅作为帮助定义需求的一种手段。
    - 演化性原型

- 螺旋模型（`Spiral Model`）：在快速原型的基础上扩展而成，也有人把螺旋模型归到快速原型。此模型支持大型软件开发。每个阶段都进行以下四步迭代：
    1. 目标设定
    2. 风险分析：对可选方案进行风险识别和详细分析，制定解决方法，采取有效措施避免这些风险。
    3. 开发和有效性验证
    4. 评审

- 敏捷模型（`Agile Model`）：
    - 核心思想：
        - 适应性而非预设性
        - 面向人而非面向过程
        - 迭代增量式的开发过程
    - 敏捷方法有以下几种：
        - 极限编程（`Extreme Programming`）
        - 水晶系列方法
        - `Scrum`：侧重于项目管理：
            - 使用 `Backlog` 管理产品的需求，并进行优化级排序
            - 将开发过程分为若干个短的迭代周期 `Sprint`
            - 在每个 `Sprint` 从产品 `Backlog` 中挑选最高优先级的需求组成 Sprint Backlog
        - 特征驱动开发（`Feature Driven Development`, FDD）

- 统一过程模型（`Rational Unified Process`, RUP）：是 Rational 软件公司创造的软件工程方法，是一种重量级过程。

### CMMI
`CMMI`（Capability Maturity Model Integration for Software，软件能力成熟度模型集成）：
- Level 1 初始级：过程通常是随意且混乱的，成功依赖于人员的能力与英雄主义。
- Level 2 已管理级：组织要确保策划、文档化、执行、监督和控制项目级的过程，并且需要为过程建立明确的目标，并能实现成本、进度和质量目标等。
- Level 3 已定义级：企业能够根据自身的特殊情况定义适合自己企业和项目的标准流程，将这套管理体系与流程予以制度化，同时企业开始进行项目积累，企业资产的收集。
- Level 4 量化管理级：建立了产品质量、服务质量以及过程性能的定量目标。与 3 级的区别在于对过程性能的可预测。
- Level 5 优化级：企业的项目管理达到了最高的境界。关注于通过增量式的与创新式的过程与技术改进，不断地改进过程性能。使用从多个项目收集来的数据对整体的组织级绩效进行关注。


## 系统质量属性与架构评估

系统属性：
- 功能属性
- 质量属性：是一个系统可测量或可测试的属性，用来描述系统满足利益相关者需求的程度。
    - 质量属性有多个标准化的出处和定义，目前广泛认可的软件质量模型标准是 ISO/IEC 25010:2011 。
    - 在实际应用中，组织往往会根据自身需求和行业特性，从这些标准中选择适合的部分，或者制定自己的质量属性框架。
    - 重要的是要注意，这些标准和模型在不断演化。例如，随着技术的发展，一些新的质量属性如可观察性（Observability）和弹性（Resilience）变得越来越重要，尽管它们可能还没有被正式纳入这些标准中。


架构评估普通关注的质量属性有：
- 功能性
- 性能
- 可靠性：在规定的条件和时间内完成预期功能的能力。通常用平均失效等待时间 MTTF 和平均失效间隔时间 MTBF 来衡量。
    - 容错
    - 健壮性
- 可用性：系统能够正常运行的时间比例。
- 安全性：
    - 机密性：保证信息不泄露给未授权用户。
    - 完整性：保证信息的完整和准确，防止信息被非法修改。
    - 不可否认性：信息交换的双方不能否认其在交换过程中发送信息或接受信息的行为。
    - 可控性：保证对信息的传播及内容具有控制的能力，防止被非法者所用。
- 可修改性：
    - 可维护性
    - 可扩展性
    - 结构重组
    - 可移植性
- 可变性：架构经扩充或变更而成为新架构的能力。
- 互操作性


系统架构评估方法：
- 基于调查问卷或检查表
- 基于场景
    - SAAM（Software Architecture Analysis Method）
    - ATAM（Architecture Tradeoff Analysis Method）
    - CBAM（the Cost Benefit Analysis Method）：在 ATAM 的基础上增加了成本效益分析
- 基于度量：
    - 首先建立质量属性和度量之间的映射原则
    - 然后从软件架构文档中获取度量信息
    - 最后根据映射原则分析推导出系统的质量属性


## 如何做技术选型

从以下方面考察：
- 需求满足度：
    - 是否满足当前业务或技术需求
    - 是否满足未来规划或设想的需求
- 非功能质量属性：
    - 性能
    - 可靠性/可用性
    - 安全性
    - 可修改性：可维护性、可扩展性
    - 成熟度与发展趋势：不要选择不怎么维护的开源软件
- 团队亲和性：
    - 研发人员对该技术的掌握情况或快速学习上手的可能
    - 运维人员对该技术的掌握情况


## 互联网三高（高并发、高可用、高性能）

### 可靠性

软件可靠性设计：
- 容错
- 检错
- 降低复杂度

在系统配置中常见的容错技术有：
- 双机热备：
    - 双机热备，Active/Standby
    - 双机互备
    - 双机双工
- 服务器集群

SRE 黄金指标：
- 延迟（Latency）：处理请求所需的时间
- 流量（Traffic）：QPS TPS
- 错误（Errors）：请求失败的比例
- 饱和度（Saturation）：系统资源的使用情况，CPU、内存、磁盘

### 高并发

两个方向、三板斧

两个方向：
- scale up 纵向扩展
- scale out 横向扩展

三板斧：
- 分流
    - 多数据中心
    - 应用服务多实例
    - 数据库：分库分表、主从
    - 分流到其它组件
- 缓存
    - 客户端缓存
    - CDN 缓存
    - Redis 缓存
    - 进程内缓存
- 排队
    - 消息队列
    - 限流

### 性能数据估测值（Numbers everyone should know）

| Operation                          | Latency (ns) | Latency (µs/ms)     | Comparison                               |
| ---------------------------------- | ------------ | ------------------- | ---------------------------------------- |
| L1 cache reference                 | 0.5          |                     |                                          |
| Branch mispredict                  | 5            |                     |                                          |
| L2 cache reference                 | 7            |                     | 14x L1 cache                             |
| Mutex lock/unlock                  | 25           |                     |                                          |
| Main memory reference              | 100          |                     | 14x L2 cache, 200x L1 cache              |
| Compress 1K bytes with Snappy      | 3,000        | 3 µs                |                                          |
| Read 1 MB sequentially from memory | 20,000       | 20 µs               | ~50GB/sec DDR5                           |
| Read 1 MB sequentially from NVMe   | 100,000      | 100 µs              | ~10GB/sec NVMe, 5x memory                |
| Round trip within same datacenter  | 500,000      | 500 µs              |                                          |
| Read 1 MB sequentially from SSD    | 2,000,000    | 2,000 µs (2 ms)     | ~0.5GB/sec SSD, 100x memory, 20x NVMe    |
| Read 1 MB sequentially from HDD    | 6,000,000    | 6,000 µs (6 ms)     | ~150MB/sec 300x memory, 60x NVMe, 3x SSD |
| Send 1 MB over 1 Gbps network      | 10,000,000   | 10,000 µs (10 ms)   |                                          |
| Disk seek                          | 10,000,000   | 10,000 µs (10 ms)   | 20x datacenter roundtrip                 |
| Send packet CA->Netherlands->CA    | 150,000,000  | 150,000 µs (150 ms) |                                          |

简记：
- 内存（10GB/s，100us），磁盘（HDD 100MB/s，10ms），相差 100 倍。
- SSD 与 HDD 相差 5 倍。
- 1Gbps 网络发送 1MB 数据需要 10ms。


## 分布式

CAP：（只能满足两者 CP / AP）
- Consistency（一致性）
- Availability（可用性）
- Partition tolerance（分区容错性）


一致性与共识：
- 一致性：
    - 强一致性
    - 线性一致性：使系统看起来只有一个副本
    - 最终一致性
- 共识算法：
    - Paxos
    - Raft


分布式数据：
- 分区：
    - 键值数据的分区：
        - 根据键的范围
        - 根据键的哈希
            - 一致性哈希
    - 次级索引的分区：
        - document-based 基于文档的分区：每个分区内部维护自己的次级索引，即本地索引，查询时需要分散/聚集，目前被广泛使用。
        - term-based 基于关键词的分区：覆盖所有分区的全局索引。
    - 问题：
        - 负载偏斜
        - 热点
        - 分区再平衡
        - 请求路由
- 复制：
    - 变更复制：
        - 单主
        - 多主：最后写入胜利
        - 无主：法定人数
    - 同步、异步、半同步
    - 复制延迟


## Million RPS

理论分析系统是否能够支撑百万并发？

一个比较典型的系统架构为：
- 网关层：Load Balancer / API Gateway
- 服务层：HTTP Server
- 中间件：Redis、MQ
- 存储层：MySQL

忽略业务复杂性，依次分析：
- LB：常用软件 LB 支持的 RPS 从几万到几十万不等，在高性能硬件配置的情况下能支持百万请求转发。云服务商提供的 LB 宣称支持百万、千万级别。
- HTTP Server：RPS 从几万到几十万不等，甚至几百万，参考 [Web Framework Benchmark](https://www.techempower.com/benchmarks/#section=data-r22&test=fortune)。
- Redis：根据集群的线性扩展性理论，Redis Cluster 可以支撑百万级、甚至千万 QPS。AWS 扩展到了单集群 5 亿 QPS。
- MQ：转换成接受百万条消息的能力，MQ 很容易做到。
- MySQL：根据某云服务商提供的数据，16C64G 并发度 128 支持的 QPS 也不到 14 万，另外还有连接数和线程的限制，数据库是一个明显的瓶颈点，只能做分库、缓存、限流。

网络带宽规格：
- 10 Gbps（10 Gigabit Ethernet, 10GBASE-T）: 用于数据中心、高性能计算（HPC）和企业网络。
- 40 Gbps（40 Gigabit Ethernet, 40GBASE-T）: 多用于数据中心和高吞吐量应用。
- 100 Gbps（100 Gigabit Ethernet, 100GBASE-T）: 高端数据中心和服务提供商网络。

10 Gbps = 1.25 GB/s，假设每个请求大小是 1 KB，那么 1 million 请求需要 1 GB，网络负载很接近极限了。


## 参考资料

- 《系统架构设计师教程》
- 《高并发系统设计40问》
- 《Designing Data-Intensive Applications》
- https://static.googleusercontent.com/media/sre.google/en//static/pdf/rule-of-thumb-latency-numbers-letter.pdf
- https://gist.github.com/BlackHC/2d0a3a21542b524a7cf2f8eac977481e