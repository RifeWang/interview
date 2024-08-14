# Load Balancer

## 分类

- 按类型分类：
    - 硬件负载均衡器
    - 软件负载均衡器
    - 云负载均衡器
- 按网络层级分类：
    - L4
    - L7

常用：
- LVS（Linux Virtual Server）
- NGINX
- HAProxy
- Kong
- Traefik
- Envoy

负载均衡算法：
- 随机
- 轮询、加权轮询
- IP 地址哈希
- 最少连接数
- 最短响应时间

负责均衡 vs API 网关：
- 负责均衡主要关注的是流量分发这一单一功能。
- API 网关涉及更广泛的功能集，包括但不限于 API 管理、安全、转换、监控等。
- 两者在具体的产品上界限比较模糊。


## 限制

- 软件负载均衡器一般单实例支持几万到几十万 QPS 不等，依赖于硬件和配置。
- [HAProxy 单实例每秒两百万请求转发](https://www.haproxy.com/blog/haproxy-forwards-over-2-million-http-requests-per-second-on-a-single-aws-arm-instance)。
- 云负载均衡器可以支持百万、千万每秒请求，比如：[AWS NLB 可以支持每秒百万请求](https://aws.amazon.com/blogs/aws/new-network-load-balancer-effortless-scaling-to-millions-of-requests-per-second/)。