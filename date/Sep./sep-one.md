# 9 月第一周

## 排序

### 选择排序

1. 循环数组，找出剩余数组的最小值
2. 将最小值与当前位置的值交换

```
function select_sort(arr) {
  for (let i = 0; i < arr.length; i++) {
    const index = find_min(arr, i, arr.length - 1);
    exchange(arr, i, index);
  }
  return arr;
}
```

**优点**

1. 代码实现简单
2. 运行时间与输入无关
3. 元素交换次数较少

**缺点**

1. 时间复杂度较大为 $O(n^2)$
2. 运行时间与输入无关，即使输入为有序数据，也是同样的运行时间

### 插入排序

1. 保证左边是元素是有序的
2. 将当前位置元素插入到左边合适位置中，并保证元素依旧有序

```
function insert_sort(arr) {
  for (let i = 0; i < arr.length; i++) {
    let j = i;

    // 方法1：连续与前一位元素比较，以确定元素位置
    while (arr[ j - 1 ] > arr[ j ] && j > 0) {
      exchange(arr, j, --j);
    }

    // 方法2：找到元素合适的位置，把大数右移，并将元素插入到该位置,
    int index = find_index(arr,0,i);
    const v = arr[i];

    // 该步骤可以直接合并为插入元素
    exchange(arr,i,i-1);
    exchange(arr,i-1,i-2);
    ...
    exchange(index+1,index);

    arr[index] = v;
  }
  return arr;
}
```

**优点**

1. 代码实现简单,最坏时间复杂度为 $O(N^2)$
2. 如果元素部分有序，排序时间较快；比如元素是有序的，则时间复杂度为$O(N)$

**缺点**

1. 如果元素完全无序，或者大规模乱序，任然很慢
2. 元素交换次数较多

### 希尔排序

h 有序数组

1. 找到间隔 h
2. 将数组按照 h 分割为多个小数组，并将小数组排序
3. 最后将大数组插入排序

举个栗子

```
原始数据为 [1,5,6,2,4,3];
当 h = 3 时，分割为[1,2] [5,4] [6,3],保证三个小数组有序，排序后得到数组为 [1,2],[4,5],[3,6] => [1,4,3,2,5,6]
当 h = 1 时，分割为 [1,4,3,2,5,6], 排序后得到数组为 [1,2,3,4,5,6]
```

```
function shell_sort(arr) {
  let h = Math.trunc(arr.length / 2);
  while (h > 0) {
    for (let i = h; i < arr.length; i++) {
      let j = i;
      while (arr[ j - h ] > arr[ j ] && j >= h) {
        exchange(arr, j, j - h);
        j -= h;
      }
    }
    h = Math.trunc(h / 2);
  }
  return arr;
}
```

**优点**

1. 插入排序在部分有序的情况下表现良较好，所以，希尔排序效率较好
2. 在大规模数据下或乱序下，表现情况较好，最坏时间复杂度为 $O(N^{3/2})$
3. 在极端情况下，例如最小值在最右侧，表现比插入排序好

**缺点**

1. 代码较多，理解也比插入排序和选择排序复杂
2. 算法性能还取决于 h，但最佳 h 难以确认，书中是以 $3n+1$ 来计算的

### 归并排序

保证两个小数组有序，合成大数组后依旧有序

1. 二分法将数组分为最小单元，最小为 1 个元素
2. 逐级合并小数组，保证小数组合并的大数组有序
3. 直到合成为最大的数组

```
// 这种实现很多弊端，例如需要切割、组合数组，可以使用数组索引而非切割数组
// 每次合并需要创建一个新数组，这也是消耗，可以使用局部变量或者传参来避免每次创建新数组
function merge_sort(arr) {
  if (arr.length <= 1) {
    return arr;
  }

  const m = Math.trunc(arr.length / 2);
  return merge(merge_sort(arr.slice(0, m)), merge_sort(arr.slice(m)));
}

function merge(arr1, arr2) {
  let i = 0, j = 0;
  const new_arr = [];
  while (i < arr1.length || j < arr2.length) {
    if (i >= arr1.length) {
      return new_arr.concat(arr2.slice(j));
    } else if (j >= arr2.length) {
      return new_arr.concat(arr1.slice(i));
    } else if (arr1[ i ] < arr2[ j ]) {
      new_arr.push(arr1[ i ]);
      i++;
    } else {
      new_arr.push(arr2[ j ]);
      j++;
    }
  }
  return new_arr;
}
```

**优点**

1. 在大规模数据下或乱序下，表现情况较好，一般时间复杂度为 $O(NlgN)$
2. 他和希尔排序在运行时间差距在常数级别内，所以到底和希尔排序谁快，取决于排序实现

**缺点**

1. 代码较多，理解较为复杂
2. 如果数据较少，每次创建数组的消耗会很大，这点事可以优化的

### 快速排序

1. 将数组按照基准值分为两部分(**二向切分**是两部分，**三向切分**则是三部分)
2. 保证左边的不大于基准值，右边的不小于基准值
3. 递归

```
function fast_sort(arr, lf, rt) {
  if (lf >= rt) return;

  const v = arr[ lf ]; // 基准值
  let i = lf + 1, j = rt;

  while (i < j) {
    // 从左边找到大的值与右边交换
    while (arr[ i ] < v && i < rt) {
      i++;
    }
    // 从右边找到小的值与左边交换
    while (arr[ j ] > v && j > lf) {
      j--;
    }
    if (i >= j) break;

    exchange(arr, i, j);
    i++;
    j--;
  }
  exchange(arr, lf, j);
  // 保证j左边的都比j小，j右边的都比j大

  fast_sort(arr, lf, j - 1);
  fast_sort(arr, j + 1, rt);
  return arr;
}
```

##### 重复元素多时候处理

**三向切分**：当数组中，重复元素较多，如果按照二向切分，重复的元素仍然需要排序，三向切分将数组分为三段 比基准值小的，等于基准值的，大于基准值的

```
...
const v = arr[ lf ]; // 基准值
let i = lf, l = lf, r = rt;
while (i <= r) {
  if (arr[ i ] === v) {
    i++;
  } else if (arr[ i ] < v) {
    exchange(arr, i, l);
    l++;
    i++;
  } else {
    exchange(arr, i, r);
    r--;
  }
}
...
```

**优点**

1. 排序效率较高
2. 比较次数较少
3. 可应对大量重复数据

**缺点**

1. 对于小数组，插入排序效率较高
2. 实现较为复杂

## 深入理解计算机系统

### 冯诺依曼计算机组成

1. 控制器
2. 运算器
3. 存储器
4. 输入
5. 输出

#### CPU

_CPU_ = _控制器_ + _运算器_

CPU 通过 控制器 控制其他部件，通过 运算器 进行算术运算

控制器主要操作有

1. 加载：将*主存*数据加载到*寄存器*
2. 存储：将*寄存器*数据存储到*主存*
3. 运算：读取*寄存器*数据，进行逻辑运算
4. 跳转，根据 _PC_ 存储的指令地址，执行下一个指令

**CPU 指令体系**：  
_指令体系_：指机器码如何在 CPU 中运行  
_微指令体系_：指 CPU 如何工作

#### 存储器

系统将存储器分层处理，速度快但存储量小的处于顶层，速度相对慢但存储量大的位于下层。自顶向下为：_寄存器，缓存（L1,L2,L3 SRAM），主存（DRAM），二级存储，远程二级存储_

#### I/O

输入、输出 通过 *I/O 总线*与 *总线*连接，通过 *I/O 总线*传输数据。例如 鼠标通过 _USB 适配器_ 连接 _I/O 总线_，显示器通过 _图形适配器_ 连接 _I/O 总线_。

### 操作系统及抽象

#### 线程

_线程_：软件运行时的抽象。

**线程切换**

计算机可以同时运行多个软件，看似软件都是同时占有 CPU，但是 单核 CPU 在同一时刻只能运行一个程序，是使用 **线程切换** **上下文切换**实现的

_上下文_：计算机跟踪保存软件的执行状态

例如 Shell 运行 脚本文件

```
Shell 进程 => 执行系统调用指令
操作系统内核 => 切换
脚本文件进程 => 执行系统调用指令
操作系统内核 => 切换
Shell 进程
```

#### 虚拟内存

_虚拟内存_：存储器的抽象

每个程序运行时，都像独自占用整个内存，对于进程而言，内存都是一致的，称为 _虚拟内存空间_。

请注意，当前内存示意图 是**从下往上增大** 的

【 系统内核内存 】**不允许用户访问**  
【 用户栈 】  
【 共享内存 】  
【 用户堆 】  
【 可读写内存 】  
【 只读内存 】  
【 程序开始 】

#### 文件

_文件_：_I/O_ 的抽象（Linux 中一切皆文件的思想）

**利用网络传入文件**

_磁盘 => 磁盘控制器 => I/O 总线 => 总线 => 控制器 => 总线 => 网络适配器 => 远程二级存储_

### 虚拟机

_虚拟机_：对计算机的抽象

### 并发 并行

_并发_：多个资源同时运行，即存在多个活动系统  
_并行_：利用并发来加快执行速度，即同时执行多个任务

**线程并发**

_多核处理器_：CPU 存在多个核心，同一时间处理多个指令，核心之间共享 L3 缓存，但独立 L2，L1;  
_超线程_：CPU 一个核心同时运行超过一个线程，例如 4 核 8 线程，一个核心同时运行两个线程

**指令级并行**

_超标量_：CPU 一个时钟周期运行超过一条指令

**单指令 多数据 SIMD**

一条指令运行多条数据，例如在浮点数运算的优化

## 其他

### vue-cli 生成的项目 webstorm 无法读取 alias

在 Preferences => Language and Framework => Javascript => Webpack 下配置如下路径

<project_path>/node_modules/@vue/cli-service/webpack.config.js

[vue-cli 官方文档](https://cli.vuejs.org/zh/guide/webpack.html#%E4%BB%A5%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%E7%9A%84%E6%96%B9%E5%BC%8F%E4%BD%BF%E7%94%A8%E8%A7%A3%E6%9E%90%E5%A5%BD%E7%9A%84%E9%85%8D%E7%BD%AE)

### mac 修改 app 快捷键

在 System Preferences => Keyboard => Shortcuts => App Shortcuts
点击 "+" 号，选择你想设置的 Application 和 Keyboard Shortcut

例如，我将谷歌的 History back 设置为 "cmd + ["

1.  Application 点击选择 **Google Chrome**
2.  输入 Menu Title **Back**，这里需要注意，<font color="red">Menu Title 必须和应用软件的 Menu Title 保持一致</font>
3.  在 Keyboard Shortcut 同时按下 **cmd + [**

### 编码

[Javascript 常用的编码及乱码问题](../../note/coding/ASCII_Unicode_GBK.md)

### phantomjs

**phantomjs 截图，生成 pdf...**

```

phantom.onError = function (msg) {
  console.log('phantom error: ', msg);
  phantom.exit();
}

const webPage = require('webpage');
const system = require('system');

const page = webPage.create();
const args = system.args; // 参数

page.onError = function (msg) {
  console.log('page error', msg);
  phantom.exit();
}


page.viewportSize = { width: 0, height: 0 }; // 视口设置
page.open(system.args[ 1 ], function (status) {
  console.log(status);
  if (status === 'success') { // enum [success,fail]
    page.render(xx.png); // 截图
    page.content; // 抓取的html，可用于解析
    page.evaluate(function () {
      // 操作界面，如窗口滚动
      window.scrollTo(0, 0);
      console.log('xx'); // 此处打印 在命令行是看不见的，相当于在浏览器界面打印东西,必须通过 onConsoleMessage 事件监听，才能在命令行中看到
    });
  }
});


page.onConsoleMessage = function (msg) {
  console.log(msg);
}
page.onResourceRequested = function () { } // 资源开始请求
page.onResourceReceived = function name() { } // 请求资源有返回
```

1. phantomjs 可以读取本地文件，但严格按照协议，例如 "file:///home/user/....."
2. phantomjs 错误

- 将 phantom.onError 监听放在代码顶部，监听全局错误
- 如果有需要，可以使用 try...catch
- 一些语法错误，phantom.onError 无法捕捉到，例如 **箭头函数**，此时 phantomjs 不运行也不报错
  ，很尴尬

### c++ 数组初始化

1. 静态初始化 `int arr[10] = {0}`
2. 动态初始化 `int *arr = new int[10]{0}`
3. vector 初始化 `vector<int> arr = {}; vector<int> arr1(10,0)`

### createElement createElementNS

1. createElement: 无命名空间创建元素
2. createElementNS：有命名空间的创建元素(没使用过)

[createElementNS MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/createElementNS)

## 算法题

### 三色问题

[leetcode-荷兰三色问题](https://leetcode-cn.com/problems/sort-colors/)
存在三种颜色，0，1，2，现在有乱序数组全部是由三种颜色组成，请排序。1、时间复杂度为 O(N)

- 快速排序 (三向切分，只循环一次)

```
void sortColors(vector<int>& nums) {
  int lf = 0, ri = nums.size() - 1;
  int base_value = 1;
  int i = 0;

  while (i <= ri) {
    if (nums[i] < base_value) {
      int s = nums[i];
      nums[i] = nums[lf];
      nums[lf] = s;

      i++;
      lf++;
    } else if (nums[i] == base_value) {
      i++;
    } else {
      int s = nums[i];
      nums[i] = nums[ri];
      nums[ri] = s;

      ri--;
    }
  }
}
```
