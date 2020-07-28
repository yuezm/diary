## Javascript

### Babel

#### Babel 自定义 Visitor 模式

```
import * as babel from '@babel/core';
import * as types from '@babel/types';
import * as fs from 'fs';

interface Visitor {
  [ key: string ]: Function;
}

const ast = babel.parseSync(fs.readFileSync('./index.js').toString());
walk(ast, {
  Identifier(node, parent, index, key) {
    // 可以获取到该节点
  }
});

// 深度遍历函数
function walk(nodes: types.Node, visitor?: Visitor) {
  if (!nodes) return;
  _walk(nodes, null);

  function _walk(node: types.Node, parent: types.Node, index?: number | null, key?: string | null) {
    if (visitor && node.type in visitor) {
      // 回调传参 当前节点，父节点
      visitor[ node.type ](node, parent, index, key);
    }

    for (const attr in node) {
      // 数组遍历
      if (Array.isArray(node[ attr ])) {
        const nodeList: types.Node[] = node[ attr ];
        for (let i = 0; i < nodeList.length; i++) {
          if (types.isNode(nodeList[ i ])) _walk(nodeList[ i ], node, i, null);
        }
      } else {
        if (types.isNode(node[ attr ])) _walk(node[ attr ], node, null, attr);
      }
    }
  }
}
```

### 微前端

#### 特点

1. 独立部署、独立交付
2. 拆分项目、项目间解耦
3. 技术栈无关

#### 实例划分

1. 单实例: 同一时刻只存在一个实例展示
2. 多实例: 同一时刻存在多个实例展示

#### 整合方法

1. 模板嵌套: 模板语法，Nginx 嵌套
2. webpack 打包组合
3. 运行时组合: 主框架负责路由调度，微服务负责渲染

### ES2018

#### 异步 Iterator

```javascript
for await (const p of ps) {
  // 异步执行任务，切记异步任务时推入线程池执行，所以异步人不并非需要等到上一个异步执行完成，即使是 await
}
```

#### Promise.prototype.finally

无论失败与否都会执行该步骤

#### 正则新增

##### 标识符 s

标识符 s: 表示 "." 是否可以表示换行符，原来正则中.表示除换行符以外的所有字符

##### 正则具名组匹配

```javascript
const str = '2014-02-02';
const re = /(\d{4})-(\d{2})-(\d{2})/;
str.match(re); // 原来只能使用match数组下标获取对应值

const re_new = /(?<year>\d{4})-(?<month>\d{2})-(?<date>\d{2})/;
str.match(re_new).groups; // 可以通过具名匹配 year,month,date获取

const str1 = 'aaa_bbb';
const re_n1 = /(?<A>a)/g;
console.log(str1.replace(re_n1, '$<A>')); // 使用具名匹配替换$<A>可以访问对应的名称

// 具名匹配引用 \k<组名>，当然\n还是有效的
/(?<A>a{3})_\1/;
/(?<A>a{3})_\k<A>/;
```

##### 后行断言

Javascript 原先只存在线性断言，即匹配在 xx 之前的字符，现在增加后行断言，匹配处于 xx 之后的字符

```javascript
const str = 'aaa_bbb';
const re_p = /a*(?=_bbb)/; // 匹配处于_bbb前的字符
const re_b = /(?<=aaa_)b*/; // 匹配处于aaa_后的字符
```

## Node

### Node 最佳实践

#### Node 日志实践

判断日志好坏标准:

1. 日志类型明确
2. 能根据日志迅速排查出问题
3. 不需要临时增加日志
4. 可以同时给测试人员和运维人员使用
5. 需要考虑日质量级

##### 日志分类

1. 系统日志: 包含系统状态、内存、磁盘、CPU 使用情况等，应该由运维人员维护
2. 统计日志: 统计用户的使用状态、用户操作等，由前端打点、后端日志同时组成，一般跟着业务而定
3. 诊断日志:
   - 系统启动日志: 一般由开发框架统一完成，普通开发人员无需关心
   - 系统配置、加载配置日志: 一般由开发框架统一完成，普通开发人员无需关心
   - 操作日志、调用日志: 开发人员常用日志

##### 日志等级

2. FATAL: 致命错误，服务已经挂了，需要运维马上处理
1. ERROR: 严重错误，此时服务已经不能访问

**FATAL、ERROR 不可以轻易使用**

3. WARN: 警告错误，虽然服务还能访问，但可能会导致问题，需要处理
4. INFO: 普通信息展示，一般为记录正常运行状态，INFO 不应该大于 DEBUG 10%
5. DEBUG: 普通记录信息，一般为详细操作步骤

##### 日志内容

1. 日志等级
2. 日志时间
3. 日志主机: 分布式部署时，记录是哪个主机的记录
4. requestID: 用户请求唯一 ID，串联整个用户操作
5. 参数，例如请求参数和响应值

##### 动态日志

由于排错时，往往需要很详细的日志信息，但如果平时全部输出的话，日志量巨大。所以采用动态日志**在参数中带上 DEBUG=ON，即为开启 DEBUG 日志，会打印出当前详细操作**，平时不会打印 DEBUG 日志

##### 快慢日志

打印用户操作用时，用于收集系统性能，确定系统优化点

### C++ 扩展

#### N-API

```c++
napi_value Echo(napi_env env, napi_callback_info info) {
  napi_status status;

  size_t argc = 1;
  napi_value argv[1];

  status = napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
  if (status != napi_ok) {
    napi_throw_error(env, "NORMAL", "参数转换错误");
  }

  napi_valuetype s = napi_string;
  status = napi_typeof(env, argv[0], &s);

  if (status != napi_ok) {
    napi_throw_error(env, "NORMAL", "参数类型错误");
  }

  if (argc < 1) {
    napi_throw_error(env, "NORMAL", "参数个数错误");
  }

  char str[10];
  napi_get_value_string_utf8(env, argv[0], str, 10, nullptr);
  cout << str << endl;
}

napi_value Init(napi_env env, napi_value exports) {
  napi_status status;
  napi_property_descriptor des = {"echo",nullptr, Echo,nullptr, nullptr, nullptr, napi_enumerable, nullptr};
  napi_define_properties(env, exports, 1, &des);
}

NAPI_MODULE(addon, Init);
```

## 网络

### gRPC

gRPC: google 在 HTTP2 基础上封装的 RPC 协议。包含 unary，client stream，server stream，双向流。虽然在 HTTP2 都是流，但 gRPC 是 Request-Response 模型的

#### Request

包含 Request-Header，0 或者多个 Length-Prefixed-Message(理解为 Body)，EOS

1. Request-Header 使用 HTTP2 Header，在 Header Frame 或 CONTINUATION Frame 派发，例如

   - Call-Definition: 定义请求方法，POST
   - Custom-Metadata: 主要定义任意 key-value 数据, 但不建议 key 以\*grpc-\*\*起始，gRPC 可能会使用到（可以理解为保留字）

2. Length-Prefixed-Message 在 Body Frame 派发

   - Compressed flag: 表示数据是否压缩，压缩算法使用 Message-Encoding 定义

3. EOS(end of stream): 如果 Data frame 带上 END_STREAM, 则表示不会再发送数据

#### Response

包含 Response-Header，0 或者多个 Length-Prefixed-Message，Trailers，如果出现错误，则可能只返回 Trailers

1. Response-Header

   - HTTP 状态码
   - Content-Type
   - Custom-Metadata

2. Length-Prefixed-Message

3. Trailers
   - gRPC Status
   - 0 或者多个 Custom-Metadata
   - EOS: 如果在最后收到的 HEADERS frame 里面，带上了 Trailers，并且有 END_STREAM 这个 flag，那么就意味着 response 的 EOS

gRPC 的 service 接口是基于 protobuf 定义的，我们可以非常方便的将 service 与 HTTP/2 关联起来。

```javascript
Path : /Service-Name/{method name}
Service-Name : ?( {proto package name} "." ) {service name}
Message-Type : {fully qualified proto message name}
Content-Type : “application/grpc+proto”
```

## LeetCode

### 二叉搜索树中第 K 小的元素

思路:

1. 二叉搜索树特点: 左节点比根节点小，右节点比根节点大；
2. 二叉搜索树排序: 二叉搜索树的中序遍历，即为从小到大的排序

### 括号生成

思路:

1. 括号由 ( 开始
2. 每次在右边加上 ) 或者 (
   - 如果结果的 )的个数大于(，则失败 (回溯剪枝)
   - 如果结果数量符合要求，则返回正确
   - 如果个数不符合要求，则继续进入步骤 2

## 算法

### 压缩算法

**压缩方式**

1. 有损压缩: 无法恢复到原来的数据
2. 无损压缩: 可以恢复到原来的数据

**压缩效率**

1. 压缩后的大小（压缩率）
2. 压缩用时

**压缩特点**

1. 无法寻找到通用的压缩方式
2. 无法寻找到最佳的压缩算法，不同的数据最佳的压缩算法不同

所以，必须要求压缩数据已知结构

1. 小规模字母表
2. 较长连续相同的位和字符
3. 频繁使用的字符
4. 较长连续重复的位和字符

1) 栗子 1：基因码，基因码由 A、C、G、T 组成，所有可以用 2 位来表示，以 00,01,10,11 来分别表示 A,C,G,T，压缩率接近 25%
2) 栗子 2：游程编码，取一截字符为一种编码，例如 00011100001111，00，01，10，11 表示 000,111,0000,1111
3) 栗子 3：位图，游程编码的应用，用于压缩图片等栗子 4：普通压缩，例如用 int(16 位)来分别保存不同的数据，例如第 1~5 位保存日，6~9 位保存月

#### 哈夫曼算法

以字符出现的次数为权重，权重小的表示字符长，权重大的字符较短

**压缩步骤**

1. 计算字符出现的频率
2. 根据频率组成二叉树，如下 Demo 所示

```text
[1,3,4,6,7]

// 第一二叉树由 1、3组成，父节点为 4
 4
1  3

// 4<=4 ，则合并
   8
 4   4
1 3

// 8>6并且8>7，则将8移到最后
  13
6   7

// 最终 整个二叉树
      21
   8      13
 4   4   6   7
1  3
```

3. 计算编码时，左树边加 0，右树边加 1，举个栗子，**1 节点的编码为 000，7 节点编码为 11**

#### LZW 算法
