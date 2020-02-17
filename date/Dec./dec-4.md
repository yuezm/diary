# 11 月第 4 周

## Javascript

### Babel

Babel: 将 Javascript 转换为向下兼容的代码，例如常用的将 ES6+代码转换为 ES5

1. 解析（语法解析，词法解析），使用@babel/parser
2. 转换，插件介入过程，操作 AST 树，使用@babel/traverse
3. 代码生成，生成目标代码，使用@babel/generator

4. @babel/types: 提供各种的工具和节点的定义。包含删改节点、增加节点的函数、验证节点
5. @babel/template: 字符串形式占位符，计算机科学中，这种能力被称为准引用（quasiquotes）

```
const type = require('@babel/types');
const generate = require('@babel/generator');
const template = require('@babel/template');

const buildRequire = template.default(`
  var NAME = require(PATH);
`);

const ast = buildRequire({
  NAME: type.identifier('code'),
  PATH: type.stringLiteral('./node.js'),
});

generate.default(ast); // var code = require("./node.js");
```

#### 配置

配置文件可以单独写入文件（_babelrc_, _.babelrc.json_, _babel.config.json_, _babel.config.js_ 等），也可以写在 package.json 文件中

##### preset

预设规范，提供整套转换机制（其实就是一整套差价的集合）

**执行顺序**: preset 按照既定顺序，**从后往前执行**，这主要是为了确保向后兼容，由于大多数用户将 "es2015" 放在 "stage-0" 之前

1. Stage 0 - 设想（Strawman）：只是一个想法，可能有 Babel 插件。
2. Stage 1 - 建议（Proposal）：这是值得跟进的。
3. Stage 2 - 草案（Draft）：初始规范。
4. Stage 3 - 候选（Candidate）：完成规范并在浏览器上初步实现。
5. Stage 4 - 完成（Finished）：将添加到下一个年度版本发布中

```
// babel.config.js

module.exports = {
  presets: [
    "@babel/preset-env",
    ["@babel/preset-env", options], // 参数传递
    require("@babel/preset-env"),  // 手动加载模块
    // 如果 preset 名称的前缀为 babel-preset-xx，你还可以使用它的短名称 xx
    // babel-preset-demo, @babel/babel-preset-demo
    "demo",
    "@babel/demo"
  ]
}
```

**自定义 preset**

```
module.exports = function(){
  return {
    plugins: [],
  };
}
```

###### env

env 通过配置，设置目标环境，从而根据需要转换代码

```
{
  preset: {
    [
      [
        "@babel/env",
        {
          targets: {
            edge: "17",
            firefox: "60",
            chrome: "67",
            safari: "11.1",
          },
          useBuiltIns: "usage", // 解决 polyfill 的问题
        },
      ],
    ];
  }
}
```

###### polyfill

@babel/polyfill：用于模拟完整的 es2015 环境，但缺点是

1. 打包代码体积较大；
2. 会污染全局变量;

所以推荐使用`@babel/plugin-transform-runtime @babel/runtime`（<font color="red">最新提供"useBuiltIns"参数</font>）

##### plugins

插件机制: 对 AST 进行操作，增删改 AST 节点

**执行顺序**: preset 按照既定顺序，**从前往后执行**

```
// babel.config.js

module.exports = {
  plugins: [
    // 如果插件名称的前缀为 babel-plugin-，你还可以使用它的短名称
    "babel-plugin-demo",
    [ "babel-plugin-demo",options],
    "demo",
    "./babel-plugin-demo"
  ]
}
```

#### Babel 插件

##### AST

1. 语法分析树：准确表达整个文本、源代码，基本上说是文本的直接翻译
2. 抽象语法树：简单来说就是语法树的压缩，省略不重要的部分，将树精简，保留重要部分。一般来说都是先生成 CST，再转换为 AST

例如：

```
5\*(1+2)

{
  Exp: [
    {
      Exp: [
        {
          val: 5,
        }
      ]
    },
    {
      val: '*'
    },
    {
      Exp: [
        {
          val: '('
        },
        {
          Exp: [
            {
              Exp: [
                {
                  val: 1,
                }
              ]
            },
            {
              val: '+'
            },
            {
              Exp: [
                {
                  val: '2',
                }
              ]
            }
          ]
        },
        {
          val: ')'
        }
      ]
    }
  ]
}

向这样的语法树中，很多Exp是没有必要的，例如单后继节点（5，1，2）只是计算值，没有必要使用Exp，进行类似的简化后就可以形成AST

[
  {
    val:5,
  },
  {
    type:'*',
  },
  {
    val:1,
  },
  {
    type:'+'
  },
  {
    val:2,
  }
]

自下向上构建
```

Babel 插件是使用 Visitor 语法，在深度遍历整个 AST 树时，访问每个节点，操作节点

#####

##### Visitor

以 visitor（访问者）来访问，在深度遍历中的每个节点；在遍历过程中，如果遇到和 visitor 中相匹配的节点，则执行回调

```
function babelPlugin() {
  return {
    visitor: {
      // 以 visitor 对象中的属性名 和 节点type 比较，如果相等则触发回调

      // 省略表示，默认为enter
      Identifier(path, state) {
        console.log(path, state);
      },
      // 详细表示
      Identifier:{
        enter() {
          console.log('enter');
        },
        exit() {
          console.log('exit');
        }
      },
      // 多重
      "VariableDeclarator|Identifier"(){
      }
     // 获取使用别名，例如 Function 是 FunctionDeclaration，FunctionExpression，ArrowFunctionExpression，ObjectMethod，ClassMethod 的别名
    },
  };
}
```

##### Path

节点信息 + 节点路径信息，用于表示两个节点之间联系，在 visitor 中，以回调第一个参数传入，且在 path 中，还可以继续执行 Visitor 模式访问

```
const test = [1, 2, 3];

function babelPlugin() {
  return {
    visitor: {
      Identifier(path, state) {

        // 找到声明为test的path
        if (path.isIdentifier({ name: 'test' })) {

          // 并对它的父path再次进行visitor模式访问
          path.parentPath.traverse({

            // visitor
            ArrayExpression(path) {
              console.log(path)
            },
          });
        }
      },
    },
  };
}
```

##### State

存储当前状态，包含插件信息，插件传参，配置参数等一系列信息，做为 Visitor 模式中回调参数的第二个参数

##### Scopes

作用域，处理 AST 时需要考虑作用域的问题

```
function square(n) {
  return n * n;
}
const n = 2;
```

次数存在 3 个不同的作用域，处理时不能混淆，不能直接暴力 `Identifier(){}`来操作，得借助 path 内部使用 Visitor 模式

```
function babelPlugin() {
  return {
    visitor: {
      FunctionDeclaration(pathFun) {
        pathFun.traverse({
          BinaryExpression(pathBin) {
            pathBin.traverse({
              Identifier(path) {
                // 在此处操作 n*n的节点
              },
            });
          },
        });
      },
    },
  };
}
```

#### Babel 自定义语法

```
function @@f(){} // 以双@@代表柯理化函数，自动实现柯理化
```

##### 自定义 babel-parse

1. git clone 代码 `https://github.com/babel/babel.git`
2. 进入 `packages/babel-parser`
3. 增加新 token，修改解析方式

```
// packages/babel-parser/src/tokenizer/types.js 增加新的token

...
at: new TokenType("@"),
curry: new TokenType("@@"), // 在at标记下增加新的标识，curry
...
```

```
// packages/babel-parser/src/tokenizer/index.js 新token的解析方式

...

getTokenFromCode(code: number): void {

  ...
  case charCodes.atSign: // atSign 对应的code为64，对应字符为"@"

    // 匹配@@，则为curry
    if (this.input.charCodeAt(this.state.pos + 1) === charCodes.atSign) {
      this.finishOp(tt.curry, 2);
    } else {
      // 否则为at
      this.finishOp(tt.at, 1);
    }
    return;
  ...
}
...
```

```
// packages/babel-parser/src/parser/statement.js 消耗掉新token，作为函数Node属性curry

...
parseFunction<T: N.NormalFunction>(
    node: T,
    statement?: number = FUNC_NO_FLAGS,
    isAsync?: boolean = false,
  ): T {
    ...
    node.generator = this.eat(tt.star);
    node.curry = this.eat(tt.curry); // 此处增加curry标记，且消耗掉令牌，如果不消耗令牌，则会报错
    ...
}
...
```

`make build 或者 npm run build`，构建 babel，babel-parse/lib/index.js 为我们要的自定义 parse

此步骤后，自定义 parse 完成，可以尝试使用如下代码验证

4. 将 babel-parse/lib/index.js 复制到工程 _custom-babel-plugin/lib/parser_ 目录下

##### 插件开发

1. 安装依赖

```
npm i -S @babel/types @babel/helpers @babel/template
npm i -D @babel/core jest
```

```
// template.js 柯理化模板

const helpers = require('@babel/helpers/lib/helpers');
const { helper } = require('./index');

helpers.default.curry = helper('7.2.6')`
  function curry(fn) {
    const len = fn.length;
    let args = [];

    return function handle(...arg) {
      args = args.concat(arg);
      if (args.length >= len) {
        return fn(...args);
      } else {
        return handle;
      }
    }
  }
  export default function handle(fn){
    return curry(fn);
  }
`;


// index.js
const { variableDeclarator, variableDeclaration, callExpression, identifier, toExpression } = require('@babel/types');
require('../util/template');
const parse = require('./parser');

module.exports = function customPlugin() {
  return {
    parserOverride(code, opt) {
      return parse.parse(code, opt);
    },
    visitor: {
      FunctionDeclaration(path) {
        const node = path.node;

        if (node.curry) {
          path.replaceWith(
            variableDeclaration(
              "const",
              [
                variableDeclarator(
                  identifier(path.get('id').node.name),
                  callExpression(
                    // identifier('curry'),
                    this.addHelper('curry'),
                    [
                      toExpression(node),
                    ]
                  )
                )
              ]
            )
          );
        }
      }
    }
  }
}
```

## Typescript

## Node

### API

#### querystring

对 URL ? 号后的字符串系列化和反序列化

```
queryString.parse('x=1&y=2'); // {x:1,y:2};
queryString.stringify({x:1,y:2}); // x=1&y=2
querystring.esacpe(str) ===encodeURI(str)
```

#### readline

node 逐行读取文件

```
const readline = require('readline');
const fs = require('fs');
const rl = readline.createInterface({ input: fs.createReadStream('./code.js'), output: process.stdout, });

rl.on('line', d => { console.log(d); console.log('---') });

rl.question('?',ans=>{

});

```

#### repl

node 命令行操作窗口，类似于在 shell 中输入 node 后的交互式命令窗口。*读取-求值-输出*的循环

### C++扩展

#### Nan

##### 函数参数类型

```
Nan::FunctionCallbackInfo
Nan::PropertyCallbackInfo
Nan::InfoValue

NAN_METHOD // 函数声明宏


// 访问器
NAN_GETTER // property, info
NAN_SETTER // property, value, info

Nan::SetAccessor

// 拦截器
NAN_PROPERTY_GETTER
NAN_PROPERTY_SETTER
NAN_PROPERTY_ENUMERATOR
NAN_PROPERTY_QUERY
NAN_PROPERTY_DELETER

NAN_INDEX_GETTER
NAN_INDEX_SETTER
NAN_INDEX_ENUMERATOR
NAN_INDEX_QUERY
NAN_INDEX_DELETER

Nan::SetNamedPropertyHandler
Nan::SetIndexedPropertyHandler


// 函数定义
Nan::New<v8::FunctionTemplate>() // 函数模板 v8::FunctionTemplate::New(iso);
Nan::SetMethod // 给某个对象设置方法
Nan::SetPrototypeMethod // 设置原型方法

Nan::Set // 设置，类似于 exports->Set


// 作用域

Nan::HandleScope handleScope; // 等于 v8::HandleScope scope(iso);
Nan::EscapableHandleScope escScope; // 等于 v8::EscapableHandleScope escScope(iso);

Nan::Persistent<v8::String> pers;
NAN_METHOD(Echo) {
  v8::Local<v8::String> str =
      Nan::New<v8::String>("HELLO WORD").ToLocalChecked();

  pers.Reset(str); // 升级持久句柄
}

// 创建元数据
Nan::New // 创建(布尔值，数字)=> 本地句柄 ；字符串=>待确定本地句柄，所以需要调用ToLocalChecked();

// 正则
Nan::New<v8::RegExp>("a", "g");

// Nan::New 生成的待实本地句柄
v8::String
v8::Date
v8::Regexp
v8::Script
v8::UnboundScript

// 快捷方法
Nan::Undefined();
Nan::Null();
Nan::True();
Nan::False();
Nan::EmptyString();

// 类型转换
As<T>()    // v8::Local<v8::Number> num = info[0].As<v8::Local<v8::Number>>();
Nan::To<v8::Value>() // v8::MaybeLocal<v8::Number> num = Nan::To<v8::Number>(info[0]);

// 字符串
v8::MaybeLocal<v8::String> mb_c = Nan::New<v8::String>("HELLO WORD");
v8::Local<v8::String> l_c = mb_c.ToLocalChecked();
Nan::Utf8String val(l_c);
string s(*val);

// 对象 返回值均为 MaybeLocal、Maybe
Nan::Set
Nan::ForceSet
Nan::Get
Nan::GetPropertyAttributes
Nan::Has
Nan::Delete
Nan::GetPropertyNames
Nan::GetOwnPropertyNames
Nan::SetPrototype
Nan::ObjectProtoToString
Nan::HasOwnProperty
Nan::HasRealNamedProperty
Nan::HasRealIndexedProperty
Nan::HasRealNamedCallbackProperty
Nan::GetRealNamedPropertyInPrototypeChain
Nan::GetRealNamedProperty
Nan::CallAsFunction
Nan::CallAsConstructor
Nan::HasPrivate
Nan::GetPrivate
Nan::SetPrivate
Nan::DeletePrivate

// 数组
Nan::CloneElementAt
Nan::ToArrayIndex

v8::Local<v8::Array> arr = info[0].As<v8::Array>();
v8::Local<v8::Array> arr = v8::Local<v8::Array>::Cast(info[0]);

// json
Nan::JSON json;
v8::Local<v8::Value> obj =
    json.Parse(info[0].As<v8::String>()).ToLocalChecked();

info.GetReturnValue().Set(obj);
```

##### 函数回调

```
Nan::Call
Nan::MakeCallback

NAN_METHOD(Add) {
  int i = info[0].As<Number>()->NumberValue();
  int j = info[1].As<Number>()->NumberValue();
  info.GetReturnValue().Set(Nan::New<Number>(i + j));
}

NAN_METHOD(Plus) {
  Local<Function> fn = Nan::New<FunctionTemplate>(Add)->GetFunction();

  const int argc = 2;
  Local<Value> args[2] = {};

  for (int i = 0; i < info.Length(); i++) {
    args[i] = info.operator[](i);
  }

  Local<Value> res =
      Nan::Call(fn, info.This(), info.Length(), args).ToLocalChecked();

  info.GetReturnValue().Set(res);
}
```

##### Buffer

```
Nan::MaybeLocal<Object> buf = Nan::NewBuffer(uint len);
Nan::MaybeLocal<Object> buf = Nan::NewBuffer(char* str,uint len);
Nan::MaybeLocal<Object> buf = Nan::NewBuffer(char* str,uint len,Nan::FreeCallback callback,void *hint); // NewBuffer的内存应该通过V8管理，而不能使用释放；或者在FreeCallback函数内处理

Nan::MaybeLocal<Object> buf1 = Nan::CopyBuffer(const char* data, uint len);

```

##### 异常机制

```
Nan::Error
Nan::RangeError
...
Nan::ThrowError
```

##### 异步

**Nan::AsyncWorker**

```
new Nan::AsyncWorker(Nan::Callback)：Nan::Callback，封装勒Nan::Function，将句柄封装在内部，防止v8回收，作为回调函数，进行异步调用

Nan::AsyncWorker::Execute 函数，虚函数，在 libuv 唤起线程时，执行的即是该函数，该函数虽然为异步函数，但是Libuv线程池线程有限，一般默认为4个，所以任务过重也会影响性能；且该函数在Worker执行，不能访问V8及其数据对象，只能使用C++的数据

Nan::AsyncWorker::SetErrorMessage：在执行时，可以通过SetErrorMessage来返回错误，Nan会调用HandleOKCallback或HandleErrorCallback来回调

class MyWorker : public Nan::AsyncWorker {
 public:
  void Execute();

  MyWorker(Nan::Callback *callback) : Nan::AsyncWorker(callback) {}
};

void MyWorker::Execute() {
  for (int i = 0; i < 5000; i++) {
    std::cout << i << std::endl;
  }
}

NAN_METHOD(Calc) {
  Local<Function> fn =
      FunctionTemplate::New(Isolate::GetCurrent())->GetFunction();

  Nan::AsyncQueueWorker(new MyWorker(new Nan::Callback(fn)));
}

void Nan::AsyncQueueWorker(Nan::AsyncWorker *worker);
```

**Nan::AsyncProgressWorker**

#### libuv 编程

在 libev 基础上开发，移除 libev 依赖。为 Node 提供**事件循环、非阻塞文件 I/O、非阻塞网络 I/O、平台兼容、提供以流和句柄的方式对 I/O、Socket 等的抽象**

先要安装 libuv，且在编译时，需要将 libuv 依赖加入 `g++ main.cpp /usr/local/include/lib.a -o main`

```
int main() {
  uv_loop_t *loop = uv_loop_new();

  cout << "now doing" << endl;

  uv_run(loop, UV_RUN_DEFAULT);
  uv_loop_close(loop);

  return 0;
}
```

##### idling

空转句柄，在每次时间循环初都会被执行，(Node 时间循环中的 idle)

##### 线程编程

###### libuv 跨线程

libuv 基于事件循环和回调的操作是可预料的，但是 libuv 线程时不可预知的，甚至完全脱离事件循环，他们在需要时可以**被阻塞**

```
#include <stdio.h>
#include <uv.h>
#include "nan.h"
#include "node.h"
#include <iostream>
#include <vector>

#ifdef WINDOWS
#include <windows.h>
#else

#include <unistd.h>

#endif

namespace __Demo__ {
  using v8::Local;
  using v8::Array;
  using v8::FunctionCallbackInfo;
  using v8::Value;
  using v8::Object;
  using v8::Number;
  using namespace std;

  struct Args {
    uint val;
    vector<uint> *rec;
  };

  void Sleep(void *arg) {
    Args *args = (Args *) arg;
    usleep(args->val * 1000000);
    args->rec->push_back(args->val);

    cout << args->val << endl;
  }

  NAN_METHOD(Sort) {
    // Info[0] 接收一个数组
    if (info.Length() < 1 || !info[0]->IsArray()) {
      return Nan::ThrowError("请输入数组参数");
    }

    Local<Array> arr = info[0].As<Array>();
    uint len = arr->Length();
    vector<uint> recv_arr(len, 0);
    for (uint i = 0; i < arr->Length(); i++) {
      if (!arr->Get(i)->IsUint32()) {
        Nan::ThrowError("数组内部必须为正整数");
      }
      recv_arr[i] = Nan::To<uint>(arr->Get(i)).FromJust();
    }

    vector<uv_thread_t> uv_thread(len); // 线程句柄
    vector<uint> rec;

    for (uint i = 0; i < len; i++) {
      Args *args = new Args();
      args->val = recv_arr[i];
      args->rec = &rec;

      // 进入libuv线程
      uv_thread_create(&uv_thread[i], Sleep, args);
    }

    for (uint i = 0; i < len; i++) {
      uv_thread_join(&uv_thread[i]);
    }

    Local<Array> res = Nan::New<Array>();
    for (uint i = 0; i < len; i++) {
      res->Set(i, Nan::New<Number>(rec[i]));
    }
    info.GetReturnValue().Set(res);
  }


  NAN_MODULE_INIT(Init) {
    Nan::Export(target, "sort", Sort);
  }

  NODE_MODULE(addon, Init)
}
```

**代码在存在巨大问题，如果排序 [1,1,1,2,2] 这样的数据，则会在同时访问内存空间，形成资源抢占，导致进程崩溃**

###### 互斥锁

```
uv_mutex_t： 句柄类型

uv_mutex_init()
uv_mutex_lock()
uv_mutex_trylock()
uv_mutex_unlock()
uv_mutex_destroy()


uv_mutex_t lock_handle;
uv_mutex_init(&lock_handle);
uv_mutex_lock(args->lock_handle);
  args->rec->push_back(args->val); // do somethings
uv_mutex_unlock(args->lock_handle);
```

###### 读写锁

```
uv_rwlock_t

uv_rwlock_init()
uv_rwlock_rdlock()
uv_rwlock_tryrdlock()
uv_rwlock_rdunlock()

uv_rwlock_wrlock()
uv_rwlock_trywrlock()
uv_rwlock_wrunlock()

uv_rwlock_t rwlock;
uv_rwlock_init(&rwlock);
uv_rwlock_wrlock(args->lock_handle);
args->rec->push_back(args->val); // 做些事情
uv_rwlock_wrunlock(args->lock_handle);
```

###### 信号量

通过生成令牌授权来使同一时间只有一个线程可以访问资源

信号量具有原子性，且只能进行等待（P），和发送（V），最简单信号量只有 1、0

```
uv_sem_t sem;
uv_sem_init(&sem, 1); // 初始为1，表示只有一个可以访问

uv_sem_post();
uv_sem_wait()
uv_sem_destroy()
uv_sem_trywait()


uv_sem_t sem;
uv_sem_init(&sem, 1)
uv_sem_wait(args->lock_handle); // 信号量减1
args->rec->push_back(args->val);// do somethings
uv_sem_post(args->lock_handle);//信号量加1
```

###### 工作队列

uv_queue_work：Nan::AsyncWorker 就是以此实现的

```
uv_queue_work(uv_loop_t* loop, uv_work_t* req, uv_work_cb work_cb, uv_after_work_cb after_work_cb)
```

##### 线程通信

uv_async_t 允许从别的线程唤起事件循环，并在事件循环中执行它指定的回调函数

```
uv_async_init(uv_loop_t*, uv_async_t* async, uv_async_cb async_cb);
uv_async_send(uv_async_t* async);// 该函数唤起事件循环，且会合并唤起行为后执行，如果再执行回调前发起5次调动，则会合并成1次
```

### NPM

## C++

## 网络

## 算法
