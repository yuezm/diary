# 10 月第二周

## 计划完成情况

1. CSAPP 结束（几乎就是过了一遍，后续需要再看一遍）
2. 深入 TS 结束（收货挺大，只是为看编译那一块）
3. <font color="red">计算机网络 第一章未完成</font>
4. <font color="red">编译器设计 第一章未完成</font>
5. <font color="red">算法 第 3 章未完成</font>

## 下周计划

1. 推进进入 计算机网络完成到第二章，编译器完成到第二章，算法待定
2. 下周需要想想文章题目，10 月份两篇文章还未完成

## Javascript

### import() 加载数据流

1. import 可以动态加载模块

```
import("./index.js"); // 返回的Promise
```

2. import 加载数据流

url 数据流: "data:[<mediaType>][;base64],<data>"；例如 "data:text/javascript;chartset-utf-8,console.log('HELLO WORd')"

import("data:text/javascript;chartset-utf-8,console.log('HELLO WORd')"); // 打印 HELLO WORD，类似 eval 功能

### encodeURI encodeURIComponent

共同点：都是将字符串转换为统一资源定位符（URL）

不同点：

1. encodeURI：转换为 url 编码

   无法转义的字符：

- 保留字符：; , / ? : @ & = + \$
- 非转义字符：字母 数字 - \_ . ! ~ \* ' ( )
- 数字符号：#

2. encodeURIComponent：转换 url 编码

   无法转义的字符：

- 字母、数字、(、)、.、!、~、\*、'、-和\_

### TS

1. TS 结构化类型特点：结构类型和其他强类型语言不一致，`如果类中成员一致，也认为两个类型相同`，但是如果类含有`private protected`，则认为不一致，即使两个类成员完全相同
2. TS 函数参数特点 不赋值，void 返回值问题：声明返回值的函数可以赋值给 void 类型函数，理解为`当函数声明为 void 时，赋值时不会对返回值进行校验`
3. 空接口特点：空接口匹配一切类型（never 除外）

## 网络

## Linux

## 其他

计算的本质（王垠）
