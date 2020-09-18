# Aug-1

## CS

### 编程发展

#### 早期编程

1. 打孔纸卡、纸带
2. 插线板 -> 可拔拆线板
3. 冯诺依曼结构（冯诺依曼结构当时也是使用的是打孔纸卡、纸带）
4. 面板编程
5. 编程语言

#### 编程语言发展

1. 机器语言
2. 汇编语言（助记符）
3. 高级语言

#### 软件工程

1. 编辑器
2. 代码
3. 测试
4. DEBUG
5. 文档+注释
6. 版本控制

### 集成电路

1. 分离元件：包含真空管、电子管时期，真空管/电子管 之间使用其他电路元件连接
2. 集成电路：
   - 印刷电路板：通过腐蚀金属板的方式，将电子元件连接在一起，大幅度减少独立的元件和连线。但是印刷电路板也无法塞进较多的电子元件
3. 大规模集成电路

   - 光刻：使用光将图案复制到材料上，例如半导体（硅），光刻使得晶体管大幅度增加
     **光刻优势**：

     1. 元件路基更近，传输速度更快，发热量更小
     2. 信号延迟更低，时钟周期更短
     3. 元件更小
     4. 价格更低

     **光刻劣势**：

     1. 光的波长是局限性：光的波长，精度已经到达上限，科学家正在研究更短的波长
     2. 量子隧道贯穿：当晶体管的距离过近，例如只有几个原子的距离，电子会跳过间隙

## Go

### 语言结构

#### 包

```go
package main // 主模块，打包后为可执行文件，每个项目只有一个主包

package Test // 其他模块，打包后为 p.a，可以存在多个其他包
```

#### 加载

```go
import fmt // 加载标准库，当然也可以加载其他的库、模块
```

模块内部**大写变量**才能被其他模块加载，否则为私有变量

#### main，init

```go
// 主方法
main(){

}

// 变量除了可以在全局声明中初始化，也可以在 init 函数中初始化
// 这是一类非常特殊的函数，它不能够被人为调用，而是在每个包完成初始化后自动执行，并且执行优先级比 main 函数高
init(){

}
```

#### 关键字

```go
_：空白标识符
var：
const：
func：
```

### 数据结构

#### 数字类型

1. int、uint：int 和 uint 的长度时根据系统决定的，但是即使是 int 长度与 int32 一样，但两个也不是一个类型
2. int8，int16，int32，int64
3. uint8，uint16，uint32，uint64
4. float32，float64
5. uintptr：指针，指向一段内存地址

   ```go
   & 取地址
   * 根据地址，取址

   var c = 2
   var p *int = &c
   *p = 100 // c也为100
   ```

   go 指针不能和 c++一样进行运算，直接运算容易造成内存泄漏

   ```go
   *p++ // 报错，属于不合法操作
   ```

   对一个空指针的反向引用是不合法的，并且会使程序崩溃

   ```go
   var p *int = nil
   *p = 2 // 报错
   ```

6. complext： 复数

#### 字符串

1. byte：UTF-8 的别名，字符串就是 byte 数组
2. string： 字符串，string 是 byte 的数组，但是**无法获取每个字节的地址，这是非法的**

```go

str := "Hello word"
s := str[0]
```

#### 布尔

```go
var t true = true
var t false = false
```

#### 数组

```go
arr := [5]int{} // 长度声明
arr :=[5][5]int{}
arr := [...]int{1,2,} // 根据元素推测长度

arrPtr := &arr // 数组指针
```

数组声明时，长度即确定了，且无法使用变量来规定长度(const 常量可以)

如果想使用变量，则

```go
arr := make([]int, 20);
arr2 := make([][]int, 20); // 二维数组
```

#### struct

```go
type Test struct {
 name string
}
```

#### map

```go
var map[string]string Test
```

#### interface

```go
interface Test{

}
```

#### 函数

```go
func Test(){

}
```

#### 类型别名

```go
type Color int
```

### 声明

#### 变量

```go
var identifier type
var identifier type, identifier2 type2
var a,b,c = 1,2,3 // 多变量

var b = 2 // 类型推断

c := 2 // 省略var，如果:=没有值，则编译错误
```

1. 一个变量（常量、类型或函数）在程序中都有一定的作用范围，称之为作用域。如果一个变量在函数体外声明，则被认为是全局变量，可以在整个包甚至外部包（被导出后）使用，不管你声明在哪个源文件里或在哪个源文件里调用该变量。
2. 在函数体内声明的变量称之为局部变量，它们的作用域只在函数体内，参数和返回值变量也是局部变量

函数内，尽量使用 `:=` 写法

#### 常量

```go
const BLUE = 3
const (RED = 1,BLUE = 2) // 也可以当成枚举使用
```

### 语句

#### 循环语句

```go
for init; condition; post { }

# for range 结构
for index,value := range arr {}

for {
  // 无限循环，由内部判断什么时候结束
}
```

记住，go 无 while 循环，可以使用 for 循环拉模拟 while 循环

```go
break;
continue; continue LABEL;

goto LABEL


LABEL1:
 for {
  if 1 == 1 {
  break LABEL1
  continue LABEL1
  goto LABEL1
 }
}
```

#### 判断

```go
if condition {
  ...
} else if condition {
  ...
}else{

}


if initialization; condition {
  ...
}

// 例如

if i :=10; i < 20{

}
```

请注意 go 没得三目运算

```go
switch value {
  case v1: # v1 可以为值，也可以为表达式，例如 case value < 2:
    ...    # 不需要break语句
   case v2:
   ...
   default:
    ...
}
```

## 设计模式

### 设计原则

1. 单一职责原则：描述功能模块关系，表示功能模块的工功能应该单一
2. 开闭原则：*类、函数、模块*应该是可以被扩展的；但不应该被修改。对于以前的抽象模块，可以做到不修改，不改变现有功能的情况下，进行扩展
3. 里氏替换原则：子类可以扩展父类的方法，但不应该修改父类的方法
4. 依赖倒置原则：面向接口编程，我定义好了接口，凡是实现了该接口的类，我都可以使用，而无需关心类的本身
5. 接口隔离原则：不应该依赖他不需要的接口，可以将接口更小的粒度化，但同时更小粒度会增加复杂度
6. 迪利特法则（最少知道原则）
7. 组合复用原则：使用已存在的对象，使之成为新对象的一部分；而不是继承

// 单一职责，开闭原则，里式替换原则，（依赖注入，面向接口编程），最少知道原则，组合

## LeetCode

### 颠倒二进制

思路 1. 数字 => 二进制 => 反转 => 数字

```typescript
function reverseBits(n: number): number {
  return Number('0b' + n.toString(2).split('').reverse().join('').padEnd(32, '0'));
```

思路 2. 每次取 n 末尾的数字，然后将 result 左移一位，再加上该末尾数，执行 32 次

```typescript
/**
 * @param {number} n - a positive integer
 * @return {number} - a positive integer
 */
var reverseBits = function (n) {
  let result = 0;

  for (let i = 0; i < 32; i++) {
    result *= 2; // 因为js为32位有符号整型，所以不能直接使用位运算符
    result += n & 1;
    n >>= 1;
  }

  return result;
};
```

[颠倒二进制](https://leetcode-cn.com/problems/reverse-bits/)

### 缺失的数字

思路 1.

```typescript
function missingNumber(nums: number[]): number {
  const n = nums.length;
  let sum = (1 + n) * (n / 2);

  for (const i of nums) {
    sum -= i;
  }

  return sum;
}
```

思路 2. 位运算，nums XOR，再 [1,n] XOR，由于缺失的数字只会 XOR 一次，则剩余的值为目标值

```typescript
function missingNumber(nums: number[]): number {
  let result = 0;

  for (const i of nums) {
    result ^= i;
  }

  for (let i = 0; i <= nums.length; i++) {
    result ^= i;
  }

  return result;
}
```

思路 3. 排序

```typescript
function missingNumber(nums: number[]): number {
  nums.sort((a, b) => a - b);

  for (let i = 0; i < nums.length; i++) {
    if (i !== nums[i]) {
      return i;
    }
  }

  return nums.length;
}
```

[缺失的数字](https://leetcode-cn.com/problems/missing-number/)
