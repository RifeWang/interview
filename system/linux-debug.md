# Linux 环境 Debug

1. 问题：线上环境某个程序卡住了，只执行了一部分，后面的没执行下去。

排查：
- 通过日志定位代码块，第一眼未发现有任何问题
- 想到是否有可能死锁，但直接排除
- 进入线上环境 debug：
    - 拿到进程的 pid，可以通过：
        - ps + grep
        - 容器运行时工具 crictl ps + crictl inspect
    - 查看 `/proc/[pid]/syscall` 文件获取进程的系统调用信息（输出的信息不便于阅读）
    - 使用 `strace -p [pid]` 输出可读的进程系统调用，得到了 `epoll_wait(13, [], 1024, 0)`，说明问题出在网络上。
    - 进一步排查网络情况，使用 `ss -t -i` 查看 TCP 连接的数据接收和发送情况，发现有个 TCP 连接存在了很久，但是又没有数据传输。
    - 通过 `ll /proc/<pid>/fd` 可以查看连接的建立时间。
    - 最后排查代码找网络请求的部分，定位到了使用的某个第三方库进行 http 请求的时候没有设置超时，然后刚好连接就出了问题一直卡在那里。


2. Nginx 日志的 `request_time` 只有 0.001s，但是客户端记录的响应时间却在 6s 以上。

排查：
- `request_time` 记录的是 nginx 从接收到客户端请求的第一个字节到发送完响应的最后一个节点之间的时间，反映的是 nginx 层面的处理时间，时间计算到响应数据通过系统调用发送到操作系统的网络协议栈为止。甚至有可能是 0.000s。
- `request_time` 并不包括网络协议栈之后的延迟（例如 TCP 堆栈的处理、网络传输时延等）。因此，您不需要特别关心网络协议栈的具体处理情况，因为 NGINX 无法获取和记录这部分时间。
- 怀疑网络丢包：
    - 检查网络接口：
        - `ifconfig` / `ip addr show`：检查网卡的统计信息，查看是否有错误包、丢包、重传等，检查 RX 和 TX 统计信息中的 errors 和 dropped 字段。
        - `ethtool eth0`：检查网卡的详细信息，包括链路状态、协商速度等。
    - 系统日志：
        - `dmesg | grep -i eth`：检查系统内核日志，查看是否有与网络相关的错误，例如驱动故障、硬件问题等。
        - `/var/log/syslog` 或 `/var/log/messages`: 查看系统日志是否有相关的错误信息。
    - 监控网络使用：
        - `nload` / `iftop`: 实时监控网络流量，看看是否有网络拥塞的现象。