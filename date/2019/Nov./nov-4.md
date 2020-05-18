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

```
function findPattern(str: string, p: string): boolean {
  const pLen = p.length;

  for (let i = 0; i < str.length; i++) {
    let j = 0;
    for (; j < pLen; j++) {
      if (str[ j + i ] !== p[ j ]) {
        break;
      }
    }

    if (pLen === j) {
      return true;
    }
  }
  return false;
}
```

**KMP**

关键思路：

1. 计算 next 数组，（计算最长相等的前后缀）
2. 暴力法中，i 是按照 i++来递增，而 KMP 中，按照 i = i+ max(1,(j+1)-next[j])`递归公式为 i = max(1,已匹配字符串长度 - next[最后一个匹配字符串索引])`

**计算 next 数组**

举个栗子：

```
模式：ABCABD
索引        字符串        前缀                     后缀                       最大匹配长度
0           A            []                       []                          0
1           AB           [A]                      [B]                         0
2           ABC          [A,AB]                   [BC,C]                      0
3           ABCA         [A,AB,ABC]               [BCA,CA,A]                  1
4           ABCAB        [A,AB,ABC,ABCA]          [BCAB,CAB,AB,B]             2
5           ABCABD       [A,AB,ABC,ABCA,ABCAD]    [BCABD,CABD,ABD,BD,D]       0

则 next = [0,0,0,1,2,0]

前缀特点，每个前缀必含有首字符 p[0]，后缀必含有尾字符p[k]，则 next 数组就是寻找首字符的位置
  使用动态规划
    dp[k] 推演公式
    dp[k-1] > 0 && p[ dp[k-1] ] === p[k] ：dp[k] = dp[k-1] +1; // 思路是，从首字符开始，按照索引匹配
    否则：dp[k] = p[k] === p[0] ? 1:0;

function findNext(p: string) {
  const dp: number[] = [ 0 ];

  for (let i = 1; i < p.length; i++) {
    if (dp[ i - 1 ] > 0 && p[ i ] === p[ dp[ i - 1 ] ]) {
      dp[ i ] = dp[ i - 1 ] + 1;
    }else{
      dp[ i ] = Number(p[ i ] === p[ 0 ]);
    }
  }
  return dp;
}
```

**KMP i 移动规则**

```
字符串：BBC ABCABE ABCDABCABDE，模式：ABCABD

对齐：
  BBC ABCABE ABCDABCABDE
  ABCABD
第1次匹配：i=0,j=0时不匹配，则i移动 max(1,j-next[j]=0) = 1

  BBC ABCABE ABCDABCABDE
   ABCABD
第2次匹配：i=1,j=0时不匹配，则i移动 max(1,j-next[j]=0) = 1

...

  BBC ABCABE ABCDABCABDE
      ABCABD
第5次匹配，i=4,j=0起始，一直到i=9,j=5时匹配错误，则i移动max(1,5-next[4]) = 3

  BBC ABCABE ABCDABCABDE
         ABCABD

第6次匹配，i=7,j=0起始，一直到i=9,j=2时匹配错误，则i移动max(1,2-next[1]) = 2

  BBC ABCABE ABCDABCABDE
           ABCABD

第7次匹配，i=9,j=0时不匹配，则i移动max(1,0-next[-1]) = 1

...

  BBC ABCABE ABCDABCABDE
             ABCABD

第9次匹配，i=11，j=0起始，一直到i=14,j=3匹配错误，则i移动max(1,3-next[2]) = 3

  BBC ABCABE ABCDABCABDE
                ABCABD
第10次匹配，i=14，j=0匹配错误，则i移动max(1,0-next[-1]) = 1

  BBC ABCABE ABCDABCABDE
                 ABCABD
第10次匹配，i=15 匹配正确
```

```
function findPatternKMP(str: string, p: string): boolean {
  const next = findNext(p);
  const pLen = p.length;

  for (let i = 0; i < str.length;) {
    let j = 0;
    for (; j < pLen; j++) {
      if (str[ i + j ] !== p[ j ]) {
        i += Math.max(1, j - (next[ j-1 ] || 0));
        break;
      }
    }
    if (pLen === j) return true;
  }
  return false;
}
```

**BM**

思路：

1. 字符串对齐后，从后往前匹配
2. 移动规则分为两种，取其中最大值

   - 坏字符规则：当匹配到坏字符时<font color="red">坏字符 1 指字符串中未匹配的字符；坏字符 2 指模式中未匹配的字符</font>，
     `i移动位数 = 模式中坏字符2索引 - 模式中坏字符1最右侧出现的索引(不存在则为-1)`
   - 好匹配缀规则：当匹配到坏字符时，右侧已经存在匹配成功的字符串了，<font color="red">好字符后缀表示已匹配的字符串所有后缀，是个数组</font>
     `i的移动位数= 模式中好后缀位置 - 模式中好后缀上次出现的位置`

     **好后缀的位置规则规则：**

     1. 好后缀取索引，取最右侧位置索引：如`ABCEDF 匹配 CEDF，则[F,DF,EDF,CEDF] 全部以F索引为标准计算`
     2. 好后缀如果在模式中上次出现的位置不存在，则为-1 `CASE3`
     3. 如果存在多个好后缀，**除去最长的好后缀，其余好后缀必须以模式"起始字符开始匹配"`**，如`ABDABDA 匹配 DABDA [A,DA,BDA,ABDA,DABDA]，则此时为6-0=6`；最长好后缀匹配为 `CASE1`，最佳好后缀匹配为 `CASE2`

**i 的移动规则**

```
ABCBBABA 匹配 ABA

首先对齐，
  ABCBBABA
  ABA
- 第一次匹配，C 为坏字符1，A为坏字符2，则移动位数 i= (A的索引2)-(C的最右侧索引，由于未出现，则等于-1) => 2-(-1)=3;

  ABCBBABA
     ABA

- 第二次匹配,坏字符规则： B 为坏字符1,A为坏字符2，则移动位数 i= 0-1 = -1
- 第二次匹配，好后缀规则：BA 为好后缀，则后缀为[BA,A]，
  BA后缀：则移动位数 i = 1 - (-1) = 2;
  A 后缀：则移动位数 i = 2 - 0 = 2;

  则 max(-1,2,2) = 2;

  ABCBBABA
       ABA
- 第三次匹配成功
```

```
// 计算第i个最大公约后缀
function suffix(p: string): number[] {
  const suffix: number[] = [];
  const pLen: number = p.length;
  suffix[ pLen - 1 ] = pLen;

  for (let i = pLen - 2; i >= 0; i--) {
    let j = i;
    for (; j >= 0; j--) {
      if (p[ j ] !== p[ pLen - 1 - (i - j) ]) {
        break;
      }
    }
    suffix[ i ] = i - j;
  }
  return suffix;
}

// 计算匹配模式，字符最右侧的位置，加上匹配前缀的位置（最右侧的位置）
function findCharsIndex(p: string): Map<string, number> {
  const pLen = p.length;
  const pMap = new Map<string, number>();

  for (let i = 0; i < p.length; i++) {
    const fChar = p.slice(0, i + 1);
    pMap.set(p[ i ], i);
    pMap.set(fChar, i);
  }
  return pMap;
}

// 寻找最合适的匹配后缀 suffix：第i个最大匹配后缀长度，p：模式，j：坏字符位置
function findBackChar(suffix: number[], pMap: Map<string, number>, p: string, j: number): number {
  const pLen = p.length;
  const goodCharLen = (pLen - 1 - j); // 好后缀长度

  // 存在好后缀时
  if (goodCharLen > 0) {
    let goodCharPrevIndex = j; // 以suffix来计算，suffix表示第i开始的公共后缀，如果suffix[i]是大于等于好后缀的长度时，则表明能找到该好后缀，否则无法找到

    // 满足最大后缀 CASE1
    while (goodCharPrevIndex >= 0) {
      if (suffix[ goodCharPrevIndex ] >= goodCharLen) {
        return pLen - goodCharPrevIndex;
      }
      goodCharPrevIndex--;
    }

    // 如果不存在最大后缀，则寻找合适后缀，且合适后缀必须从模式首位开始匹配，即模式的前缀匹配 CASE2
    for (let i = j + 2; i < pLen - 1; i++) {
      if (pMap.has(p.slice(i, pLen))) {
        return pLen - pMap.get(p.slice(i, pLen));
      }
    }

    // 如果寻找不到最佳后缀，则匹配最后一位字符是否与首字符相等
    return p[ pLen - 1 ] === p[ 0 ] ? pLen - 1 : pLen; // CASE2 + CASE3
  }

  return 0;
}


function findPByBM(str: string, p: string) {
  const pLen = p.length;
  const suffixList: number[] = suffix(p);
  const pMap: Map<string, number> = findCharsIndex(p);

  // 对齐字符串，从模式右侧开始查找
  for (let i = pLen - 1; i < str.length;) {
    let j = pLen - 1;
    for (; j >= 0; j--) {
      // 匹配失败
      const n = i - pLen + 1 + j;
      if (p[ j ] !== str[ n ]) {
        // 以坏字符跳转
        const badCharStep = j - (pMap.has(str[ n ]) ? pMap.get(str[ n ]) : -1);
        // 以好后缀跳转
        const goodCharStep = findBackChar(suffixList, pMap, p, j);

        i += Math.max(badCharStep, goodCharStep);
        break;
      }
    }

    // 自然中断，则匹配成功
    if (j === -1) return true;
  }
  return false;
}
```

**Sunday**

总体和 BM 类似，但从前往后匹配，且省略`好后缀匹配`
思路：

1. 从前往后匹配
2. 移动位数：`下一位字符`，指字符串中，第(i+pLen)个字符
   - 如果下一位字符没有在模式出现，则`移动位数=模式长度+1`
   - 如果在模式出现，则`移动位数=模式中该字符最右侧到末尾的距离+1`

```
ABCBBABA 匹配 ABA

首先对齐，
  ABCBBABA
  ABA
第1次匹配：从i=0,j=0，到i=2,j=2时失败，则nextChar为B(C后面一位)，B在模式存在，且到末尾为2位，则移动2位

  ABCBBCABA
    ABA
第2次匹配：i=2,j=0失败，下一位为C，在模式不存在，则移动pLen+1=4，则移动4位
  ABCBBCABA
        ABA

匹配成功
```

```
function findPBySunday(str: string, p: string) {
  const pLen = p.length;
  const pMap = new Map<string, number>();

  for (let i = 0; i < pLen; i++) {
    pMap.set(p[ i ], i);
  }

  for (let i = 0; i < str.length;) {
    let j = 0;
    for (; j < pLen; j++) {
      if (str[ i + j ] !== p[ j ]) {
        const nextChar = str[ i + pLen ];

        if (pMap.has(nextChar)) {
          i += pLen - pMap.get(nextChar);
        } else {
          i += (pLen + 1);
        }
        break;
      }
    }
    if (j === pLen) return true;
  }
  return false;
}
```

## LeetCode

### 递增三元序列

[递增三元序列](https://leetcode-cn.com/problems/increasing-triplet-subsequence/)

思路：

1. 寻找三个数，让 n1<n2<n3 成立即可
2. 不断寻找最小的 n1，n2>n1 即可；此时主要目的就是寻找 n3
   - 下一个数 > n2，等式成立，立即返回
   - 下一个数 < n1，则 替代 n1；否则替代 n2

```
function increasingTriplet(num: number[]) {
  let n1 = num[ 0 ];
  let n2 = null;

  for (let i = 1; i < num.length; i++) {
    if (n2 !== null && num[ i ] > n2) return true;

    if (num[ i ] > n1) {
      n2 = num[ i ];
    } else {
      n1 = num[ i ];
    }
  }
  return false;
}

```
