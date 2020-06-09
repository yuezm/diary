# Node.js

## 什么是 node.js?

node.js 是一个 脱离浏览器执行环境的 javascript runtime。说的再明确些，node.js 是一个服务器程序，旨在提供一个基于事件驱动、异步 I/O 的高性能 web 服务器。

node.js 最初只是单纯的开发一个 web 服务器，但最后变成了一个构建网络应用的基础框架，例如：服务器、客户端、命令行工具...，非常容易扩展来达成构建大型网络应用的目的，每个 node 进程都构成这个网络应用的一个节点，这也是它名字的含义。

## Node.js 架构

![node.jpeg](/static/images/node.jpeg)

1. Node Standard Library: 以 javascript 呈现的 node module，平时用的最多的模块
2. Node Bindings: Node Standard Library 和底层 c++ 代码沟通的桥梁，隐藏底层实现
3. 底层依赖
   - v8: javascript 引擎，解析执行 javascript 代码
   - libuv: 1. 事件循环；2. 平台兼容；3. 异步 I/O；4. 对 I/O 抽象为流或句柄
   - c-ares: DNS 工具
   - http_parser: 解析 http 报文
   - openssl: 提供 ssl
   - zlib: 压缩
   - ...

```javascript
// node STL
fs.open()、fs.read();

// node bindings
process.binding('fs').read(); // lib/fs.js
static void Read(const FunctionCallbackInfo<Value>& args) {} // src/node_file.cc

// 底层
uv_fs_read(); // libuv 根据各个平台，调用不同的API
```

#### V8

v8 是 node 的核心之一，v8 主要负责 javascript 的代码解析及运行。

![v8.png](/static/images/v8.png)

#### Libuv

libuv 是 node 的核心之一，libuv 提供 事件循环、跨平台、异步I/O, 对 I/O 抽象，将 I/O 抽象为句柄或流

```javascript
// 流抽象
// class Socket extends stream.Duplex {}

const net = require('net');
const server = net.createServer().listen(9001);

server.on('connection', (socket) => {
  console.log(
    `有一个新的 tcp 连接进来了，连接ip为：${socket.remoteAddress} 连接端口为：{socket.remotePort}`
  );

  socket.on('data', (data) => {
    console.log(data.toString());
  });
});

// 句柄抽象
const fs = require('fs');
const fd = fs.openSync('./node.js', 'r');
console.log(fd);
```

![libuv](/static/images/libuv.png)

## Node.js 四大特性

### 异步I/O

异步I/O：计算机操作系统对输入输出的一种处理方式：发起 I/O 请求的线程**不等 I/O 操作完成，就继续执行随后的代码，I/O 结果用其他方式通知发起 I/O 请求的程序**

**优势**：公平有效的利用计算机资源，提高服务器的响应能力
**劣势**：异步思维，不符合编程人员直觉

#### 非阻塞 I/O

异步I/O 和非阻塞I/O 都是为了**并行 I/O** 的目的，提高服务器的响应能力。但 *异步/同步I/O* 和 *阻塞/非阻塞I/O* 还是有区别的

在操作系统层面，只存在阻塞和非阻塞I/O：

阻塞I/O：调用阻塞I/O时，程序需要等待I/O完成才返回结果
非阻塞I/O：进行非阻塞调用时，程序会立即返回，操作系统将I/O抽象为文件，通过文件描述进行管理，程序可以通过文件描述符去实现文件的读写。

对于阻塞/非阻塞I/O，举个例子，泽泽点餐，将餐厅抽象为计算机，将老板（华东）抽象为操作系统，将点餐抽象为I/O，input=菜名，output=菜

```text
泽泽：input => 点一份 "小炒肉"
华东老板：大吼一声，告诉后厨，小炒肉一份

    此时，问题来了，泽泽可以选择如下两种取餐方式

1. 泽泽站在收银处等着小炒肉做好，做好再端走（output => "小炒肉"），在这过程中，后面的人跟着泽泽一起排队傻等 --- 阻塞I/O
2. 华东老板给泽泽一个小牌 1号，让泽泽站旁边等着，一会凭着号牌取餐（output => "小炒肉"） --- 非阻塞I/O

    如果是第2种，那么问题又来了，**一会** 凭着号牌取餐，这个一会是多久？一分钟？一小时？一天？，所以泽泽又有如下几种方式来看小炒肉做好没有

1. 隔一分钟泽泽去问下华东老板，小炒肉做好了没有，第一次讯问，第二次询问...直到泽泽拿到我小炒肉回座位，才停止询问（output => "小炒肉"）--- 较为原始的轮询技术 read
2. 泽泽在座位盯着取餐口屏幕，等着屏幕显示1号可以取餐了（参考肯德基取餐），然后泽泽去拿到我小炒肉回座位（output => "小炒肉"）--- 通过文件描述符事件或状态来判断 select、poll
3. 泽泽在座位耍手机、睡觉...等着，等着华东老板吼一声1号来取餐了，然后泽泽去华东老板处拿到小炒肉，然后回到座位（output => "小炒肉"）--- I/O事件通知机制  epoll、kqueue
```

#### Node.js 异步 I/O

libuv 对I/O分为 Network I/O 和 File I/O、User Code...（参照libuv架构）

1. 对于 NetWork I/O：由操作系统自身实现
2. 对于 File I/O、User Code...：让一个线程负责计算处理（主线程），让其他线程进行阻塞I/O或者非阻塞I/O加轮询技术来获取数据（I/O线程池）。

对于libuv线程池，举个栗子，泽泽做足疗，将足疗厅老板抽象为主线程，将技师（华东）抽象为I/O线程，将做足疗抽象为I/O，input => 足疗需求，output=> 足疗服务


```text
```


### 事件驱动（函数回调）

### 单线程

### 跨平台

## Node.js 包管理器

npm（Node Package Manager）：npm 是一个 javascript 包管理工具，且是 node 的默认包管理工具，可以使用 npm 对 javascript 包（node 包） 进行管理。

npm 主要由三部分组成：

- 网站：是开发者查找包（package）、设置参数以及管理 npm 使用体验的主要途径，[https://www.npmjs.com/](https://www.npmjs.com/)
- 注册表（registry）：是一个巨大的数据库，保存了每个包（package）的信息，常用的有 [https://registry.npmjs.org/](https://registry.npmjs.org/)，[http://r.cnpmjs.org/](http://r.cnpmjs.org/)，[https://registry.npm.taobao.org/](https://registry.npm.taobao.org/)，[https://registry.yarnpkg.com/](https://registry.yarnpkg.com/)
- CLI 通过命令行或终端运行

#### npm 应用场景

## 为什么使用 Node（Node 的应用场景）
