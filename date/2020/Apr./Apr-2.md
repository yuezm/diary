## Javascript

### Array.prototype.fill

fill: 将数组填充为指定数据，**当 fill 参数为对象时，不会创建新对象，而是引用**

### fetch / AbortController

fetch: 异步请求，可以替代 Ajax

**fetch 和 Ajax 区别**

1. 兼容性: fetch 兼容性较差；Ajax 交融性较好
2. 是否可拦截: fetch 可以配合 Service Worker 进行请求和响应拦截；Ajax 无法拦截
3. 是否可终止: **fetch 原来不可终止**，但现在可以使用 AbortController 进行终止；Ajax 可终止
4. **响应状态**: fetch 不会拒绝 HTTP 错误码，例如 4xx、5xx 也不会 reject，除非**网络错误、阻止请求完成**才会 reject；Ajax 会对 4xx、5xx 状态码进入 reject
5. **发送 Cookie**: fetch 默认不会发送 Cookie；Ajax 默认发送 Cookie

#### AbortController

fetch 可以使用 AbortController 终止

```javascript
const abort = new AbortController();

fetch('url', { signal: abort.signal }).then().catch();

abort.abort(); // 终止，且使 fetch 进入 Rejected 状态
```

## 浏览器安全

### 浏览器沙箱

沙箱: 资源隔离模块，运行不可信任代码

浏览器大多采用多进程架构，每个进程挂了后不会影响其余进程。且 Render 进程运行于沙箱中，其通信依靠 IPC Channel，其中包含了安全检测

### 恶意网址拦截

浏览器维护**恶意网址黑名单**，在用户访问时发出报警。浏览器网址黑名单维护与服务器，由浏览器周期性获取

### CSP

CSP: 内容安全策略，由 HTTP 头部控制当前页面的安全策略，而一般的 XSS 无法控制 HTTP 头部，所以可以有效防止 XSS。

CSP 通过 Response 头部控制当前页面的安全策略，例如

```
X-Content-Security-Policy: allow self // 表示只信任当前域名，拒绝加载其余的一切资源，例如CSS、图片、音频...
```

**CSP 配置较为复杂**，在页面较多的情况下，维护成本过高，所以未能很好的推广

## HTTP

### 认证

1. 用户名、密码认证
2. 一次性认证（验证码）
3. 证书认证
4. 生物认证
5. IC 卡认证
6. ...

### HTTP 认证

#### HTTP Basic Auth

由 HTTP 头部控制的认证

```
// HTTP Response
HTTP/1.1 401 Authorization Required
WWW-Authenticate: Basic realm ="Input you username password"


// HTTP Request
Authorization: Basic xxx // 向服务端发送由用户名和密码的 base64 编码
```

由于是直接发送的 base64 编码，所以和明文并无区别，安全性低

#### HTTP Digest Auth

Digest: 在 Basic 基础上，增加了安全性，他并非直接发送用户名和密码明文

1. 由客户端发起认证请求
2. 服务端返回**质询码**
3. 客户端由质询码和密码算出**响应码**，并发送给服务器

```
// HTTP Response
HTTP/1.1 401 Authorization Required
WWW-Authenticate: Digest realm ="Digest" nonce="xxx" // 服务端发送质询码


// HTTP Request
Authorization: Digest username="xxx" realm ="Digest" nonce="xxx" // 向服务端发送响应码
```

#### SSL Auth

以 SSL 证书形式认证，但 SSL 证书只能识别设备，无法识别用户

#### Form Auth

**表单认证:** 基于表单提交用户名和密码来认证。**目前大多数为表单认证**

## LeetCode

### 旋转矩阵

思路:

1. 先上下交换
2. 在按对角线交换

### 机器人运行范围

思路:

深度遍历，由 [0, 0] 开始遍历

```typescript
function movingCount(m: number, n: number, k: number): number {
  let s = 0;
  const arrive: number[][] = [];
  const cal = new Map<number, number>([
    [0, 0],
    [1, 1],
  ]);

  for (let i = 0; i < m; i++) {
    arrive[i] = [];
  }

  handle(0, 0);

  return s;

  function handle(i: number, j: number): void {
    // 超出范围 或已经访问过了
    if (i >= m || j >= n || f(i) + f(j) > k || arrive[i][j] === 1) {
      return;
    }

    arrive[i][j] = 1; // 表示已经访问过了
    s += 1;

    handle(i + 1, j);
    handle(i, j + 1);
  }

  // 计算 数字和
  function f(num: number): number {
    let s = 0;
    while (num > 0) {
      if (cal.has(num)) {
        return s + cal.get(num);
      }

      s += num % 10;
      num = Math.trunc(num / 10);
    }
    return s;
  }
}
```

**注意点：**

1. 对重复访问判断
2. 对无法访问点的判断

[机器人运行范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

### 翻转字符串单词

思路：

1. 去除首位空格
2. 遍历每个单词，以" "字符串为界限，拼接每个字符串，拼接完成后

```typescript
// api调用
function reverseWords(s: string): string {
  return s
    .split(' ')
    .filter((i) => i !== '')
    .reverse()
    .join(' ');
}
```

```typescript

```

**注意：**

1. 不是每个字符都反转，而是按照单词反转，单词内部还是原来的顺序

[翻转字符串单词](https://leetcode-cn.com/problems/reverse-words-in-a-string/)
