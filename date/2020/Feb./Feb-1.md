## Javascript

### 理解 Javascript 版本

理解 Javascript 版本差异，例如 ES5 和 ES6；以及严格模式下的差异

**严格模式**

1. 修复了松散模式下的静默错误，严格模式下会报错
2. 提升了性能
3. 禁用在 ECMAScript 将来可能使用的语法

**严格模式规则**

1. 变量需声明，再使用
2. 抛出静默错误
   - 给不可赋值属性赋值，例如 ** IIFE 考题**
   - 删除不可删除属性
   - `delete 变量`
3. 禁止部分语法
   - with 语法
   - 不可使用 arguments.callee

### 理解 Javascript 数字

#### 双精度模型 (double)

1. 双精度模型: 1 符号位，11 指数位，52 尾数位

   - 符号位: 0 正；1 负
   - 指数位: 无符号整数，以 [1, 2046 ] 表示规约数，且[1, 1022] 表示负数值，1023 表示 0，[1024, 2046] 表示正数；为 0 和 2047 时都表示特殊数值
   - 尾数位: 1. 将所有数字以二进制科学计数法表示；2. 省略第一位 1，所以最大数能表示 53 位

```
1. 当指数 === 0: 尾数为 0 时，表示数值 0；尾数不为 0 时表示非规约数值
2. 当 1<= 指数 <= 2046 时: 规约数，按照公式计算 `(-1)^S * 2^(E - 1023) * (1 + F);`
3. 当 指数===2047 时: 尾数为 0 代表(正、负)无穷；尾数不为 0 代表 NaN

符号位(1)   指数位(11)   指数位表示值   位数位(52)   表示值
*           0           -1024         0            0       // 全部为0时表示为0
0           1~2^11-1    -1023~1023    *            正数
1           1~2^11-1    -1023~1023    *            负数
0           2^11-1       1024         0            正无穷
1           2^11-1       1024         0            负无穷
*           2^11-1       1024         非0           NaN

------------------------ 非规约数 ------------------------
*           0            -1023        非0          非规约数
```

2. 数字位运算: Javascript 中位运算只能使用整数运算（32 位有符号整数），如果不在该范围则会转换为该范围的整数

### == 转换规格

```
1. null == undefined
2. null、undefined 与其他值都不相等（包含false，但是注意，布尔转换时，null、undefined为false）
3. 原始类型(string,number,boolean) 和 Date 对象，原始类型转换为数字，Date对象转换为原始类型，先尝试toString()，再尝试valueOf()
4. 原始类型(string,number,boolean) 和 非Date 对象，原始类型转换为数字，非Date对象转换为原始类型，先尝试valueOf()，再尝试toString()
5. 原始类型(string,number,boolean)相互比较，将原始类型转换为数字
```

### 自动补充分号

1. 分号在 "}" 括号结束前，一行结束后，程序结束时
2. 引擎自动补充分号只是在**无法正常解析程序**时触发
3. for 语句无法省略分号

```
let a;
function b() {
  console.log('b');
}
function f() {
  console.log('f');
}

a = b
(f()); // 此时是不会添加分号的，因为可以正常执行，但有无分号有歧义
```

### unicode

ES6 增加 unicode 支持

unicode: 全球统一编码，编码格式有 utf-8、utf-16 等。unicode 最初是由 16 位表示，即只有 2^16 个码点，后续 unicode 扩充到 2^20 个码点。

但 **Javascript 中代码单元是由 16 位组成的**，可以表示 2^16 次，但无法表示 2^16~2^20 的字符。所以 ES6 增加为由**两个代码单元来表示一个 2^16 以上的 unicode 字符**，称为代理对，同时 ES6 也新增了各种字符串方法来支持 unicode

```
'\z' === 'z'  // true
'\172' === 'z' // true
'\x7A' === 'z' // true
'\u007A' === 'z' // true
'\u{7A}' === 'z' // true
```

### 函数名称

```
function f(){}

const f1 = function f() { // 函数名为f1，而非f，外部无妨访问f
  console.log(f === f1); // 内部可访问f
};
```

函数调用 toString()时，ECMAScript 没有任何规定，全靠引擎实现

## Node

### Stream

Stream 继承 EventEmitter，是 Node 流式编程的基础，由 Buffer 实现，`stream.push() => Buffer => stream.read()`

主要分为 Readable，Writable，Duplex，Transform 四种流，对于双工流则同时维护两个缓冲区

#### Writable

##### 事件

1. close: 如果使用 emitClose 选项创建可写流，则它将会始终发出 'close' 事件
2. drain: 当 writable 无法写入且（`writable.write(chunk)===false`）后，**可继续写入时触发**
3. error: 错误，回调传入 Error，除非在创建流时将 autoDestroy 选项设置为 true，否则在触发 'error' 事件时不会关闭流
4. finish: 调用 `writable.end()`缓冲区全部传给底层系统后触发
5. pipe: `readable.pipe()`方法
6. unpipe：`readable.unpipe()` 移除管道

##### 方法

1. cork(): 强制把所有写入的数据都缓冲到内存中,当调用 writable.uncork() 或 writable.end() 时，缓冲的数据才会被输出。防止当写入大量小块数据到流时，内部缓冲可能失效，从而导致性能下降
2. uncork()：如果一个流上多次调用 writable.cork()，则必须调用同样次数的 writable.uncork() 才能输出缓冲的数据
3. destroy(): 销毁流，可选触发 error 事件,并且触发 'close' 事件（除非将 emitClose 设置为 false）
4. end([chunk[, encoding]][, callback]):
5. setDefaultEncoding(encoding)
6. write(chunk[, encoding][, callback])

##### 属性

1. destroyed: 是否为销毁
2. writable: 是否安全，是否 `writable.write()===true`
3. writableEnded: 是否完成 `writable.end()`后为 true
4. writableFinished: 触发`finish`事件后为 true ...

[可写流](http://nodejs.cn/api/stream.html#stream_writable_streams)

#### Readable

readableFlowing: null、true、false

所有 Readable 分为 两种状态 flowing 和 paused，当创建流时，readableFlowing 为 null，当调用`pipe()、data事件、resume()` 时 readableFlowing 为 true，变为 flowing 状态；调用 `unpipe()、pause()` 时 readableFlowing 为 false，且变为 paused 状态

**当 readable.readableFlowing 为 false 时，数据可能会堆积在流的内部缓冲中**

```
// 可读文件流实现
const LEN = 100;
const bf = Buffer.alloc(LEN);
const fd = fs.openSync('./node.ts');
let op = 0;

const rd = new stream.Readable({
  read(size) {
    fs.read(fd, bf, 0, LEN, op, (err, bytesRead, bf) => {
      if (bytesRead <= 0) return;
      this.push(bf);
      op += LEN;
    });
  }
});
```

#### Duplex

双工流

如 TCP Socket 流

#### Transform

转换流是一种 Duplex，但是它的输入和输出是相关的

如 zlib 流

#### stream 方法

1. finished(stream[, options], callback): 结束流，例如 HTTP 结束，不会触发 end,finish 事件
2. pipeline(...streams, callback): 多个流进入管道
3. Readable.from(iterable, [options])

**继承流时，避免重写诸如 write()、 end()、 cork()、 uncork()、 read() 和 destroy() 之类的公共方法，或通过 .emit() 触发诸如 'error'、 'data'、 'end'、 'finish' 和 'close' 之类的内部事件。 这样做会破坏当前和未来的流的不变量，从而导致与其他流、流的实用工具、以及用户期望的行为和/或兼容性问题**

### StringDecoder

将 utf8、utf16 Buffer 转换为字符串

```
const sd = new stringDecoder.StringDecoder('utf8')
sd.write(buf);
```

buf.toString(encoding, start, end); // 可以选择编码、起始和结束为止。而 StringDecoder 开始必须制定编码，且无法选择起始和结束位置

### Timer

定时器句柄

#### Immediate

由 setImmediate 返回，只可以给 clearImmediate 调用

#### Timeout

有 setTimeout 和 setInterval 返回，只可以给 clearTimeout 和 clearInterval 调用

## 网络

### HTTP2

#### 特点

1. 在 HTTP1 基础上升级，而非完全重写，且完全支持 HTTP1 语义
2. 二进制分帧
3. 多路复用
4. 头部压缩
5. 服务端推送(server push)

#### 二进制分帧

**流（Stream）**：HTTP2 中，对服务端和客户端连接的抽象的虚拟信道，可以承载双向消息，每个流都存在唯一标识。\*为了防止标识冲突，客户端发起的流具有**奇数 ID**，服务端发起的流具有**偶数 ID\***。一个 HTTP2 连接可打开多个并发流

**消息（Message）**：逻辑上的 HTTP 消息，例如 HTTP 请求、响应，由 1 到多个帧组成

**帧（Frame）**：HTTP2 通信传输的最小单位，每个帧包含首部，标识当前所属流，有效载荷

HTTP2 将信息分割，以二进制编码封装，以 Message（由 Frame 组成） 的形式在流中传输，且每个帧传输时是无序的，根据首部标识和流标记重新组装

#### 多路复用

1. 在 HTTP1.0 中，HTTP 无法复用 TCP 连接，每次 HTTP 结束请求后都会断开 TCP 连接，等到下次请求时再建立。在 HTTP1.1 中使用 "keep-alive" 来使 TCP 连接等待一定时间，再断开连接；但在域名分片情况下，任然需要建立多个连接。

2. 在 HTTP1.0 中，如果出现**队头阻塞**，则后续资源也无法发送，必须等待前资源发送完毕。在 HTTP1.1 使用 "pipeling" 来解决，单个 TCP 连接可以同时发送多个 HTTP 请求，但是**后续响应也必须等待前响应处理完毕**，即使是它先返回

**多路复用**：

1. HTTP2 中一个域名只创建一个 TCP 连接，减少 TCP 连接的时间和资源消耗；
2. 单个 HTTP 连接可并发多个流，不在依靠并发 TCP 连接；
3. 将信息分割、封装为 Frame 且可以无序发送，可以并发的发送请求、响应
4. 每个 Frame 头部带有 32 bit 优先值，0 表示最大，2^32-1 表示最小，**但可能传输过程中，不会严格按照优先级传输**

#### 头部压缩

在 HTTP1 中，每个请求、响应都带有头部信息，为了减少该资源消耗，HTTP2 中采用头部压缩

1. HTTP2 中客户端和服务端各自缓存一份头部的字段表(首部表)，存储之前发送和接收的头部信息
2. 对于不会改变的头部信息，只会传输一次
3. 对于变化的头部信息，要么添加到头部表末尾，要么替换头部表中某个值

头部压缩使用 HPACK 算法，使用 静态表+动态表同时维护。静态表是只读的；动态表面向连接，每个连接中请求和响应的动态表不同，不同连接动态表不同

#### 服务端推送

**服务端推送**：HTTP2 是长连接，已在连接时，把客户端一定需要的资源推送（浏览器 prefetch），例如可以在浏览器微解析 HTML 情况下推送 CSS，Javascript 等。推送基于**同源策略 + 客户端确认**(客户端也可以拒绝推送)

#### HTTP2 未解决的问题

1. 队头阻塞：由于 HTTP 基于 TCP，如果 TCP 出现丢包重传、超时重传等，此时 TCP 会终止其他包的传输，重新传输丢失的包。随着丢包率增加，HTTP2 性能急速下降

## 算法

### B 树

#### B-Tree

B-Tree: 多节点平衡树，是以前学过的 2-3 节点树的延伸。M 阶 B-Tree，表示节点最多包含 M 个子节点，M-1 个关键词

##### 特点

1. 如果根节点存在子节点，则最少存在两个子节点
2. 每个中间节点（**非叶节点**）至少包含 k 个节点，k-1 个关键字，k 在 M/2~M 之间
3. 所有叶子节点必须处于同一层

##### 优势

和平衡二叉树比较：二叉树的是一颗"瘦高"的树，查询次数和树的深度有关(时间复杂度为 logN)，B-Tree 是一颗"矮胖"的树，查询的次数不比平衡二叉树少，因为节点内部还需要比较。但是，这里涉及到**磁盘 IO**。

磁盘 IO: 持久化的数据是保存在磁盘中的，在进行索引读取时，不可能将整个索引全部加载到内存中，都是逐一加载每个磁盘页，由于树所需要磁盘 I/O 次数和**树的深度**有关，则深度越大的树，所需要的磁盘 I/O 越多，由于磁盘 I/O 时间消耗 和 内存中操作时间消耗差距巨大，所以在磁盘 IO 情况下，B-Tree 性能提升巨大

#### B+Tree

B+Tree: 在 B-Tree 的扩展

##### 特点

1. 节点子树数量和节点的关键词**数量一致**（B-Tree 关键词比子树数量少 1）
2. 叶节点保存数据，除开叶节点，其他节点不保存数据（B-Tree 中间节点会保存数据）
3. 所有的中间节点元素，同时存在于子节点，且在子节点中为最大值，所以叶节点包含所有的元素
4. 叶节点被单向链表连接

##### 优势

1. 由于非叶节点不存储数据，则同样的磁盘空间可以存储更多的节点；意味着 B+Tree 比 B-Tree 更加"矮胖"，磁盘 I/O 更少
2. B-Tree 由于所有节点都存储数据，则查询数据效率不稳定，而 B+Tree 查找树稳定，必然会查询到叶节点
3. **范围查找**时，B-Tree 只能遍历整个树（中序遍历）来确定，但 B+Tree 以叶节点来确定，效率极高
