# 10 月第 4 周

## Javascript

## C++

## 网络

### 分组

### 攻击

蠕虫、病毒、嗅探

嗅探：在网络传输重建截取信息，由于不会像分组插入任何数据，很不容易被发现

病毒：

1. 需要传播途径
2. 需要手动触发
3. 可以自我攻击主机程序

蠕虫：

1. 需要传播途径
2. 无序手动触发
3. 可以自我攻击并且可以自我复制和自我传播

### 应用层协议

进程传递特点：每个进程都是通过 socket 传递信息，socket 是进程程进行网络传输的大门

应用层 => 套接字 => 传输层

传输协议特性:

1. 可靠性
2. 吞吐量
3. 定时
4. 安全性

应用层协议特性：

1. 定义应用层接收和发送报文，及发送和接收报文后的动作
2. 定义报文的语法，类型
3. 定义报文的语义

#### HTTP

特点：

1. 无状态连接
2. 非持续性
3. 80、443 端口

组成：

1. Request Line：GET / HTTP/1.1
2. Header Line:：Content-Type: application/json
3. Body Entity

#### HTTP 协议

1.建立在 TCP 协议之上 2.无状态协议

**持续连接和非持续连接**

非持续连接：一个 HTTP 建立一个 TCP 连接，在 HTTP 返回后会将该 TCP 连接断开

1. 会多余浪费 TCP 握手时间，即一个 RTT（三次握手，前两次握手为一个 RTT，第三次握手会随着 HTTP 请求一起发送）
2. 会增加客户端和服务器的 TCP 缓冲区的维护成本和 TCP 变量的维护成本

持续连接：多个 HTTP 公用一个 TCP，如果再一定时间内没有 HTTP 使用该连接，也会断开

RTT: 往返时间，一个短分组由客户端出发，到服务器再返回客户端的时间（包含了链路延时）

`HTTP 请求的时间` = 2RTT + 响应文件传输时间（简单的计算）

#### FTP

作用：文件传输

协议特点：

1. 两个 TCP 连接，一个 TCP 连接负责用户口令及指令的传递，另一个负责文件传输
2. 需要存储连接状态

```
FTP 使用两个TCP: TCP1控制连接,传输用户的口令及命令；TCP2传输连接，传输文件

FTP 客户端 => FTP 服务端：创建 TCP 连接，传输用户口令，及命令

FTP 服务端 => FTP 客户端：接收到传输文件命令后，创建 TCP 连接来传输文件

FTP：需要存储用户的状态，即他和 HTTP 不同
```

#### SMTP

作用：负责邮件传输

特点：

1. TCP 连接，25 端口
2. 邮件服务器存在重试机制，一般几天后还无法发送邮件，则邮件通知发送方发送失败
3. 现代一般都是用邮件代理
4. 属于推送协议，且只存在请求报文（HTTP 属于拉取协议，存在请求和响应报文）

邮件系统一般由：用户代理，邮件服务器，SMTP 协议组成（使用 25 端口）

发送邮件的方式：

```
A 的邮件服务器 => B 的邮件服务器

A 邮件服务器`直接`和 B 邮件服务器建立 `TCP 连接`，`握手并确认身份`后开始传输文件，如果无法连接 B 服务器，则存在`重试机制`，直到几天后还是无法推送到 B 邮件服务器，则取消发送，并通知 A 邮件服务器
```

但是现在一般都是使用用户代理，而不是使用邮件服务器

```
用户代理 A => A 邮件服务器 => SMTP => B 邮件服务器 => B 的代理（桌面邮件程序）
```

与 HTTP 异同

相同：

1. 都是 TCP 连接

不同：

1. HTTP 属于`拉协议`，SMTP 属于`推协议`
2. HTTP 属于字节流，STMP 要转换为 ASCII 码
3. HTTP 存在两个报文，请求和响应，而 STMP 只有推送报文

#### DNS

作用：负责查询域名，一个大型分布式数据库系统

服务器特点：

1. 分布式部署

- 解决单点部署服务器故障率较大的问题
- 解决服务器负载较大，导致请求速度问题
- 解决链路太远，导致请求速度问题
- 解决维护成本过高的问题

2. 层次分布

- 根域名服务器
- 顶级域名服务器
- 权威域名服务器（二级域名服务器，三级域名服务器等）
- 本地域名服务器，`不属于层次分布`，但是作用十分重要

流程

1. 客户端 => 本地域名服务器
2. 本地域名服务器 => 根域名服务器；返回顶级域名服务器的 IP 地址
3. 本地域名服务器 => 顶级域名服务器；返回权威域名服务器的 IP 地址
4. 本地域名服务器 => 权威域名服务器；返回 目标域名的 IP 地址
5. 本地域名服务器 => 返回客户端

DNS 缓存：域名服务器都可以缓存其他域名服务器返回的信息，减少重复查询，但缓存是有时效的，过一段时间后会丢弃

报文

1. TTL
2. Type

   - A：A（主机） NAME 主机名，VALUE IP；A 为 ipv4,AAAA 为 ipv6
   - NS：NS（域）NAME 域名，VALUE DNS 权威服务器域名
   - CNAME：CNAME（主机别名）NAME 主机别名，VALUE 主机规范名称
   - MX：MX（邮寄服务器别名）NAME 邮件服务器别名，VALUE 邮件服务器规范名称

3. Name
4. Value

nslookup 工具

## 编译（编译学不动，先做做小 Demo，后续再学习）

### 语法解析

上下文无关语法

## 算法

### 图的遍历

深度优先、广度优先

### 字符串排序

键索引计数法、低位优先排序、高位有限排序

#### 键索引计数法

思路：通过计算每个**键位的起始位置**来排序

originData：输入数据：{key: '键值'键值, value：元素值}
count：存储各个键的频率 / 起始位置
targetData：排序好的数据

- 计算各个键的频率

```
for (const item of originData) {
  const k = item.key + 1;
  if (count[ k ] === undefined) count[ k ] = 0;
  count[ k ] += 1;
}
// 描述 item 的键频率，但是是以 key + 1 存储，这样方便后续计算时，计算键位的起始位置
```

- 计算各个键的起始位置

```
for (let i = 0; i < R; i++) {
  count[ i + 1 ] = (count[ i + 1 ] || 0) + count[ i ] || 0;
}
```

- 将数按照键放在数组内

```
for(let i =0;i < originData.length ;i++ ){
  targetData[ count [ originData[i].key ] ++ ] = originData[i].value; // 将当前值放入 count 内 标记的 key值起始位置，并将起始位置+1
}
```

#### 低位有限排序

主要是解决**同长度**字符串排序

```javascript
function sort(strNums, n) {
  // 从后往前排 !!!!
  const R = 257;
  let storeArr = [];

  for (let c = n - 1; c >= 0; c--) {
    const count = [];

    // 计算频率
    for (const item of originData) {
      const code = item[c].charCodeAt(0) + 1;
      if (count[code] === undefined) count[code] = 0;
      count[code] += 1;
    }

    // 计算起始位置
    for (let i = 0; i < R; i++) {
      count[i + 1] = (count[i + 1] || 0) + (count[i] || 0);
    }

    // 根据起始位置赋值
    for (let i = 0; i < strNums.length; i++) {
      const t = strNums[i];
      storeArr[count[t[c].charCodeAt(0)]++] = t;
    }

    strNums = storeArr;
    storeArr = [];
  }
  return strNums;
}
```

#### 高位优先

如果字符串长度不同，排序时，就不能从左向右排了

1. 先将第一位的排序
2. 然后将第一位相同的再分组排序
3. 最后合并到一块

## Leetcode

### 验证二叉搜索树

思路：验证

```
左节点 < 父节点
右节点 > 父节点

但要特点注意判断（右边必须必比根节点大，左边必须必根节点小）
```

[验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

### 对称二叉树

思路 1：验证每一个节点，它的左节点和右节点是否相等（递归），例如 左 - 右 => 左-左 - 右-右 | 左-右 - 右-左
思路 2：验证每一层树的节点是否为回文数组

[对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/submissions/)

### 有序数组转平衡二叉搜索树

思路：二分法，将数组从中间分开，中间为父节点，左边右边分别为左树和右树（递归）

[有序数组转平衡二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/solution/)

### 合并两个有序数组

思路：归并排序

[合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/solution/he-bing-liang-ge-you-xu-shu-zu-by-leetcode/)
