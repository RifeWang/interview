# 后端工程师/系统架构师 - 面试准备 & 系统总结

分层架构：
- client: web / desktop / mobile app
- 通信协议：HTTP（HTTPS、HTTP/2、HTTP/3）/ gRPC
- gateway: nginx / kong / k8s ingress
- service: microservices / 服务注册 / 服务发现
- middleware:
    - MQ: RabbitMQ / NSQ / Kafka
    - Redis
    - Elasticsearch(ELK)
- storage:
    - Relational DB / NoSQL
        - MySQL / InfluxDB / Clickhouse / Milvus
    - 文件存储
    - 对象存储
    - 块存储
- virtual environment:
    - VM
    - docker / kubernetes
    - CICD
- os: Linux

## 工程师基础

- [Git](./basic/git.md)
- [数据结构与算法]()

## 通信协议

- [HTTP](./communication-protocol/http.md)
- [gRPC]()

## 负载均衡 & 网关

- [k8s ingress]()

## 中间件

- [Redis](./middleware/redis.md)
- [Elasticsearch]()

## 存储

- [MySQL](./storage/mysql.md)
- [Clickhouse]()

## 环境

- [Docker](./environment/docker.md)
- [Kubernetes](./environment/kubernetes.md)
- [CICD]()
- [SRE]()

## 操作系统

- [Linux]()

## 编程语言

- [Go](./language/go.md)

## 系统架构

- [分布式系统]()
- [微服务架构]()
- [DDD 领域驱动设计]()