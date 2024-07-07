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


## HTTPS

HTTPS = HTTP + SSL/TLS , 采用混合加密机制
- 使用公开密钥加密（非对称加密）交换密钥
- 使用共享密钥加密（对称加密）进行通信

## HTTP2/3

- 头部压缩
- 二进制传输，数据流，分包
- 多路复用
- 服务器推送

HTTP/3 QUIC
- 基于 UDP ，增加可靠性传输
- 实现快速握手（ TCP + TLS 需要两次握手 ）
- 多路复用，同一个物理连接上可以有多个独立的逻辑数据流，互不影响

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


## DNS 寻址过程？

`DNS`: Domain Name System 域名系统
本地 /etc/hosts 域名映射
本地 /etc/resolv.conf 找域名服务器

浏览器 DNS 缓存 -> 本地 DNS 缓存 -> 根域名服务器，返回下一级的 DNS 服务器 -> 递归一级一级的找

www.alibaba.com. :  . 根域名 > .com > .alibaba.com > www.alibaba.com


简述 CDN 原理？

通过智能调度 DNS 去指定最优的 CDN 节点