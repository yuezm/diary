# 11 月第 4 周

## Javascript

### PWA

**渐进式 WEB APP**，结合了 Web APP 和 Native APP 特点

特点：

1. 渐进式，可以根据需要增删功能
2. 离线访问
3. 可安装
4. 可推送消息
5. 安全，仅支持 https(localhost 除外)

### Service Worker

#### 生命周期

1. install 下载

   - 下载安装中
   - 下载安装后：此时触发 install 事件

2. activate

   - 激活中
   - 激活后：此时触发 activate 事件

3. delete：废弃，废弃当前 Service Worker，可以使用手动触发，this.clients.claim()卸载旧的 Service Worker;

#### 方法

- register：注册 Service Worker
- skipWaiting：跳过等待，在 install 事件中使用

**event 方法**

- waitUntil：等待下载完成，在 install 事件中使用
- respondWith：在 fetch 事件使用，表示此次请求受程序控制

#### 事件

- install：下载完成
- activate：激活完成
- fetch：发起请求

- message：PostMessage API
- push：消息推送
- sync：后台同步

### Fetch API

**新的异步请求 API，和 AJAX 一样，是为了实现异步调用**

区别：

1. fetch：
   - 以浏览实现(不知道如何实现)；
   - 默认不传输、接收 Cookie（除非设置 credentials）；
   - 以 Promise 实现；
   - `默认不返回错误，即使返回**4xx、5xx**也是扭转状态为 FullFilled`；
   - 发送后无法终止（可使用黑科技）
2. AJAX：
   - 客户端以 XMLHttpRequest 对象实现；
   - 默认传输 Cookie；
   - 以事件+回调函数形式实现；
   - 会抛出错误；
   - 发送后可以调用 abort 终止

### Cache API

缓存 API，不止可以在 Service Worker 内使用

#### 方法

**caches:**

- open(cacheName)：开辟缓存空间
- match(request)：匹配 key，返回第一个匹配值
- matchAll(requst)：匹配 key，返回匹配的数组

**cache(开辟空间的缓存对象)**

- put(request,response)：key-value 存储
- add(request)：等同于 fetch 请求完毕，并 put 进入缓存
- addAll(requests);
- delete(request)
- keys(request,options)

**DEMO**

```

if (window.navigator.serviceWorker) {
  window.addEventListener('load', () => {
    window.navigator.serviceWorker.register('./sw.js', { scope: '/pwa-demo/' })
      .then(reg => {
        console.log('注册成功', reg);
      })
      .catch(reason => {
        console.log(reason);
      });
  });
}


// sw.js
const self = this;

self.addEventListener('install', () => {
  self.skipWaiting();
});

self.addEventListener('activate', ev => {
  self.clients.claim();
});

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(response => {
      if (response) return response;

      const request = event.request.clone();
      return fetch(request).then(httpRes => {
        const response = httpRes.clone();

        if (httpRes.status === 200) {
          caches.open('pwa-demo').then(cache => {
            cache.put(event.request, response).then(() => {
              console.log('添加缓存成功');
            }).catch(() => {
              console.log('添加缓存失败')
            });
          })
        }

        return httpRes;
      });
    })
  )
});

```

## Typescript

## Node

### npm install

**npm install 流程**

整理大概流程，后续深入了解源码

文件存储地址：**node_modules**，**npm-cache**，**Register**

1. 发出 npm install 命令
2. 构建本地依赖树
3. 构建依赖树，按照 package.json 和 package.lock.json 构建
4. 对比两棵树，及包的版本，找出缺失的包
5. 在 npm cache 中寻找包，如果未过期，则直接使用 npm cache 包
6. 如果已过期，则向 Register 发起请求
7. 将 Register 包下载到 npm cache 中，并解压到 node_modules 目录中
8. 目录扁平化
9. 执行生命周期钩子
10. 更新 package.json 及 package.lock.json 文件

**npm install github 包**

等到看源码，研究下载机制

**依赖 git 包过大的问题**

npm package.json 申明包地址为 git 地址，但 git 地址过大，无法下载，按照`git clone --depth=1`可以指明拉取最近一次 commit，从而减少包的大小，在 npm 中如下实现

npm 中存在缓存，npm cache，寻找到 npm cache 缓存的位置，修改其中的克隆代码，则可以实现

## C++

## 网络

### 链路层

**节点**：链路层中，每个网络设施称为节点
**接口**：与网络节点连接处称为接口
**子网、子网掩码**：与路由每个接口相连接的端系统之间互相为**独立小岛**，称为子网；子网掩码：10.0.1.0/24，表示前 24 位确定一个子网

### 链路层地址

**MAC 地址**：表示端系统网卡的物理地址，每个网卡的地址是唯一的

**链路层寻址 ARP 及 DNS 区别**

链路层寻址：将 IP 地址转换为 MAC 地址，通过 **ARP** 实现

**ARP（地址解析协议）**：将 IP 地址转换为 MAC 地址

DNS：将域名转为 IP 地址，且所有地址均可访问 DNS 服务器
ARP：将 IP 地址转为 MAC 地址，且只有同子网能访问

### 网络安全

**特点**

1. 报文机密性：报文加密，不能被第三方识别
2. 报文完整性
3. 端系统身份认真
4. 运行安全

#### 报文机密性：报文加密，不能被第三方识别

传输双方加密，当解密成本远大于数据价值时，则数据足够安全

##### 对称加密

对称加密：方和解密方使用同样的秘钥，如 AES,DES 加密方法

**凯撒加密**：原始的加密算法，首先得出 k，k 表示字母以后移 k 位表示，例如当 k=2 时：

```
c ==> a
d ==> b
e ==> c
```

现代加密方法：使用*密码块*

**分组加密**：将数据切割为`以n位大小的数据`，如 n=8 时，则 8 位为一个密码块，密码块之间互相独立；当 n=8 时，总共具有(z^8)!中可能性

##### 非对称加密

非对称加密：采用公钥，私钥方式的加密方法；加密方以公钥加密，解密方以私钥解密

通常公钥必须经过认真，即**CA**颁发的证书，当然也可以自己作为 CA 颁发证书

##### 会话加密

以非对称加密交换`秘钥`，后续以对称加密通信（HTTPS）

#### 报文完整性

发送端使用摘要算法，计算报文 hash 值，传输给接收端，再由接收端校验该 hash 是否正确

#### 端系统身份确定

通过加密口令，即`不重数`来标识身份

#### 运行安全

防火墙

**特点**：

1. 流量必须经过防火墙进入内网
2. 防火墙根据策略过滤流量
3. 防火墙本身必须具有安全性，不可被渗透

**分类**：

1. 传统分组过滤器：比如可以实现过滤外网访问 WEB（丢弃 TCP SYN=1，且无 ACK 的请求），防止路由被跟踪（丢弃 ICMP TTL 超时）
2. 状态分组过滤器：内部维护状态，例如进入 TCP SYN=1,ACK=1 的分组，但此时状态内未维护该 IP 的状态，则直接过滤
3. 应用子程序网关：对应用层分组过滤

大概可以分为两种:**入侵检测系统**和**入侵防止系统**

## 算法

### 字符串查找算法

在字符串 n 中，寻找是否存在匹配 字符串 p

**暴力法**

**KMP**

**BM**

**Sunday**

## LeetCode

### 递增三元序列

[递增三元序列](https://leetcode-cn.com/problems/increasing-triplet-subsequence/)
