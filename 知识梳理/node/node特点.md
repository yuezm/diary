  # Node.js

  ## 什么是 node.js

  node.js 是一个 脱离浏览器执行环境的 javascript runtime。说的再明确些，node.js 是一个服务器程序，旨在提供一个基于事件驱动、异步 I/O 的高性能 web 服务器。

  node.js 最初只是单纯的开发一个高性能的 web 服务器，但最后变成了一个构建网络应用的基础框架，例如：服务器、客户端、命令行工具...，非常容易扩展来达成构建大型网络应用的目的，每个 node 进程都构成这个网络应用的一个节点，这也是它名字的含义。

  ## Node.js 架构

  ![node.jpeg](https://public.keven.work/node%E4%B8%80%E8%A7%88.jpeg)

  1. Node Standard Library: 以 javascript 呈现的 node modules，平时用的最多的模块
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
  uv_fs_open()、uv_fs_read(); // libuv 根据各个平台，调用不同的API
  ```

  ### V8

  ![v8.png](https://public.keven.work/v8.png)

  v8 是 node 的核心之一，v8 主要负责 javascript 的代码解析及运行。

  1. JIT：AST => 机器码，字节码 => 机器码
  2. GC：v8 借鉴 JVM 的精确垃圾回收管理，使用 Scavenge、Mark-Sweep 和 Mark-Compact 算法
  3. 内联缓存：使用内联缓存来提高属性的访问效率，例如访问 `this.name`，如果不存在内联缓存，则每次取 name 时都需要对 hash 表进行一次寻址，而加入内联缓存可以立刻知道该属性偏移量，不需要再次计算寻址
  4. 隐藏类：由于 Javascript 是一门动态语言，开发者可以任意在对象上增加、删除属性。如下所示

    ```javascript
    var obj = {};

    // 为所欲为
    obj.name = 1;
    delete obj.name;
    ```

    隐藏类就是对此对象的黑科技包装，，**所有相同的属性的对象归于同一个隐藏类**

  **隐藏类+内联缓存**：同一个隐藏类的对象，可以使用同一套内联缓存来寻址

  ### Libuv

  ![libuv](https://public.keven.work/libuv.png)

  libuv 是 node 的核心之一，libuv 提供 1.事件循环；2.跨平台；3.异步 I/O；4.对 I/O 抽象，将 I/O 抽象为句柄或流

  ```javascript
  // 流抽象 Buffer（Uint8Array）
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

  // 句柄抽象 句柄是一个非负整数，用标识一个对象、资源或一个服务。
  const fs = require('fs');
  const fd = fs.openSync('./package.json', 'r');
  console.log(fd); // 此时fd代表的数字标识了node.js文件资源，可以通过fs.read读取
  ```

  ## Node.js 特性

  ### 异步 I/O

  异步 I/O：计算机操作系统对输入输出的一种处理方式：发起 I/O 请求的线程**不等 I/O 操作完成，I/O 结果用其他方式通知发起 I/O 请求的程序**

  **优势**：公平有效的利用计算机资源，提高服务器的响应能力
  **劣势**：异步思维，不符合编程人员直觉

  #### 非阻塞 I/O

  异步 I/O 和非阻塞 I/O 都是为了**并行 I/O** 的目的，提高服务器的响应能力。但 _异步/同步 I/O_ 和 _阻塞/非阻塞 I/O_ 还是有区别的

  在操作系统层面，只存在阻塞和非阻塞 I/O：

  阻塞 I/O：调用阻塞 I/O 时，程序需要等待 I/O 完成才返回结果
  非阻塞 I/O：进行非阻塞调用时，程序会立即返回，操作系统将 I/O 抽象为文件，通过文件描述进行管理，程序可以通过文件描述符去实现文件的读写。

  对于阻塞/非阻塞 I/O，举个例子，泽泽点餐，将餐厅抽象为计算机，将老板（华东）抽象为操作系统，将点餐抽象为 I/O，input=菜名，output=菜

  ```text
  泽泽（code）：大吼一声，老板，点一份小炒肉（input => "小炒肉"）
  华东（os）：大吼一声，告诉后厨，小炒肉一份

      此时，问题来了，泽泽可以选择如下两种取餐方式

  1. 泽泽站在华东处等着小炒肉做好，做好再端走（output => "小炒肉"），在这过程中，后面的人跟着泽泽一起排队傻等 --- 阻塞I/O
  2. 华东给泽泽一个小牌 1号，让泽泽站旁边等着，一会凭着号牌取餐（output => "小炒肉"） --- 非阻塞I/O

      如果是第2种，那么问题又来了，**一会** 凭着号牌取餐，这个一会是多久？一分钟？一小时？一天？，所以泽泽又有如下几种方式来看小炒肉做好没有

  轮询：
  1. 隔一分钟泽泽去问下华东，小炒肉做好了没有，第一次讯问，第二次询问...直到泽泽拿到我小炒肉回座位，才停止询问（output => "小炒肉"）--- 较为原始的轮询技术 read
  2. 泽泽在座位盯着取餐口屏幕，等着屏幕显示1号可以取餐了（参考肯德基取餐），然后泽泽去拿到我小炒肉回座位（output => "小炒肉"）--- 通过文件描述符事件或状态来判断 select、poll
  3. 泽泽在座位耍手机、睡觉...等着，等着华东吼一声1号来取餐了，然后泽泽去华东处拿到小炒肉，然后回到座位（output => "小炒肉"）--- I/O事件通知机制  epoll、kqueue
  ```

  **异步**：强调了调用过程，表示调用结果不会直接返回（_或者立即返回了，但是没得结果_），而是不知道什么时候会返回。
  **非阻塞**：调用时等待结果的状态，表示不会阻塞后续代码执行

  #### Node.js 异步 I/O

  libuv 对 I/O 分为 Network I/O 和 File I/O、User Code...（参照 libuv 架构）

  1. 对于 NetWork I/O：由操作系统自身实现
  2. 对于 File I/O、User Code...：让一个线程负责计算处理（主线程），让其他线程进行阻塞 I/O 或者非阻塞 I/O 加轮询技术来获取数据（I/O 线程池）。libuv 线程池默认是 4 个，可以通过 _UV_THREADPOOL_SIZE_ 环境变量来设置，
    但不能超过最大 `MAX_THREADPOOL_SIZE = 1024` 个

  ![](https://public.keven.work/node%E5%BC%82%E6%AD%A5.png)

  ```cpp
  // 举例 fs.open
  int uv_fs_open(uv_loop_t *loop, uv_fs_t *req, const char *path, int flags, int mode, uv_fs_cb cb) {
    INIT(OPEN); // 请求对象赋值，req 创建于 node_file.cc
    PATH; // 路径处理
    req->flags = flags;
    req->mode = mode;
    POST;
  }
  ```

  对于 libuv 线程池，举个栗子，泽泽做足疗，将足疗厅老板抽象为主线程，将技师（华东）抽象为 I/O 线程（池），将做足疗抽象为 I/O

  ```text
  泽泽（code）：来到足疗店，吼一声要做足疗。（input=>"做足疗"）
  老板（main）：开始下单（创建请求对象），"姓名：泽泽，性别：男，项目：足疗"（设置请求对象参数），"预计收费500元"（设置请求对象回调函数，缴费）
  老板（main）：下单完毕后，将上面的单子给泽泽，并将泽泽领到vip等候区（将请求对象推入线程池）
  华东（worker）：华东此时空闲，于是就去等候区把泽泽领到1号房间（线程池存在可用线程）
  华东（worker）：华东开始做足疗（执行I/O）
  华东（worker）：足疗完毕，并在小单子上写上，基础收费500，额外收费500，最终收费1000（执行I/O完毕，并将执行结果放入请求对象）

  ...省略事件循环...

  老板（main）：得知泽泽做完足疗，确认结果后，收费1000（主线程执行回调）（output=>"足疗服务完成"）
  ...
  ```

  ### 事件驱动（函数回调）

  大部分 node 模块，比如上面演示的 _net_ 都是基于 `EventEmitter` 模块实现的，所以它们拥有触发和监听事件的能力

  ### 单线程

  **node 主线程是单线程，而非 node 是单线程**！！！

  **缺点**：

  1. 无法利用 CPU 多核优势
  2. 错误会引起整个程序退出
  3. CPU 密集型任务会降低性能

  child_process，worker_threads，master-worker 工作模式

  **优点**：

  1. 不用在意多线程的状态同步问题
  2. 没有线程切换的开销

  ### 跨平台

  ![node跨平台](https://public.keven.work/node%E8%B7%A8%E5%B9%B3%E5%8F%B0.png)
