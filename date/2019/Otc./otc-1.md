# 10 月第一周

本月主要目标：

1. 计算网络完成一半，预计 300 页码
2. 算法 3 查找，4 图
3. 编译器设计预计 1 概观，2 词法分析，3 语法分析
4. 深入 TS 完成
5. CSAPP 第一遍完成（非常初略完成），后续需要过第二遍
6. 待定事项

## Javascript

### TS

#### 高级类型

- 联合类型

```
type Union = A | B; // 表示要么是A类型，要么是B类型
```

- 交叉类型

```
type Cross = A & B; // 表示即是A类型，又是B类型
```

- infer：在 类型 extends 语句时表示类型推断

```
type Test<T> = T extends Array<infer F> ? F : never; // 对F类型进行推断
```

**如果 T 为联合类型**

在条件类型 T extends U ? X : Y 中，当 T 是 A | B 时，会拆分成 A extends U ? X : Y | B extends U ? X : Y

**将元祖转换为联合类型**

type Trans<T> = T extends Array<infer E> ? E : never;

**将接口、内联类型转换为联合类型**

type Tran<T> = T extends { A: infer U, B: infer U } ? U : never;
type X1 = Tran<{ A: string, B: number }>

**将接口、内联类型转换为交叉类型**

type Tran2<T> = T extends { A: (a: infer U) => void, B: (a: infer U) => void } ? U : never;
type X2 = Tran2<{ A: (a: string) => void, B: (a: number) => void }> // string & number

**将元祖转换为交叉类型**
type Tuple = [ string, number ];
type Tran1<T> = (T extends any ? (k: T) => void : never) extends (k: infer I) => void ? I : never; // 将 string|number 转为 string&number
type Tran2<T> = T extends Array<infer I> ? I : never; // 将 tuple 转为 string | number
type T = Tran1<Tran2<Tuple>> // string&number

- keyof：获取类型的 keys 值

```
interface I {
  name: string;
  age: number;
  sex: never;
}

type J = I[ keyof I ]; // number | string，去除掉了never
```

- in：对类型进行循环

```
interface I {
  name: string;
  age: number;
  sex: never;
}

type J = {
  readonly [ F in keyof I ]: I[ F ]; // 将interface 转为readonly
}
```

- []：对类型取值

但无法直接取值，如下所示

```
interface A {
  name: string;
}

// type D = A[ string ]; 这样是对值的操作
```

类型操作方法，例如 Exclude，Extract...

```
interface Test {
  name:string;
  age:number;
}
type Test1 = "TOM" | "JACK";

Partial<T>：将T类型设为可选，Partial<Test>;
Readonly<T>：将T类型设置readonly，Readonly<Test>;
Record<K,T>：将类型映射 Record<Test,Test1> => { TOM:{ name:string,age:number, },JACK:{name:string,age:number} }

// Pick Omit 主要是对构造进行操作，例如interface,class等
Pick<T,K>：从T类型中取出 key为 K 类型的 Pick<Test,"name">
Omit<T,K>：从T中去除掉K类型  Omit<Test, "age">

// 主要是对联合类型，交叉类型操作
Exclude<T,U>：只在T中取出U的类型 Exclude<string | number | boolean, string>; // number | boolean
Extract<T,U>：只在T中保留U的类型 Extract<string | number | boolean, string>; // string

NonNullable<T>：去掉类型中的 null，undefined
ReturnType<T>:  构造函数返回值类型
InstanceType<T>：构造函数类型
Required<T>：将类型改为 required
ThisType<T>: 不转换和返回类型，而是标记上下文
```

## CSAPP

### 异常

只列出异常处理的流程和分类

处理机检测到异常 => 异常跳转表 => 操作系统子程序（异常处理系统）

1. 中断：异步，一般为 I/O 设备除法
2. 陷阱：同步，主要为为操作系统提供调用（系统调用）
3. 故障：同步，一般为执行程序出现错误，但是可以恢复执行
4. 终止：同步，不返回，执行程序出现致命错误，且不会恢复

### 进程

进程是对应用程序的抽象

1. 以逻辑控制流来模拟独占 CPU：并发流、并行流
2. 以虚拟内存来模拟独占 内存

### 文件 IO

1. 普通文件
2. 二进制文件
3. 套接字等

### 网络

网络：由节点（包括端系统，分组交换机等）和链路组成的图

IP 协议：主要定义了网络的`命名系统`和简单的定义了`传输方式`
TCP：定义了网络可靠的传输方式

## Leetcode

### 只出现一次数字

思路 1：hash 表

```
int singleNumber(vector<int>& nums) {
      map<int, int> m;

      for (int i : nums) {
        if (m.find(i) != m.end()) {
          m.find(i)->second++;
        } else {
          m[i] = 1;
        }
      }

      auto s = m.begin();

      while (s != m.end()) {
        if (s->second == 1) return s->first;
        s++;
      }
      return 0;
    };
```

思路 2：异或

```
int singleNumber(vector<int>& nums) {
    int n = 0;

    for (int i : nums) {
      n ^= i;
    }
    return n;
  };
```

[只出现一次数字](https://leetcode-cn.com/problems/single-number/solution/zhi-chu-xian-yi-ci-de-shu-zi-by-leetcode/)

### 加 1

思路：从后往前加，主要是考虑`进位`的问题

```
vector<int> plusOne(vector<int>& digits) {
  int n = 1;
  int len = digits.size();

  for (int i = len - 1; i >= 0; i--) {
    int v = digits[i] + n;
    digits[i] = v % 10;
    n = v / 10;
    if (n < 1) break;
  }

  if (n > 0) {
    digits.insert(digits.begin(), 1, 1);
  }
  return digits;
}
```

### 有效数独

### 移动 0

思路：快慢指针，快指针指向需要移动的数字（如果数字不为 0 才移动），慢指针指向移动后的位置

```
void moveZeroes(vector<int>& nums) {
  int i = 0;
  int j = 0;

  int len = nums.size();
  while (j < len) {
    if (nums[j] != 0) {
      if (i != j) {
        int s = nums[i];
        nums[i] = nums[j];
        nums[j] = s;
      }
      i++;
    }
    j++;
  }
}
```

### 旋转图像

思路 1：暴力反转
思路 2：上下反转 + 对角线反转

```cpp
void rotate(vector<vector<int>>& matrix) {
  int size = matrix.size();
  if (size <= 1) return;

  // 首先是上下交换
  for (int i = 0; i < size / 2; i++) {
    auto s = matrix[i];
    matrix[i] = matrix[size - 1 - i];
    matrix[size - 1 - i] = s;
  }

  // 再是按照对角线交换
  for (int i = 0; i < size; i++) {
    for (int j = 0; j < i; j++) {
      int s = matrix[i][j];
      matrix[i][j] = matrix[j][i];
      matrix[j][i] = s;
    }
  }
}
```

### Typescript 类型转换

```typescript
// 将;
interface Test {
  name: string;
  age: number;
  show<T, U>(args: Promise<T>): Promise<U>;
}

// 转换为;

type Result = {
  show<T, U>(args: T): U;
};
```
