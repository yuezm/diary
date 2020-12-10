# Aug-4

## Javascript

### requestIdleCallback/cancelIdleCallback

```javascript
window.requestIdleCallback(callback[, options])
/*
callback:
options:
  timeout: 毫秒，当超过该数未调用回调函数时，则在浏览器空闲时，强制调用该回调函数
*/

cancelIdleCallback(handle); // 取消 requestIdleCallback
```

requestIdleCallback/cancelIdleCallback API 将在浏览器空闲时间调用回调函数，这使得开发者可以在主事件循环上执行后台和优先级较低的代码，从而不会影响关键事件。

**建议**

1. 强烈建议使用 timeout 选项进行必要的工作，否则可能会在触发回调之前经过几秒钟
2. requestIdleCallback 建议不要使用 DOM 操作，requestIdleCallback 为空闲时间段执行回调函数，在此函数执行时，说明浏览器已经完成了重绘、重排等操作，如果此时再去修改 DOM 的话，浏览器不得不再进行重排、重绘操作
3. 不建议在 requestIdleCallback 使用 Promise, Promise 属于微任务，将会在 requestIdleCallback 回调函数完成后立即执行，从而延长了该回调函数时间，可能会阻塞其他关键事件

### 浏览器指纹

以下介绍仅为原理，生成请使用成熟的包 [fingerPrintJs](https://github.com/fingerprintjs/fingerprintjs)

**浏览器指纹**：通过浏览的设置信息，网站配置来跟踪WEB浏览器（用户）的方法。之所以称之为"指纹"，是因为在常规情况下，期望浏览器指纹是和人的指纹一样，是唯一的

浏览器指纹辨识度可以为：IP地址，时区，地理位置（经纬度），User-Agent，语言，操作系统...

#### 追踪用户的技术实现

1. 通过session和cookie追踪，是基于用户登录
2. 浏览器指纹
3. 基于用户的行为（浏览习惯、设置...）、习惯建立特征值、模型

#### 普通指纹

1. IP
2. HTTP Headers 中的各种信息
3. ...

#### 高级指纹

1. Canvas 指纹：利用 Canvas 在每个浏览器，每个机器的渲染参数、抗锯齿等算法不同，生成的图片DataURI也不同

  ```javascript
  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d');

  context.font = '18px Arial';
  ctx.fillText('Hello word', 2, 2);

  canvas.toDataURL('image/jpeg');
  ```

2. AudioContex：利用硬件差异，生成的不同的音频输出

3. WebRTC: 网页实时通信，可以让浏览器音视频或其他任意数据的实时传输，通过 WebRTC 的能力，可以获取用户真实 IP(NAT 穿透)

4. ...

#### 跨浏览器指纹

在以上浏览器指纹中，有一个很大的缺陷，就是用户_更换浏览器或更换电脑_后，无法追踪，此时可以使用如下参数进行修正

1. Task(a)~Task(r): 利用显卡渲染图片的功能的特征值
2. List of fonts（JS）: 获取页面字体支持的情况
3. TimeZone: 时区
4. CPU 核心数: 可通过 navigator.hardwareConcurrency，或者其他的 polyfill core-estimator，大致原理是借用 Web Worker 能力，监听 payload 时间，计算硬件达到最大的并发时间，以计算内核数量
5. ...

#### 防范

1. 禁用 JS
2. 浏览器插件，例如 Tor Browser 工具
3. HTTP 头部，DNT 标识，Do Not Track，但大多数网站并没有遵守此约定
4. ...

### Node.js 获取用户IP

通过用户的请求头获取 IP

1. remoteAddress：客户端真实IP，但是经过代理服务器之后，会修改为代理服务器IP地址 `req.socket.remoteAddress`
2. X-Forwarded-For：客户端真实IP，在经过代理服务器后，会添加代理服务器IP，例如 `IP<,Proxy2><,Proxy2>`
3. X-Real-IP：客户端真实IP，经过代理服务器后也为真实IP，但并不是标准，所以服务器可能遵守也可能不遵守

通过用户的请求头获取 Host

1. host：指出请求的主机名和端口
2. X-Forwarded-Host：和X-Forwarded相似

## CPP

### 排序

```c++
#include<algorithm>

// sort: 对给定区间排序
// stable_sort: 给定区间稳定排序
// partial_sort: 给定区间部分排序
// partial_sort_copy: 给定区间复制排序
// nth_element: 给定区间某个对应位置元素
// is_sorted: 是否排好序
// partition: 使得某个符合的元素放到前面
// stable_partition: 相对稳定的使得某个符合条件元素放到前面
```

```c++
// equal_to: 相等
// not_equal_to: 不相等
// less
// less_equal
// greater
// greater_equal

// 示例

vector<int> arr;
sort(arr.begin(), arr.end(), less<int>());
```

模板内部排序比较时，不允许使用 "<=、>="，必须使用" <、>"，即less或greater

### map

```c++
#include<map> // map 红黑树存储
#include <unordered_map> // hash_map
```

### vector

```c++
// vector 拷贝

// 循环

// copy 函数
#include <algorithm>
#include <vector>

copy(source.begin(), source.end(), back_inserter(target)); // 拷贝函数
```

### 二维数组

```c++
int arr[10] = {0}; // 全部置为0，否则可能出现任意数
int arr[10][20] = {0}; // 全部置为0，否则可能出现任意数

int* arr = new int[20]();
int* arr = int[VAR]{...};
int* arr = new int[20]{};
int** arr = new int*[VAR]; // 二维数组
```

## LeetCode

### 两个数组的交集

思路1：map作为存储，存储已经出现过的

```typescript
function intersect(nums1: number[], nums2: number[]): number[] {
  if (nums1.length === 0 || nums2.length === 0) {
    return [];
  }

  const store = new Map<number, number>();
  const result: number[] = [];

  if (nums2.length > nums1.length) {
    const s = nums1;
    nums1 = nums2;
    nums2 = s;
  }

  for (const item of nums2) {
    store.set(item, store.has(item) ? store.get(item)! + 1 : 1)
  }

  for (const item of nums1) {
    if (store.has(item) && store.get(item)! > 0) {
      result.push(item);
      store.set(item, store.get(item)! - 1);
    }
  }

  return result;
};
```

思路2：排序，如果是有序的情况下，右侧肯定比左侧大，类似于归并排序

```typescript
function intersect(nums1: number[], nums2: number[]): number[] {
  nums1.sort((a, b) => a - b);
  nums2.sort((a, b) => a - b);

  const result: number[] = [];
  let i = 0, j = 0;


  while (i < nums1.length && j < nums2.length) {
    if (nums1[ i ] === nums2[ j ]) {
      result.push(nums1[ i ]);
      i++;
      j++;
    } else if (nums1[ i ] < nums2[ j ]) {
      i++;
    } else {
      j++;
    }
  }
  return result;
};
```

[两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays-ii/)

### 异步串行

题目：

```typescript
// 使用，串行执行
const delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms));
const subFlow = createFlow([() => delay(1000).then(() => log('c'))]);

createFlow([
  () => log('a'),
  () => log('b'),
  subFlow,
  [() => delay(1000).then(() => log('d')), () => log('e')],
]).run(() => {
  console.log('done');
});
```

1. 整理参数和返回值：createFlow 数组参数

  - 参数可以为函数
  - 参数可以为subFlow返回值
  - 参数还可以为数组

    - 数组内部可以为 Promise
    - 数组内部可以为函数

2. createFlow 返回值存在 run 方法，且返回值可以接受数组为参数

3. 在执行 run 方法后，依次执行参数，但参数有同步和异步的，那么就全部处理为异步（可以全部处理为Promise）

```typescript
interface runInterface {
  (callback?: () => void): Promise<any>;
}

type createFlowReturnType = { run: runInterface };
type createFlowParameterType = () => void | createFlowReturnType | Promise<any>;

function createFlow(params: (createFlowParameterType | createFlowParameterType[])[]): createFlowReturnType {
  /**
   * 1.由于参数是多种类型，所以会对参数进行判断，譬如函数，Promise，带run的对象...
   * 2\. 需要触发下一个函数的执行时机
   *    - 对于函数来说： 函数运行完
   *    - 对于Promise来说：then
   *    - 对于 run 对象：run 运行完，可以让 run 返回 Promise，来进行统一控制
   *    - 对于数组：数组内部运行完（可以调用createFlow，在run后运行）
   * */

  async function recursion(i: number = 0): Promise<void> {
    if (i === params.length) return;
    await runner(params[ i ]);
    await recursion(i + 1);
  }



  return {
    async run(callback?: () => void): Promise<void> {
      await recursion(0);
      callback && callback();
    }
  }
}

async function runner(params: createFlowParameterType | createFlowParameterType[]): Promise<void> {
  if (Array.isArray(params)) {
    await createFlow(params).run();
  } else {
    // 判断参数
    if (typeof params === 'function') {
      params();
    } else if ((params as any) instanceof Promise) {
      await params;
    } else {
      await (params as createFlowReturnType).run();
    }
  }
}
```
