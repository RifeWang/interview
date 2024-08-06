# HTTP

## HTTP 协议
请求报文格式：
  method  url  协议版本
  请求头 request headers
  空行
  请求体 request body

响应报文格式：
  协议版本  状态码  状态码短语
  响应头 response headers
  空行
  响应体 response body

常用状态码：
- `200`: OK
- `204`: No Content
- `206`: Partial Content 断点续传或者分段下载 对应 `Range` header

- `301`: 永久重定向
- `302`: 临时重定向
- `304`: Not modified，表示资源在由请求头中的 `If-Modified-Since` 或 `If-None-Match` 参数指定的这一版本之后，未曾被修改。

- `400`: Bad request
- `401`: Unauthorized 未认证
- `403`: Forbidden
- `404`: Not Found
- `405`: Method Not Allowed
- `408`: Request Timeout
- `413`: Request Entity Too Large
- `415`: Unsupported Media Type
- `429`: Too Many Requests

- `500`: Internal Server Error
- `502`: Bad Gateway
- `503`: Service Unavailable 服务不可用
- `504`: Gateway timeout

## HTTP 缓存
缓存控制，协商缓存相关的头部之间的优先级关系

- `强缓存`：浏览器不会向服务器发送任何请求，直接从本地缓存中读取文件
  - 200 from memory cache 浏览器关闭后数据会被释放掉
  - 200 from disk cache 浏览器关闭后数据仍然存在，下次仍然可用

  - `Expires`：GMT 时间，过期时间，如果设置了过期时间，则浏览器会在该时间内直接读取缓存
  - `Cache-Control`：（ `Cache-Control` 是 http 1.1 ，`Expires` 是 http 1.0 ，前者优先级更高，会覆盖后者）
    - 可以有多个字段，逗号分隔：
    - `max-age`：资源可以被缓存多长时间，单位为秒，比如 max-age = 300
    - `public`：响应可被任何缓存区缓存
    - `private`：只能针对个人用户，不能被代理服务器缓存
    - `no-cache`：不是说不缓存，而是强制要求必须向服务器发送请求，由服务器评估缓存的有效性
    - `no-store`：禁止一切缓存

- `协商缓存`：向服务器发送请求，服务器根据请求的 header 参数判断是否命中缓存
  - `Etag` / `If-None-Match` 成对出现，一一对应
  - `Etag`: 服务器生成，帮助服务器控制 web 端的缓存验证，哈希值
  - `If-None-Match`: 资源过期时，浏览器发现响应头里有 Etag，则再次向服务器请求时带上请求头 `If-None-Match` 值是 `Etag` 的值，服务收到请求后进行对比，决定返回 200 或 304

  - `Last-Modified` / `If-Modified-Since`
  - `Last-Modified`: 浏览器向服务器发送资源最后的修改时间
  - `If-Modified-Since`: 当资源过期时（浏览器判断 Cache-Control 标识的 max-age 过期），发现响应头具有 `Last-Modified` 声明，则再次向服务器请求时带上头`If-modified-since`，表示请求时间。服务器收到请求后发现有 `If-modified-since` 则与被请求资源的最后修改时间进行对比 `Last-Modified` ,若最后修改时间较新（大），说明资源又被改过，则返回最新资源，HTTP 200 OK;若最后修改时间较旧（小），说明资源无新修改，响应HTTP 304 走缓存。

  - `Last-Modifed / If-Modified-Since` 的时间精度是秒，而 `Etag` 可以更精确。
  - `Etag` 优先级是高于 `Last-Modifed` 的，所以服务器会优先验证 `Etag`
  - `Last-Modifed / If-Modified-Since` 是 http1.0 的头字段


## HTTP stream

### Content-Length
通常 `Content-Length` 会指定 request/response body 的 byte 长度，但是在 stream 的情况下，无法使用。
如果没有 `Content-Length` 头, http 服务器会隐式添加 `Transfer-Encoding: chunked` 头。
`Content-Length` 和 `Transfer-Encoding: chunked` 不能同时使用。
`Content-Length` 和 `Range` 可以配合使用完成部分下载、暂停下载、恢复下载等。

### Transfer-Encoding

`Transfer-Encoding: chunked` 表示 stream 传输，数据是 chunked


## HTTPS

HTTPS = HTTP + SSL/TLS , 采用混合加密机制
- 使用公开密钥加密（非对称加密）交换密钥
- 使用共享密钥加密（对称加密）进行通信


## HTTP2/3

HTTP/1.1 的缺点：
- 请求-响应模型，队首阻塞
- Header 头部重复且巨大
- 文本传输
- 服务器不能主动推送


HTTP/2 ：
* 优点：
  - Header 头部压缩：两端维护字典，传递索引号，降低了数据传输大小
  - 二进制传输，多路复用，TCP -> stream -> message -> frame：
    - 1 个 TCP 连接包含一个或者多个 Stream，Stream 是 HTTP/2 并发的关键技术；
    - Stream 里可以包含 1 个或多个 Message，Message 对应 HTTP/1 中的请求或响应，由 HTTP 头部和包体构成；
    - Message 里包含一条或者多个 Frame，Frame 是 HTTP/2 最小单位，以二进制压缩格式存放 HTTP/1 中的内容（头部和包体）；
  - 服务器推送
* 缺点：基于 TCP 和 SSL/TLS
  - 队首阻塞：HTTP/2 多个请求在一个 TCP 连接中，当 TCP 丢包时，整个 TCP 都要等待重传，该 TCP 连接中的所有请求都会被阻塞。
  - TCP 与 TLS 的握手延迟
  - 网络迁移需要重新连接：一条 TCP 连接由（源 IP、源端口、目的 IP、目的端口）确定，当网络环境变化比如 4G 切换成 WiFi 时需要重新建立连接，另外 TCP 协议存在「慢启动」的特性。


HTTP/3 QUIC ：基于 UDP ，在应用层增加可靠性传输
- 无队首阻塞：基于 UDP ，多个 stream 互不影响
- 实现快速握手，连接建立更快
- 连接迁移：不使用四元组，而是通过「连接 ID」来标记通信的两个端点


## DNS 寻址过程？

`DNS`: Domain Name System 域名系统

过程：
1. 浏览器 DNS 缓存
2. 本地操作系统 DNS 缓存
3. 查询 `/etc/hosts` 文件
4. DNS 查询：
  - 域名服务器（`/etc/resolv.conf` 文件），根域名服务器 -> 顶级域名服务器 -> 递归下级域名服务器
  - www.google.com. :  . 根域名 > .com > .google.com > www.google.com

HTTP 建立在 TCP 连接之上，TCP 在 IP 层之上，所以需要先进行 DNS 确认 IP 地址才能建立 TCP 连接。


## HTTP Keep-Alive & TCP keepalive

- HTTP 的 `Connection: keep-alive` 由应用程序实现，使得一个 TCP 连接可以处理多个 HTTP 请求响应。
- TCP 的 keepalive 由内核实现，用来检查长时间没有数据交互时对方是否还在线，防止中间设备（如 NAT）关闭连接。


## 参考文章

- https://xiaolincoding.com/network/2_http/http2.html