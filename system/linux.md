# Linux

## IO

- 同步阻塞 IO：
    - 一个进程一个 socket，一个进程只能等待一条连接，每个进程占用约几 MB 内存。
    - socket 数据没到则进程出让 CPU，数据到了再唤醒进程，总共两次进程上下文切换。
- 多路复用 IO：一个进程多个 socket，阻塞说的是进程因为等待某个事件而主动出让 CPU 挂起的操作。
    - select
    - poll
    - epoll：epoll 本身也是阻塞的（但是实践中只要活儿足够多，也不会让进程阻塞），socket 不会阻塞

## 内核发送网络包

内核发送网络包：
- 用户进程（系统调用）-> 协议栈（传输层、网络层）-> 邻居子系统 -> 网络设备子系统 -> 驱动程序 -> 硬中断
- 传输层：申请 skb 拷贝内存；滑动窗口管理；TCP 头设置
- 网络层：查找路由项；netfilter 过滤；IP 分片
- 邻居子系统：发送 arp 请求获取目标 mac

IP 层分片：包含 IP 头在内的数据包大于 MTU 时（以太网为 1500 字节，lo 回环能为 65535 字节）会进行分片。

零拷贝：通过 sendfile 系统调用，数据不需要拷贝到用户空间，在内核态就能完成数据包发送。

                    | <------------ 内核态 ---------------> |
磁盘 -- DMA拷贝 --> Page Cache --> Socket 发送缓存区 --> 拷贝到 skb -- DMA拷贝 --> 网卡


read + send:
                                          | 用户态 |
磁盘 -- DMA拷贝 --> Page Cache -- CPU 拷贝--> 用户内存 -- CPU 拷贝--> Socket 发送缓存区 --> 拷贝到 skb -- DMA拷贝 --> 网卡


## 网络通信

- 跨主机通信：

用户进程 -> 系统调用 -> 协议栈（传输层、网络层）-> 邻居子系统 -> 网络设备子系统 -> 驱动程序 -> 网卡 -> |
                                                                                          |
                                                                                          |
用户进程 <- 进程调度 <- 协议栈（传输层、网络层）<- 网络设备子系统 <- 驱动程序 <- 软中断 <- 硬中断 <- 网卡


- 本机 IO：不需要网卡

用户进程 -> 系统调用 -> 协议栈（传输层、网络层）-> 邻居子系统 -> 网络设备子系统 -> 驱动程序 -> |
                                                                                   |
                                                                                   |
用户进程 <- 进程调度 <- 协议栈（传输层、网络层）<- 网络设备子系统 <- 驱动程序 <- 软中断 <---  |


## TCP 连接

1. 第一次握手：client -- SYN --> server

client：
- 选择可用的本地端口，内核参数 `net.ipv4.ip_local_port_range` 设置了端口号范围，如果端口号分配失败则报错 `Cannot assign requested address`，如果端口使用殆尽会导致查找可用端口号时 CPU 使用率飙升。
- 发送 SYN 握手请求。
- 启动重传定时器。
- 状态变为 SYN_SENT

server：
- 需要事先 listen：申请并初始化接受队列
- 判断队列是否满了：
    - 半连接队列满了会导致丢包，SYN Flood 攻击就是消耗半连接队列，可以开启 `ipv4.tcp_syncookies` 解决
    - 全连接队列满了且有 young_ack（未处理完的半连接请求）也会丢包
- 发出 SYN ACK
- 进入半连接队列
- 启动定时器
- 状态变为 SYN_RECV

2. 第二次握手：server -- SYN ACK --> client

client：
- 清除重传定时器
- 设置为已连接
- 发送 ACK 确认

3. 第三次握手：client -- ACK --> server

client：
- 状态变为 ESTABLISHED

server：
- 创建新 sock
- 从半连接队列删除
- 加入全连接队列
    - `tcp_abort_on_overflow` 为 0：如果队列满了直接丢弃数据包，让 client 等待或者超时
    - `tcp_abort_on_overflow` 为 1：队列满了则发送一个 reset 包给 client，表示废掉这个握手过程和连接，客户端会收到报错 `Connection reset by peer`
- 状态变为 ESTABLISHED

server 端 accept：从全连接队列中取走 socket


**一条 ESTABLISHED 状态的空连接消耗内存大约 3 KB 多一点点。**


## 单机百万 TCP 连接

文件描述符限制：如果不够会触发 `Too many open files` 报错
- 进程级别：
    - `soft_nofile`：可以针对不同用户配置不同的值
    - `fs.nr_open`：一台 Linux 上只能配置一次
- 系统级别：
    - `fs.file-max`：整个系统上可打开的最大文件数，不限制 root 用户


- 一个 TCP 连接由四元组（源 IP，源 Port，目标 IP，目标 Port）唯一确定。
- Port 占用 2 字节（16 位），IPv4 IP 占用 4 字节（32 位）
- 假设服务端 IP 和 Port 确定，那个该服务端理论支持的 TCP 连接数可达 2^48 约 281 万亿
- 一条空连接消耗约 3 KB 内存，一百万连接也只消耗约 3 GB


## 网络性能优化

- 网络请求优化：
    - 减少不必要的网络 IO
    - 合并网络请求
        - 例如使用 Redis 的 hmget、pipeline 等
    - 部署离得近些，降低 RTT
    - 内网调用不要走外网域名

- 接受过程优化：
    - 调整网卡 RingBuffer 大小
    - 多队列网卡 RSS 调优
    - 硬中断合并：攒一堆数据包后再通知CPU，减少中断数量虽然能使网络包吞吐更高，不过一些包的延迟也会更大
    - 软中断 budget 调整：`net.core.netdev_budget` 集中处理一波网络包的数量
    - 接受处理合并：配置 LRO（Large Receive Offload）/ GRO（Generic Receive Offload）把数据包合并后再向上层传递
        - LRO 在网卡完成合并，需要硬件支持
        - GRO 在内核中用软件的方式实现

- 发送过程优化：
    - 控制数据包大小
    - 减少内存拷贝
    - 推迟分片：IP层检查数据大于 MTU 的话会分片，但是可以开启 TSO（TCP Segmentation Offload）/ GSO（Generic Segmentation Offload）推迟到更下层的设备去做
    - 多队列网卡 XPS 调优：通过 XPS 指定 CPU 使用的发送队列
    - 使用 eBPF 绕开协议栈的本地网络 IO

- 内核与进程协作优化：
    - 少用同步阻塞 IO，多用多路复用 IO
    - 使用成熟的网络库
    - 使用 Kernet-ByPass 技术：绕过协议栈，自己在用户态处理

- 握手挥手过程优化：
    - 配置充足的端口范围
    - 客户端不要 bind 端口（一般也不会这么做）
    - 小心队列溢出
    - 减少握手重试：`tcp_syn_retries`，`tcp_synack_retries` 控制重传次数
    - 打开 TFO（TCP Fast Open）：`net.ipv4.tcp_fastopen`，客户端的第三次握手 ack 包就可以携带要发送给服务端的数据
    - 保持充足的文件描述符上限
    - 请求频率则使用长连接
    - `TIME_WAIT` 优化：TIME_WAIT 是四次挥手时客户端 CLOSE 前的状态，开启端口 reuse 和 recycle，或使用长连接。
        - 为什么需要 TIME_WAIT：防止历史连接中的数据，被后面相同四元组的连接错误的接收；保证「被动关闭连接」的一方，能被正确的关闭；
        - TIME_WAIT 的等待时长是 2 MSL（Maximum Segment Lifetime），Linux 系统停留在 TIME_WAIT 的时间为固定的 60 秒。


## 参考文章

- 《深入理解Linux网络 · 修炼底层内功，掌握高性能原理》
- https://xiaolincoding.com/network/3_tcp/tcp_interview.html