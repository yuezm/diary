## Typescript

### export

TS 的 export、import 总体与 ES6 Module 表现一致，例如如下使用

```typescript
import * as A from 'A';
import A ,{ B, C as D } from 'A';
import  from 'A';
...

export A;
export { A, B, C as D, E: F };
export default A;
export * from 'A'
...
```

但是 TS 对 CommonJS 和其他模块都做了兼容，如下处理 `export = ; import...require`

```typescript
export = A;

import A = require('A');
```

## Node

### hack process.stdout

process.stdout 是 Node 标准输出流，该属性返回 stdout （fd(1)）,是一个 net.Socket 流，除非 fd(1)指向一个文件，这种情况是一个可写流。如果你不想对标准输出打印到 Shell 中，可以对其进行 hack

```
fs.createReadStream(...).pipe(process.stdout);
```

```javascript
// lib/internal/process/stdio.js

function setupProcessStdio({ getStdout, getStdin, getStderr }) {
  Object.defineProperty(process, 'stdout', {
    configurable: true,
    enumerable: true,
    get: getStdout
  });
  ...
}
```

```typescript
// hack process.stdout
import fs = require('fs');
import tty = require('tty');

const ws = fs.createWriteStream('./log', { flags: 'a+' }); // 写入文件
const ttyStream = new tty.WriteStream(1); // 打印到终端

// 重写它的方法，如果不想原来的行为改变，可以继续往tty打印
process.stdout._write = function (
  chunk: any,
  encoding: string,
  callback: (error?: Error | null) => void
) {
  ws.write(chunk.toString(), 'utf-8');
  ttyStream.write(chunk.toString(), 'utf-8'); // 继续往tty打印
  callback();
};
```

### 热更新

**优势**

1. node: 无需重启，即可以不停止服务，快速发布更新
2. client: 无需拉取整个代码，加快更新速度；无需刷新整个界面，做到无感更新

**劣势**

1. node: 由于 node 缓存机制，无法实时获取模块变更；
2. client: 同时带来局部更新计算、包完整性校验等复杂问题

node 由于缓存，无法实时读取文件的变化，**删除缓存后，再次 require**可以加载新的文件，但会带来其他的问题:

1. 必须检测文件的变化，在变化后，需删除缓存，且再次 require，会导致整个架构的变更
2. 可能导致新、老代码同时运行。比如 某些模块重新 require 了，而有的模块未重新 require

### node 上下文

Context: v8 上下文，在执行 node 时，有许多环境变量和全局方法，Context 为 Javascript 提供这样的执行环境。一般来说 node 只有**一个 Context**，但可以使用*vm*模块和 addon 来增加额外的 Context

**Javascript 代码运行于 Context，而非 Isolate**

```cpp
// addon
using namespace v8;

void Test(const FunctionCallbackInfo<Value> &args) {
  Isolate *isolate = args.GetIsolate();

  // 获取当前上下文
  Local<Context> ctx = isolate->GetCurrentContext();
  // 创建新的执行上下文
  Local<Context> newCtx = Context::New(isolate);

  // 执行javascript
  Local<String> source = String::NewFromUtf8(isolate, "console.log('Hello word!!!')");
  Local<Script> script = Script::Compile(ctx, source).ToLocalChecked();
  script->Run(ctx).ToLocalChecked();
}
```

```javascript
// vm
const vm = require('vm');
const ctx = { name: 'Test' };

vm.createContext(ctx); // 创建时传入参数，作为上下文中的全局变量
const source = "name = 'Test1'";
vm.runInContext(source, ctx); // ctx.name => Test1

const script = new vm.Script('name = 2'); // script 方式执行
script.runInNewContext(ctx); // ctx.name => 2
```

## Vue

### Vue 无渲染组件

无渲染组件: 组件内部维护状态，且维护处理状态方法，但**不维护 UI 界面，UI 由用户自己完成**。无渲染组件有以下特点

1. 使用 _render_ 方法渲染，无 _template_，使用*scope、scopeSlots*处理父级传值
2. 由于组件未定义 UI，用户可以定义任何 UI 样式，更加灵活
3. 维护状态和处理状态方法，使组件行为保持一致

```javascript
export default {
  name: 'CustomInput',
  props: {
    value: {
      type: String,
      required: true,
    },
  },
  methods: {
    handleInput() {},
  },
  render() {
    // 提供渲染函数，无UI
    return this.$scopeSlots.default({
      value: this.value, // 维护状态
      handleInput: this.handleInput, // 提供状态处理方法
    });
  },
};

<CustomInput>
  <template v-slot="{value,handleInput}">
    <input />
  </template>
</CustomInput>;
```

**劣势**

1. 更多的代码，需要用户处写更多的 HTML
2. 当对 UI 变化要求不大时，作用不大

### Vue key

vue Key：在 diff 算法时，使用 key 可以更好的复用 DOM，减少删除、新增 DOM 的开销，提高性能

#### 数组索引做为 key 值引发的问题

数组索引作为 key 值，在数组出现**删除值、值顺序变化**时，在 diff 时，可能会降低性能

**删除值**

```
value [1, 2, 3] => [2, 3]
key    0  1  2      0  1
```

该情况，按照 diff 算法，0,1 可复用，并对子节点再次 patch；2 无法复用直接删除。**实际删除了 值为 3 的节点**

**值乱序**

```
value [1, 2, 3] => [3, 2, 1]
key    0  1  2      0  1  2
```

该情况，按照 diff 算法，0,1,2 可复用，并对子节点再次 patch。**仅仅可以移动节点可完成更新**

## HTTP

### HTTP 协议缺点及 HTTP1.1 和 HTTP2.0 改进

1. HTTP1.0 连接无法复用，在一次 HTTP 请求完成后，即断开 TCP： HTTP1.1 加入 Keep-Alive 在 Keep-Alive timeout 前可复用 TCP 连接；_HTTP2.0 多路复用_
2. HTTP1.0 头部阻塞，下一个请求必须等待上一个请求返回: HTTP1.1 加入 Pipelining，一个 TCP 连接可发送多个请求，但响应任然是有序的，_即使后发送的请求完成，也必须等待先发送的请求处理完毕_；_HTTP 二进制分帧，每帧并发传输，且可以控制传输优先级_
3. HTTP 协议开销较大，HTTP header 每次都需要传输：HTTP2.0 加入头部压缩、头部缓存
4. HTTP 不安全，明文传输: 浏览器要求 HTTP2.0 必须使用 HTTPS
5. HTTP 无法服务器主动推送：HTTP2 server push

HTTP2.0 中，由于复用一个 TCP，由于 TCP 可靠传输机制，当出现丢包时，即会阻塞后续传输，此时 HTTP2 性能会下降，**丢包越多，HTTP2 性能下降越多**

### websocket

websocket 开始提供的，使客户端-服务端建立*全双工、长连接*协议

1. 基于 TCP 协议，属于应用层协议
2. 与 HTTP 完美配合，可由 HTTP 升级
3. 新协议名 ws，也可使用安全的 wss（websocket + SSL）
4. 全双工，双方可以发送、接收信息
5. 传输多对象，可传输二进制、文本、JSON
6. **无同源限制**

**缺点**

1. 浏览器支持程度不同，有的浏览器不支持
2. 服务端需要消耗额外的资源来保持长连接

#### HTTP 升级

```
// Request

Upgrade: websocket
Connection: Upgrade
```

```
// Response

HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

#### websocket 断线重连

1. 心跳机制：定义的向服务器发送数据，如果得到指定回应，则认为保持连接中，否则进行重连
2. 监听 close 事件，触发 close 事件后，重新连接。**重新连接需要重新绑定事件回调函数**
3. 监听 error 事件，触发 error 事件，判断触发原因，如果是断开导致的错误，则进行重连

**优雅降级**：在网络较差或不稳定时，可以使用心跳拉取结合 websocket

## 安全

### 攻击

1. 主动攻击：攻击者主动攻击服务器，或客户端
2. 被动攻击：攻击者设置陷阱，诱导用户操作，从而实现攻击

#### 转义不安全

1. XSS 攻击
2. SQL 注入
3. OS 注入
4. HTTP 头部注入

#### 设计缺陷导致

1. 强制浏览
2. 不正确的错误信息返回
3. 开放重定向而未校验

#### 会话管理不当

1. 会话劫持
2. CSRF

#### 密码破解

1. 暴力法
2. 撞库（彩虹表）

## LeetCode

### 01 矩阵

思路 1: 广度遍历，对每一个 0 执行广度遍历，并对每个 1 节点取最小值

思路 2: 动态规划

1. 公式推导: f(i, j) = matrix[i][j] === 0 ? 0 : min(f(i-1, j), f(i+1, j), f(i, j-1), f(i, j+1)) + 1。简单来说，就是在四个方向中取最小值，将四个方向拆分 **左上 + 右下** 分两个遍历，则可以分别算出 **左上、右下** 的最小值
2. 存储: 矩阵存储

```javascript
function updateMatrix(matrix: number[][]) {
  const store = [];

  const row = matrix.length;
  const col = matrix[0].length;

  // 将1赋予最大值，利于后续比较
  for (let i = 0; i < row; i++) {
    store[i] = [];
    for (let j = 0; j < col; j++) {
      store[i][j] = matrix[i][j] === 0 ? 0 : 10000;
    }
  }

  // 从左上往右下，则 推导公式为 f(i, j) = matrix[i][j] === 0? 0: min(store[i][j], f(i-1,j),f(j,j-1))+1
  for (let i = 0; i < row; i++) {
    for (let j = 0; j < col; j++) {
      if (i > 0) {
        store[i][j] = Math.min(store[i][j], store[i - 1][j] + 1);
      }

      if (j > 0) {
        store[i][j] = Math.min(store[i][j], store[i][j - 1] + 1);
      }
    }
  }

  // 从右下，则 推导公式为 f(i, j) = matrix[i][j] === 0? 0: min(store[i][j], f(i+1,j),f(j,j+1))+1
  for (let i = row - 1; i >= 0; i--) {
    for (let j = col - 1; j >= 0; j--) {
      if (i < row - 1) {
        store[i][j] = Math.min(store[i][j], store[i + 1][j] + 1);
      }

      if (j < col - 1) {
        store[i][j] = Math.min(store[i][j], store[i][j + 1] + 1);
      }
    }
  }

  return store;
}
```

[01 矩阵](https://leetcode-cn.com/problems/01-matrix/)

### 合并区间

思路 1: 按照起始字符排序，如果 arr[i][1] >= arr[i+1][0] 则表示两个有重合，需要合并

```javascript
function merge(intervals: number[][]) {
  if (intervals.length <= 1) return intervals;

  // 排序
  intervals.sort((a, b) => a[0] - b[0]);

  const result: number[][] = [intervals[0]];
  let last = 0;
  for (let i = 1; i < intervals.length; i++) {
    // 表示存在关联，则合并需要注意结束数字大小
    if (result[last][1] >= intervals[i][0]) {
      result[last][1] = Math.max(result[last][1], intervals[i][1]);
    } else {
      result[++last] = intervals[i];
    }
  }
  return result;
}
```

[合并区间](https://leetcode-cn.com/problems/merge-intervals/)

### 跳跃游戏

思路: 当出现第 i 个，i + arr[i] >= arr.length-1，表示可以达到末尾；即我们的任务是寻找这个点，即从 i=0 开始，记录最大的可达到的索引，在该索引范围内，寻找达到条件的 i

```javascript
function canJump(nums: number[]): boolean {
  if (nums.length <= 1) return true;

  let maxIndex = 0; // 记录最大的索引
  for (let i = 0; i < nums.length; i++) {
    if (maxIndex >= nums.length - 1) return true; // 满足最终条件
    if (i > maxIndex) return false; // 在最大索引内寻找
    maxIndex = Math.max(maxIndex, i + nums[i]); // 记录最大索引
  }
  return false;
}
```

[跳跃游戏](https://leetcode-cn.com/problems/jump-game/)
