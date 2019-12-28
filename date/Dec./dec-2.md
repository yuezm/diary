# 11 月第 4 周

## Javascript

### 协程

_进程_：操作系统对运行程序的抽象，是计算机分配资源的最小单位，不同线程互相隔离；
_线程_：¬ 是操作系统调用的最小单位，是程序执行的最小单位，同个进程下的线程共享资源；
_协程_：也称微线程，以线程形式运行，属于线程；

**并发演变（自己整理的简略版）**：

_进程_：在单核（单核单进程）计算机上，不存在并发，操作系统利用**时间片轮转（RR 调度）**来模拟并发。时间片轮转简单来说就是执行一会 A ，暂停，将 A 状态保存，执行一会 B ，保存 B 的状态，再次执行 A...，由于操作系统快速切换，看起来就像并发。在现代计算上（多核多进程），当执行线程数小于计算机进程数时（非指 CPU 核心数，现代计算机利用超线程技术，实现多核多进程，例如 4 核 8 线程），是真正意义的并发，每个线程运行于不同的核心，同时运行，但是当进程大于计算机进程数时，必然存在线程运行多个程序，此时利用时间片轮转实现假并发。

_线程_：多个线程可运行同一个进程下，但线程没有正真意义的并发，也是利用**时间片轮转**实现并发，但线程之间切换要比进程切换消耗小很多

_协程_：多个协程可运行于同一个线程下，但协程之间是由**用户实现协程切换，而非操作系统**，**不存在系统调用，无上下文切换**，效率较高；且运行于同一个线程，避免竞争关系使用锁

## Typescript

## Node

### 性能钩子

实验性 API，监测性能，例如测量函数运行时间

```typescript
import {
  Performance,
  PerformanceEntry,
  PerformanceObserver,
  performance,
} from 'perf_hooks';

function test() {
  console.log('TEST');
}

const obs = new PerformanceObserver(list => {
  console.log(list.getEntries()[0]);
  obs.disconnect();
});

obs.observe({ entryTypes: ['function'] });

performance.timerify(test)();
```

### process

主进程抽象

**主进程事件**

部分事件未捕获，则最终会传递到 process，如

- rejectionHandled：在下一个事件循环内未补货 Promise Rejected，可能是由于 Promise 在未来某个时间点捕获，比事件循环稍晚
- unhandledRejection：未捕获的 Promise Rejected
- uncaughtException：未捕获的错误，此时程序已经崩溃，不能用作异常处理，应该直接杀死或重启（如 pm2）

### Node 最佳实践

#### 环境变量建议

1. 不在工程维护单独的环境变量，例如 env.dev、env.test、env.prod
2. 在外部单独维护，例如在单独工程中配置不同环境的变量

#### 模块划分建议

1. 模块以业务功能划分，不以角色划分（不以 service,controller 划分），例如

```
- user
  - user.controller.js
  - user.service.js
  ...
```

#### 错误处理

区分为**操作错误、程序错误**

1. 操作错误：用户操作错误，仅仅提供日志记录
2. 程序错误
   - 致命程序错误：例如 uncaughtException，应该直接重启
   - 普通程序错误：例如 unhandledRejection，以 error 等级 记录

**建议不要在公共中间件中处理错误，可以使用单独的错误捕获方法**

**stack-trace**

在 Javascript 中以，`console.log(Error.stack)`或者`console.trace(Error)`，大部分都实现了

**v8 stack-trace**

在 node 中，错误栈信息一般为 10 帧，可以通过如下来改变

```
Error.stackTraceLimit = 10

--stack-trace-limit <value>
```

具体错误追踪[]()

### C++扩展

### NPM

## C++

## 网络

### HTTP

#### HTTP 代理、网关

#### HTTP 头部

#### HTTPS、SPDY、HTTP2

## 算法

### 二叉树

#### 深度优先

**前序遍历**

**中序遍历**

**后续遍历**

#### 广度优先

**层次遍历**

#### 前序遍历、后续遍历+中序遍历

## LeetCode
