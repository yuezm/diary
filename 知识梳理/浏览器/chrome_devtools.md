# Command

## 使用命令对某个节点截图

1. Elements 选中节点
2. 右击选中节点截图（Capture node screeenshot）或者 Command 内选择节点截图（Capture node screeenshot）

## show timestamps

可以展示 console 的时间戳

# Console

## 自定义 Console

1. F1 打开设置，设置 Enable custom formatter
2. 如下代码实现 ` ``typescript window.devtoolsFormatters = [ { header: function (obj) { const style =`

  ```
  color: red;
   border: dotted 2px gray;
   border-radius: 4px;
   padding: 5px;
  ```

  `const content =`${ JSON.stringify(obj, null, 2) }`;

  try {

  ```
  return [ 'div', { style }, content ]
  ```

  } catch (err) { // for circular structures

  ```
  return null;  // use the default formatter
  ```

  } }, hasBody: function () { return false; } } ]

// 当输入可 JSON.stringify 的对象时，不会收起来，而是直接展示

````
### copy


```shell
copy(xx) # 可以在 console 中，将值赋值到粘贴面板，比如 copy(localStorage)
````

## Store object as global variable

在你输出变量的，例如

```shell
>[1,2,3,4,5]; # 输入该值
>[1,2,3,4,5]; # 此时右击该变量，则会出现该操作，可以将此处变量存储为全局变量
```

## 

## Save as

对报错信息 "右击"，此时会出现该操作，可以保存该堆栈信息

## $

```typescript
$0  第一个选中节点
$1  第二个选中节点
    ...
$n

$$  quseySelectAll
$   quseySelect

$_  上一个执行的结果
```

## await

Console 内可以直接执行 await，如下所示

```typescript
await navigator.storage.estimate()
```

## monitor、monitorEvents

monitor 类似于 mock 函数，可以对函数进行追踪，当函数运行时，会打印出函数名和调用参数

monitorEvents 是对事件的mock

## 查找

在 Element、Console、Source、Network 都支持查找 【cmd + F】

在 Console、Source、Network，查找可以使用正则

在 Source中，可以使用 【cmd + P】 来查询文件

## log

### log结合css

```typescript
console.log("%c Hello word", "font-size: 100px");
```



## Live expression

可以编辑Javascript表达式

## 

# Source

## Snippets

可以保存可执行的代码，避免重复输入，右击 run 或者 cmd + enter 运行

## Breakpoint

```
Add breakpoint：普通断点
Add conditional breakpoint：条件断点，输入表达式，返回值为 true/false
Add logpoint：日志打印，可以打印某些元素
```



### XHR断点

XHR/fetch BreakPoints

# Network

## filter

过滤：method:POST 反过滤 : -method:POST

# Element

1. h 可以显示。隐藏元素
2. 使用鼠标拖动元素
3. cmd或control + 方向箭头，来移动元素
4. Shadow editor：编辑阴影
5. Timing function editor：编辑动画
6. 颜色选择器

## 监听元素状态

当元素状态发生改变时，进入代码断点

```
元素 --> 右击 --> Break on

    subtree modifications   子元素改变（文本节点）
    attribute modifacations 元素属性改变
    node removal 元素删除
```



# 隐藏功能

ESC 打开、关闭

## Coverage

可以查看覆盖率，比如可以查看脚本哪些是否被执行
