# July-2

## Javascript

### Javascript 不用 string 方法转换大小写

#### JS API

```typescript
// API
String.prototype.toUpperCase;
String.prototype.toLowerCase;
```

#### 运算

```typescript
// 通过运算 a <=> 97 A <=> 65
// 1. 通过 加减运算，但有个致命缺点，就是无法转换 'Ab' 这样的字符，需要增加额外判断

function toUpperCase(s: string): string {
  let res = '';

  for (let i = 0; i < s.length; i++) {
    res += String.fromCharCode(s.charCodeAt(i) - 32);
  }
  return res;
}
```

```typescript
/*
97 | 32-- > 97
65 | 32-- > 97

97 & 223 --> 65
65 & 223 --> 65
*/

function toUpperCase(s: string): string {
  return String.fromCodePoint(
    ...Array.from(s).map((item, ind) => s.codePointAt(ind) & 223)
  );
}

function toLowerCase(s: string): string {
  return String.fromCodePoint(
    ...Array.from(s).map((item, ind) => s.codePointAt(ind) | 32)
  );
}
```

### call、apply、bind

```typescript
const newBind = Function.prototype.call.bind(Function.prototype.bind);
```

Function.prototype.call，Function.prototype.bind

存在两个参数：

1. thisArg：this 传参
2. argArray： 后续参数传值

别忘了，由于 call、bind 都是函数，本身也存在 this，这个表示谁调用了 call 和 bind 方法，**千万不要和 thisArg 搞混了**

我们分析 `const newBind = Function.prototype.call.bind(Function.prototype.bind)`

```text
Function.prototype.call.bind 返回一个函数，thisArg 传入了 Function.prototype.bind 做为 ，表示由 Function.prototype.bind 调用 call 函数。则 newBind(...args) 调用时，相等于 ==> Function.prototype.bind.call(...args);

这里举个栗子 newBind(Array.prototype.slice,[]);

1. Function.prototype.bind 必须由函数调用，例如 fn.bind(...)，但此时是由 call 传入 thisArg，则 args 第一个参数必须为函数，否则无法调用 bind 函数，栗子传入了 slice 函数
2. call 函数就剩余的参数传入，此时相当于 slice.bind([]); 由于 bind 函数第一个参数也是 thisArg，表示使用 thisArg 调用 slice

则综合起来说 newBind(Array.prototype.slice, []); ==> Array.prototype.slice.bind([]);
```

`Function.prototype.call.bind(Function.prototype.bind)` 其实就是一个拆分参数

## Node

### npm 和 node 对 \*_/_ 的解析

```shell
# node 版本 12
# 操作系统 mac

node main ./*/** # 解析所有的嵌套文件


# package.json scripts 中 { "main": "node main ./*/**" }
npm run main # 只解析了第一层嵌套，相等于 node main ./*.*
```

两种运行结果完全不同

## LeetCode

### 不同路径

```typescript
function uniquePathsWithObstacles(obstacleGrid: number[][]): number {
  // 动态规划
  // 思路，格子只能向下移动，则每一个格子，都存在两种到达方式，从上到达、从左到达
  // 则 dp[i][j] = dp[i-1][j] + dp[i][j-1]
  // 但格子可能障碍物，无法到达，则 dp[i][j] = marix[i][j] == 1 ? 0 : dp[i-1][j] + dp[i][j-1]
  // 注意溢出

  if (obstacleGrid[0][0] === 1) return 0; // 第一个格子被堵了，直接为0

  const store = new Array(obstacleGrid.length);

  for (let i = 0; i < obstacleGrid.length; i++) {
    store[i] = [];
    for (let j = 0; j < obstacleGrid[0].length; j++) {
      if (i == 0 && j === 0) {
        store[i][j] = 1;
        continue;
      }

      if (obstacleGrid[i][j] === 1) {
        store[i][j] = 0;
        continue;
      }

      let sum = 0;

      if (i > 0) {
        sum += store[i - 1][j];
      }

      if (j > 0) {
        sum += store[i][j - 1];
      }

      store[i][j] = sum;
    }
  }

  return store[obstacleGrid.length - 1][obstacleGrid[0].length - 1];
}
```

[不同路径 II](https://leetcode-cn.com/problems/unique-paths-ii/)
