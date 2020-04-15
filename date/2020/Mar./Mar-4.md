## Node

### 守护进程

守护进程: 守护进程*父进程先于子进程退出*，则子进程称为孤儿进程，由 Init 进程继承。守护进程不受终端控制，运行于后台。守护进程非交互式程序，没有控制终端，且输入和输出都需要特殊控制，_但可以使用 kill pid 终止_

**原理**

1. 每个进程都隶属于一个**进程组**，进程组 ID（gid）为进程组长 ID，每个进程组长可以为进程组内设置 gid
2. 在终端运行时，每个命令面板都存在**会话期**，是一个或多个进程组的集合。子进程的登录会话和 gid 都是继承于父进程，所以调用 setsid 摆脱
3. 当进程调用 setsid 时，如果该进程非进程组长，则会产生如下变化

   - 该进程与原先*父进程、进程组脱离关系*，自身成为进程组长
   - 与终端解除关系

tips: **但如果本身就是进程组长，则调用 setsid 报错**

**步骤**

1. 父进程 fork，父进程结束，子进程继续运行
2. 子进程调用 setsid，脱离原终端，脱离父进程，将自身变为进程组组长。此时子进程虽然脱离原有终端，但任然可以自己申请一个终端，所以*可以在进程中再次 fork，在子子进程中再次调用 setsid*
3. 关闭无用文件描述符
4. SIGNAL 信号处理

```cpp
#include <sys/types.h>
#include <unistd.h>
#include <fstream>
#include <iostream>

int main() {
  using namespace std;

  pid_t pid = fork();

  if (pid == 0) {
    // 子进程
    pid_t sid = setsid();
    ofstream out;

    out.open("./log.txt");

    for (int i = 1; i < 10; i++) {
      // 由于脱离了终端，该信息不会打印到终端，可以输出到文件中
      out << "I AM RUN IN CHILD" << endl;
      sleep(1);
    }
  } else {
    cout << "EXIT PARENT" << endl;
    exit(EXIT_SUCCESS);
  }
}
```

#### Node 实现

node 不存在 setsid 方法，且也无法调用，但是在 child_process 模块中，存在替代 `child_process.spawn(command[, args][, options])`;

```typescript
options: {
  detached: boolean; // 准备子进程独立于其父进程运行。具体行为取决于平台
}
```

```typescript
// main.js
import { spawn } from 'child_process';

spawn('ts-node', ['c.ts'], {
  detached: true,
});
process.exit();
```

```typescript
// c.ts
import { writeFileSync } from 'fs';

for (let i = 0; i < 10; i++) {
  setTimeout(() => {
    writeFileSync('./log.txt', 'I AM RUN IN CHILD \n', { flag: 'a' });
  }, 1000 * i);
}
```

## HTTP

#### 虚拟主机、虚拟机

虚拟机: 使用软件虚拟操作系统，是对计算机的抽象

虚拟主机: 在网络服务器上划分磁盘空间，供用户放置站点、应用组件，提供站点的数据存放、传输等功能。虚拟主机是使用特殊的软硬件技术，把一台真实的物理服务器主机分割成多个逻辑存储单元。每个逻辑单元都没有物理实体，但是每一个逻辑单元都能像真实的物理主机一样在网络上工作，具有单独的 IP 地址（或共享的 IP 地址）、独立的域名以及完整的 Internet 服务器（支持 WWW、FTP、E-mail 等）功能

当单个 IP 地址对应多个虚拟主机（不同域名），如何对请求进行划分? 使用 Host 头部

**Host**: 指明此次请求服务器域名，HTTP/1.1 中，请求报文必须含有该头部，否则将返货 400 状态码

### 代理、网关、隧道

#### HTTP 代理

HTTP 代理: HTTP 中间人

Client <=> HTTP Proxy <=> Server

**正、反向代理**

1. 对客户端透明: 正向代理
2. 对服务端透明: 反向代理

**透明、非透明代理**

1. 透明代理: 不对报文进行任何处理
2. 非透明代理: 需要处理报文

**缓存、非缓存代理**

1. 缓存代理: 缓存服务器返回的资源
2. 非缓存代理: 不缓存服务器返回的资源

Via: 代理服务器进行转发时，需要在 Via 首部标记自己的信息，用于追踪转发情况

X-Forwarded-For: 当经过代理或负载均衡时，会在 X-Forwarded-For 后添加自己的 IP 地址，会表现为 _X-Forwarded-For: CLient-IP, Proxy-IP, Proxy-Ip..._

#### 网关

网关: 转发其他服务器通信数据的服务器，可以提供非 HTTP 服务。

#### Http 隧道

HTTP Tunnel: HTTP1.1 引入新功能，主要解决 HTTP Proxy 无法在 HTTPS 中使用的问题，同时提供作为任意流量的 TCP 通道能力。

Client => HTTP Tunnel => Server

**代理、网关区别:** HTTP 代理只提供 HTTP 服务；**网关提供非 HTTP 服务**，例如数据库

**代理、隧道区别**: 代理对请求、响应数据完全可见，且会对请求进行适当修改；隧道对于请求不关心，把客户端和服务端**数据原样透传**

举个栗子: 例如 HTTPS，如果使用 Proxy，则 Client 需要和 Proxy SSL 握手，但 Proxy 没有自身的证书，也没有 Server 的证书，所以根本无法建立连接。所以此时需要使用 HTTP Tunnel，Tunnel 不在作为中间人，不再改写客户端请求，让客户端和服务器通信数据透传，客户端和服务端可以直接 SSL 握手并通信

### 首部按照类型分类

#### 通用首部

请求和响应都存在的首部

##### Cache-Control

1. Request

   - no-cache: 可以使用缓存，但强制向资源服务器再次验证
   - no-store: 不使用缓存
   - max-age=[秒]: 缓存服务器，响应最大的时间
   - max-state: 可接收已过期效应（过期时间多少秒内任然有效）
   - min-fresh: 期望在指定时间内任然有效
   - ...

2. Response
   - public
   - private
   - no-cache
   - no-store
   - no-transform: 代理不能更改 MIME 类型
   - max-age=[秒]

##### Connection

**逐跳首部**

1. 控制首部在下次请求不在发送，_Connection 标记的首部，在下次请求（转发）后不在发送_，例如

```
Upgrade: websocket
Connection: Upgrade // Upgrade 在转发一次后，不会再传输 Hop To Hop
```

2. 管理连接
   - Keep-Alive（HTTP/1.1 默认）
   - close

```
Connection: Keep-Alive
Keep-Alive: timeout=5, max=100
```

##### Date

请求、响应何时创建，在响应中，Date 是必返回字段

```
Date: Tue, 03 Jul 2012 00:00:00 GMT
```

##### Pragma

HTTP 1.0 中控制缓存，**缓存优先级最高** Pragma > Cache-Control > Expires

```
Pragma: no-cache
```

##### Transfer-Encoding

**分块传输时有效**

```
Transfer-Encoding: chunked | none
```

##### Upgrade

协议升级

```
Upgrade: websocket
```

##### Via

代理主机信息

##### Warning

警告信息

#### 请求首部

##### 内容协商

1. Accept: 可接受 MIME 类型，text/html
2. Accept-Language: 可接受语言
3. Accept-Charset: 可接受字符集 utf-8
4. Accept-Encoding: 可接受内容编码（压缩方式）gzip、br、compress

##### 缓存

1. Last-Modified-Since: 缓存请求首部
2. Last-UnModified-Since: 确定文件是否**未改变**，如果未改变则返回数据，改变则返回 412
3. If-Match: 缓存请求首部
4. If-None-Match: 确定文件是否**未改变**，如果未改变则返回数据，改变则返回 412

##### Expect: 确定得到状态码

##### Range: 分块请求、传输

##### Host: 主机域名

##### Refer: 来自哪个 URL

##### User-Agent: 客户端代理

##### Max-Forwards: 最大跳转数量

#### 响应首部

##### Accept-Range

```
Accept-Range: bytes | none
```

##### ETag

##### Location

重定向 URL

...

#### 实体首部

##### 内容协商

1. Content-Type => Accept
2. Content-Encoding => Accept-Encoding
3. Content-Charset => Accept-Charset
4. Content-Language => Accept-Language
5. Content-Length: 实体传输大小（字节）
6. Content-MD5
7. Content-Range => Range

##### 缓存

8. Expires
9. Last-Modified

### 首部按照是否端-端分类

#### End-To-End

Client => Proxy .... Proxy => Server

Client 请求首部到 Server

#### Hop-To-Hop

Client => Proxy .... Proxy => Server

Client 请求首部到 Proxy，后续不会再传输

1. Connection
2. Upgrade
3. Keep-Alive
4. Transfer-Encoding
5. Proxy-Authenticate
6. Proxy-Authorization
7. Trailer
8. TE

其余都为 End-To-ENd 首部

## LeetCode

### 三维形体的表面积

思路: 累计

1. 计算每个方块的上下、左右、前后的面积
2. 上下面积无重复，左右、前后都可能重复，则需要去掉重复面积

**注意事项**: 面积可能被覆盖；

```typescript
function surfaceArea(grid: number[][]) {
  let all = 0;
  const n = grid.length;

  // i,j分别对应上下左右
  const iList = [1, -1, 0, 0];
  const jList = [0, 0, -1, 1];

  for (let i = 0; i < n; i++) {
    for (let j = 0; j < n; j++) {
      all += handle(i, j);
    }
  }
  return all;

  function handle(i: number, j: number): number {
    if (grid[i][j] === 0) return 0;

    const m = grid[i][j];
    let s = 2;

    for (let x = 0; x < 4; x++) {
      const iNew = i + iList[x];
      const jNew = j + jList[x];

      if (iNew >= 0 && iNew < n && jNew >= 0 && jNew < n) {
        s += Math.max(m - grid[iNew][jNew], 0);
      } else {
        s += m;
      }
    }
    return s;
  }
}
```

[三维形体的表面积](https://leetcode-cn.com/problems/surface-area-of-3d-shapes/)

### 单词压缩编码

思路:

匹配后缀: 只有小单词为大单词的**后缀**，才能省略

1. 循环数组，求得每个元素的后缀
2. 如果当前元素匹配后缀，则可以省略

**注意事项**: 小字符串在前，大字符串在后，此时需要注意匹配

```
function minimumLengthEncoding(words) {
    var count = 0;
    var wordsSet = new Set(words);
    wordsSet.forEach(function (word) {
        for (var i = 1; i < word.length; i++) {
            var s = word.slice(i);
            if (wordsSet.has(s)) {
                wordsSet["delete"](s);
            }
        }
    });
    wordsSet.forEach(function (word) {
        count = count + word.length + 1;
    });
    return count;
}
```

[单词压缩编码](https://leetcode-cn.com/problems/short-encoding-of-words/
