# 从 HTTP1.\* 到 HTTP2.0

## 什么是 HTTP

**HTTP（Hyper Text Transfer Protocol）**：超文本传输协议，用于传输超媒体文档的应用层协议。

### HTTP 作用

传输超媒体（对超文本的术语延伸，包含图片、音频、视频、文本、超链接的非线性消息媒体）。HTTP 是万维网的数据通信的基础，设计 HTTP 最初的目的是为了提供一种发布和接收 HTML 页面的方法，通过 URI 来标识唯一资源

> 超文本（英语：Hypertext）是一种可以显示在电脑显示器或电子设备上的文本，现时超文本普遍以电子文档的方式存在，其中的文字包含有可以链接到其他字段或者文档的超链接，允许从当前阅读位置直接切换到超链接所指向的文字

### HTTP 特点

- 客户端-服务端模型：由客户端发起请求（Request），服务端返回响应（Response）的方式进行数据交互
- 文本协议
- 无状态协议：HTTP 自身不对通信状态进行保存
- 无连接协议：HTTP 通信一次就需要连接/断开 TCP

HTTP 创建之初，主要用于传输文字，文字可能带有超链接。在那时候内容远远没有现在丰富，排版也远远没有现在精美，交互也远远没有现在复杂，对于当时这种简单的应用场景，HTTP 表现得还可以

但是随着互联网的高速发展，WEB2.0 的兴起，更多丰富的内容，更精美的排版，更加复杂的交互等等。导致随随便便一个网站的大小都会比原来的网站大很多，更不用说社交、电商网站。请求数量多，请求文件大会引发一个问题，那就是*加载速度*。影响网络请求主要有两个因素，*带宽*以及*延迟*，我们将在恒定带宽下，分析 HTTP 协议的延迟

## HTTP 延迟分析

### 文本协议

HTTP1.\* 是文本协议，文本协议具有可读性好，扩展性好等优点。但与二进制协议相比，文本协议浪费带宽，传输效率较低。那么 HTTP2 是如何在不改变 HTTP1.\* 的语义情况下，提升性能呢

**二进制分帧**：在 HTTP 协议和 TCP 协议之间，加入二进制分帧层

![](https://public.keven.work/HTTP2_%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%88%86%E5%B8%A7%E5%B1%82.jpg)

HTTP1.\* 保持它的语义，只是在传输的时候，编码方式变了。HTTP1.\* 以换行符作为纯文本的分割，而 HTTP2 中将所有传输的信息分割为 Message 和 Frame，并采用二进制格式编码

```text
HTTP1.* Headers => HTTP2 Headers Frame
HTTP1.* Body Entity => HTTP2 Data Frame 中
```

![](https://public.keven.work/HTTP2_Stream_Message_Frame.jpg)

- Stream（数据流）: 一个 HTTP2 连接的虚拟信道，具有双向承载传输。

  - 每个连接可以创建多个 Stream，每个 Stream 都有一个唯一标识符，为了防止客户端和服务端标识符冲突，客户端发起的采用奇数 ID， 服务端发起的采用偶数 ID，流标识符零（0x0）用于连接控制消息，流标识符 0 不能用于建立新流
  - 可以向每个 Stream 分配一个介于 1 至 256 之间的整数，以控制 Stream 优先级
  - 每个 Stream 与其他 Stream 之间可以存在显式依赖关系

  数据流依赖关系和权重的组合让客户端可以构建和传递“优先级树”，表明它倾向于如何接收响应 [Google - 数据流优先级](https://developers.google.com/web/fundamentals/performance/http2?hl=zh-cn#%E6%95%B0%E6%8D%AE%E6%B5%81%E4%BC%98%E5%85%88%E7%BA%A7)

- Message（消息）: 表示一个完整的请求和响应，由一个或多个 Frame 组成
- Frame（帧）: HTTP2 的传输基本单元，帧分为 Headers Frame 和 Data Frame

HTTP1.\* 请求和 Frame 转换

![](https://public.keven.work/HTTP2_Frame.jpg)

### 连接无法复用

由于 HTTP 无法复用连接，则每次 HTTP 发起请求时，都会发起 TCP 连接，HTTP 结束请求时，断开 TCP 连接，由于如下原因

1. 由于 TCP 三次握手会额外多出 1RTT 时间，增加延迟
2. 由于 TCP 的拥塞机制中的慢启动（Slow Start），新的连接总是要过一段时间才能高效传输

**RTT（Round Trip Time）**: 往返时间，从发送端发送数据开始，到接收到来自服务端的确认的时间间隔。可以使用 ping、tcpping 来测试

#### Keep-Alive

所以在 HTTP1.\* 引入 Keep-Alive 头部

```text
 Connection: Keep-Alive # HTTP1.1 默认开启，HTTP1.0 需要显示设置头部
 Keep-Alive: timeout=10 # 可以设置超时时间

 Connection: close # 显示的表示关闭长连接，或等待超时
```

需要注意的是，Connection 为 Hop-by-hop 头部，只对单次转发有效，遇到缓存服务器和代理服务器则不再转发

长连接也还是有缺点的

1. 就算是在空闲状态，它还是会消耗服务器资源
2. 在服务器重负载时，还有可能遭受 DoS 攻击。这种场景下，可以使用非长连接，即尽快关闭那些空闲的连接，也能对性能有所提升

#### 其他复用连接的方式

不只是 Keep-Alive 头部，还有其他方法进行长连接（伪长连接）

1. HTTP 轮询、HTTP 长轮询
2. HTTP Stream
3. WebSocket

### 队头阻塞

默认情况下，HTTP 请求时按顺序发出，下一个请求只能等待当前请求得到响应才能发出。这是巨大的延迟

#### Pipelining

HTTP1.1 引入 Pipelining（流水线），客户端可以同时发送多个请求，在发送过程中不必先等待服务器响应。

![](https://public.keven.work/HTTP_Pipeling.jpg)

但是也存在如下缺点

1. 服务器可能不支持
2. 代理服务器可能不支持
3. 虽然请求可以同时发出，但是返回也是依次的，请注意，_并不是 response 先返回先处理，而是按照 FIFO 原则_，这样可能导致新的问题 **Front of queue blocking（队列前阻塞）**

所以有的浏览器要么默认关闭 Pipelining，要么直接不给使用，譬如[Chromium HTTP Pipelining](https://www.chromium.org/developers/design-documents/network-stack/http-pipelining) [Enable HTTP pipelining by default](https://bugzilla.mozilla.org/show_bug.cgi?id=264354)

#### 域名分片

HTTP1.\*中的请求是序列化的，即使本来是无序的。浏览器为每个域名建立多个连接，以实现并发请求。曾经默认的连接数量为 2 到 3 个，现在比较常用的并发连接数已经增加到 6 个。

我比较常用 FireFox 和 Chrome，FireFox 可以使用 `about:config` 进行修改并发连接数量，但 Chrome 貌似不行

为了增加并发，可以采用不同的域名来传输，从而提高性能，但是

1. 增加了资源的消耗，换取用户的等待时间
2. 可以减少请求数据，静态资源服务器不需要 Cookie，将这些资源拆分出来，以减少传输大小

域名分片可以增加并发，但并不是毫无限制

1. 对服务器进行保护
2. 但并不是域名增加的越多越好，每增加一个域名，都要承担 资源消耗增加 "DNS 解析 + 三次握手 + 慢启动" 的额外延迟，所以需要寻找平衡点。tips 可以使用`dns-prefetch、preconnect` 进行优化

#### Multiplexing

虽然 HTTP1.1 提供 Pipelining，但并没有完全解决队头阻塞，而且由于自身缺点，浏览器要么默认关闭，要么就直接不让用。所以 HTTP2.0 使用一种新的流水线实现 Multiplexing

![](https://public.keven.work/HTTP2_%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8.jpg)

多路复用允许对一个域名建立的 HTTP2 请求中，发起多个请求-响应（Message），每个 Message 被拆分个多个 Frame，Frame 可以乱序传输，最终在客户端将 Frame 拼接为 Message。请注意，这里的乱序传输指的是不同请求，即*不同 Message 的 Frame 可以互相穿插，对于同 Message 的 Frame，还是遵从 FIFO 的*

多路复用具有如下特点：

1. 在一个 TCP 连接中，可以承载多个 Stream
2. 并行交错地发送多个请求、响应，之间互不影响，即使用一个连接并行发送多个请求和响应
3. 不必再为绕过 HTTP/1.x 限制而做很多工作，例如*域名分片、Keep-Alive*
4. 消除不必要的延迟和提高现有网络容量的利用率，从而减少页面加载时间

HTTP2.0 的连接都是持久化的，而且针对每个域名只存在一个连接。且在 HTTPS 中，可以减少较大的 TLS/SSL 开销，提高会话重用率，从而整体的减少服务端和客户端资源消耗

多路复用解决了 HTTP1.\*中的队头阻塞和队列前阻塞的问题，但为此又会引发一个新的问题，虽然这不是 HTTP 的锅

HTTP2.0 解决了 HTTP 的阻塞问题，确实解决了 HTTP 协议的延迟，但由于 TCP 的队头阻塞且 HTTP2 只是单个 TCP 连接，随着丢包率增加，HTTP2 性能越来越低，大约在 2% 的丢包率（一个很差的网络质量）中，测试结果表明 HTTP/1.\* 用户的性能更好。因为 HTTP/1.\* 一般有多个 TCP 连接，哪怕其中一个连接阻塞了，其他没有丢包的连接仍然可以继续传输 [TCP 队头阻塞](https://http3-explained.haxx.se/zh/why-quic/why-tcphol#tcp-dui-tou-zu-sai-head-of-line-blocking)

### 头部开销

每个 HTTP 传输都承载一组标头，这些标头说明了传输的资源及其属性。 在 HTTP/1.\* 中，此元数据始终以纯文本形式，通常会给每个传输增加 500–800 字节的开销。如果使用 HTTP Cookie，增加的开销有时会达到上千字节（[请参阅测量和控制协议开销 ](https://hpbn.co/http1x/#measuring-and-controlling-protocol-overhead)）为了减少此开销和提升性能，HTTP/2 使用 HPACK 压缩格式压缩请求和响应标头元数据

#### HTTP2 头部压缩 HPACK

使用 HPACK 算法，对头部进行压缩

1. 支持通过静态*霍夫曼代码*对传输标头字段进行编码，从而减少传输。当然，如果使用霍夫曼编码后比原数据更大，则不会采用霍夫曼编码
2. 要求客户端和服务器同时维护和更新一个包含之前见过的标头字段的*索引列表*
   - 对于已存在索引表中的数据，则会发送索引
   - 对于在索引表中不存在的数据，则会进行缓存，下次发送时则只会发送索引
   - 对于重复数据或者说未改变的数据，则不发送

![](https://public.keven.work/HTTP2_%E5%A4%B4%E9%83%A8%E5%8E%8B%E7%BC%A9.jpg)

> 注：在 HTTP/2 中，请求和响应标头字段的定义保持不变，仅有一些微小的差异：所有标头字段名称均为小写，请求行现在拆分成各个 :method、:scheme、:authority 和 :path 伪标头字段

#### 索引表

索引表分为：静态索引表和动态索引表

##### 静态索引表

静态索引表是预定义的一些 Header 字段，且该表是有序和只读的

| Index | Header Name | Header Value |
| :---: | :---------: | :----------: |
|   1   | :authority  |
|   2   |   :method   |     GET      |
|   3   |   :method   |     POST     |
|   4   |    :path    |      /       |
|   5   |    :path    | /index.html  |
|  ...  |

如果想查看全部静态表定义，请点击 [static.table.definition](https://httpwg.org/specs/rfc7541.html#static.table.definition)

##### 动态索引表

动态以 _先进先出（队列）_ 维护，动态表最初是空的，当每个 header 块被解压缩时，将添加条目新的条目。动态表总是从最低索引处插入，即动态表最新的在最低索引处，最旧的在最高索引处。动态表条目可以包含重复的对

举个栗子

我需要传输如下头部（左表），在经过静态索引表压缩后，变成传输右表

![](https://public.keven.work/HTTP2_%E9%9D%99%E6%80%81%E8%A1%A8%E5%8E%8B%E7%BC%A9.jpg)

假设 `origin: https://...` 在动态表中索引为 65，则下次再传输 `origin: https://...` 则只会传输 `index = 65`

## HTTP3.0 简单介绍

由于 HTTP2.0 多路复用中介绍的 TCP 延迟，所以 Google 另起炉灶， 基于 UDP 协议搞了个 QUIC 协议，HTTP3.0 就是基于 QUIC 协议

### 0 RTT

HTTP2 的延迟

- 创建连接：1RTT(TCP) + 2RTT(SSL) 延迟
- 会话复用：1RTT(TCP) + 1RTT(SSL) 延迟

QUIC 协议可以实现 0RTT 就行进行数据发送

### 多路复用

上面介绍了 HTTP2 的多路复用，但 TCP 是没有这些功能的，且由于 TCP 协议原因，在丢包的情况下，HTTP2 表现较差，所以 QUIC 原生就实现了多路复用，单个传输流不会影响到其他的数据流

### 加密认证报文

TCP 协议是没有加密和认证的，在传输过程中可能被监听或篡改，这些可能是出于性能优化或者是主动攻击。但 QUIC 除了个别报文比如 PUBLIC_RESET 和 CHLO，所有报文头部都是经过认证的，报文 Body 都是经过加密的

### 向前纠错机制

QUIC 数据报除去自身数据外，还包含其他数据包的部分数据，因此在少量丢包的情况下，可以通过冗余数据来组装，而不需要重传。向前纠错增加了数据报大小，但可以减少重传

## HTTP 一些常见问题

1. POST 发送两个 TCP 包吗

   - HTTP 协议中没有明确说明 POST 会产生两个 TCP 数据包
   - 使用 WireShark 抓包，测试如下

   ```javascript
   // 启动 node 服务器
   const http = require('http');

   http
     .createServer(function (req, res) {
       res.end('Hello word');
     })
     .listen(9001);
   ```

   ```shell
   // curl 请求
   curl -X POST http://localhost:9001
   ```

   ![](https://public.keven.work/HTTP_POST_TCP%E6%8A%93%E5%8C%85.jpg)
   如截图证明，WireShark 只发送了一个 TCP 包

2. GET 请求不能携带 Body Entity 吗

   HTTP 并未规定 GET 方法不能携带 Body Entity，测试如下

   ```javascript
   // 启动 node 服务器
   const http = require('http');

   http
     .createServer(function (req, res) {
       req.on('data', (v) => {
         console.log(v.toString()); // 打印 body
       });
       res.end('Hello word');
     })
     .listen(9001);
   ```

   ```shell
   // curl 请求
   curl -X GET -d Test http://localhost:9001
   ```

   ![](https://public.keven.work/HTTP_GET_Body.jpg)

   node 服务器打印和 WireShark 截图证明，node 服务器可以读取到数据

3. GET 请求对 URL 长度有限制吗

   HTTP 并为对 GET 请求 URL 有长度限制，而是浏览器对 GET 请求有长度限制

4. POST 比 GET 更安全吗

   从 HTTP 来说，HTTP 都是裸奔传输的，所以 POST 比 GET 更安全就无从说起。但是 GET 可以被缓存，可以被浏览器历史记录保存，而 POST 不行，所以从某方面来说 POST 还是相对 GET 更安全些

5. 文本协议和二进制协议

   文本协议和二进制协议最大的一个区别就是*编码*，无论是文本格式还是二进制格式，存储、传输肯定都是二进制格式的，但二进制并没有特殊意义，例如 `1000001(65)` 这一串数字，也没什么特殊意义，知道了也没什么用，但如果使用 ASCII 编码，那么它就有用了，表示 `A`

   HTTP1.\*就是用的时 ASCII 编码（请求行、响应行和头部）；实体可以接受任意编码，由 Transfer-Encoding、Content-Encoding 指定编码

[HPACK: Header Compression for HTTP/2](https://httpwg.org/specs/rfc7541.html)  
[Hypertext Transfer Protocol Version 2](https://httpwg.org/specs/rfc7540.html)  
[详解 HTTP/2 头压缩算法 —— HPACK](https://halfrost.com/http2-header-compression/)  
[HPACK 完全解析](https://www.jianshu.com/p/f44b930cfcac)  
[http3-explained](https://http3-explained.haxx.se/zh/zh)  
[http2-explained](https://http2-explained.haxx.se/zh)
