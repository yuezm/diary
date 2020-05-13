## Node

### Error

#### 错误类型

Error 是 node 中基本错误对象，其他错误继承于 Error

1. SyntaxError：语法错误
2. TypeError：类型错误
3. ReferenceError：引用错误
4. SystemError
5. RangeError：数值不在合法范围内，例如端口为-1
6. URIError：URI 方法转换错误时错误
7. AssertionError：assert 时未通过时错误
8. OpenSSL：openSSL 的错误

#### 错误追踪

##### Stack Tree

错误追踪其实就是寻找调用栈，根据错误调用栈来确定错误的位置。常用的错误信息一般为 error.message、error.stack 这两个属性展示信息不一致

##### Error.captureStackTrace

`Error.captureStackTrace(targetObject[, constructorOpt])`，用于隐藏错误细节

```javascript
const demo = {};
Error.captureStackTrace(demo); // 向demo对象添加 stack 属性，值为栈调用信息
console.log(demo.stack); // demo的调用栈

// 自定义错误对象
class MyError {
  constructor() {
    Error.captureStackTrace(this, MyError);

    this.message = 'I am a Error';
  }
}

new MyError(); // MyError { message: 'I am a Error' }
```

##### Error.prepareStackTrace

对错误进行处理，**且注意，处理后对所有错误生效**

```javascript
Error.prepareStackTrace: (err: Error, stackTraces: NodeJS.CallSite[]) => any;

// err：错误本身
// stackTraces 错误调用栈

Error.prepareStackTrace = function(err, stackTraces) {
  // return err;
  // return stackTraces
  // return err + stackTraces
  // return 'Hello word!!!'
};
```

##### Error.stackTraceLimit

限制调用栈展示行数，默认 10

## 安全

### 点击劫持

点击劫持：**点击劫持本质上是一种视觉欺骗**，攻击者利用 iframe 嵌套目标网页，并将目标网页设置为透明，通过调整位置，使用户在操作时对 iframe 进行操作

#### 点击劫持分类

1. iframe 劫持
2. 图片劫持
3. 拖拽劫持：利用 Drag API，且**Drag API 无同源限制**，可以诱导用户将想要的数据放入**另一个页面**，攻击者从而获取数据
4. 触屏劫持：由于手机端屏幕受限，网站 URL 无法显示，更容易钓鱼和劫持。_内容保持一致，就可以让用户认为是一个网站_：

#### 点击劫持防御

1. iframe busting：利用 Javascript 代码判断，当前 window 是否处于顶层 `window.top`。但是 iframe busting 也存在缺点，由于是 Javascript 进行判断，如果阻止 Javascript 执行即可绕过 iframe busting，例如 _IE 中 iframe security 等特性，会导致 iframe 内代码不执行_
2. X-Frame-Options：使用 HTTP 头部来控制 iframe 策略：
   - DENY：拒绝加载为 iframe
   - SAMEORIGIN：同域加载
   - ALLOW-FROM：origin：定义可加载的域名
3. CSP

## Vue

### Composition API

vue 3.0 新的 API

#### 目的

1. 更好的代码重用：随着功能模块增多，vue 的重用会变得很复杂（vue2.0 主要使用的 mixin）
2. 更好的类型推断：vue2.0 时，未考虑类型推断，大部分都使用的是 this，而在 composition API 使用的都是函数

#### HOC，Mixin，Hooks 优劣势

##### HOC（高阶组件）

1. 高阶组件即是高阶函数，传参为组件（函数）或返回值为组件（函数）。高级组件就是高阶函数是没错的，但是在 vue 中，_经过编译后组件才是函数，在书写时，还是一个 JSON 形式的对象_
2. 高阶组件透传 props，事件，slots
3. 高阶组件**不修改原组件**，是无侵入式的

**典型应用**：日志打点，权限控制

```typescript
import { Component, ComponentOptions, CreateElement, VNode } from 'vue';
import Vue from 'vue/types/vue';

// HOC
export default function ConsoleWrapper(com: Component): ComponentOptions<Vue> {
  return {
    components: {
      com,
    },
    methods: {},
    created() {
      console.log('I am in consoleWrapper'); // 日志打点
    },
    render(h: CreateElement): VNode {
      return h('com', {
        on: (this as Vue).$listeners,
        attrs: (this as Vue).$attrs, // 对于props未接收的属性（出id和class）外进行接收
        scopedSlots: (this as Vue).$scopedSlots, // slots现在在scopeSlots也存在，命名为default，所以传scopeSlots就够了
      });
    },
    // template: '<com v-bind="$attrs" v-on="$listeners"><slot/></com>' // template也可以除了 scopedSlots，其他都可以实现
  };
}
```

**劣势**

1. 需要额外的 vue 实例
2. 劫持$attrs,$listeners,\$scopedSlots，不注意约定的情况下，容易造成命名冲突
3. 由于 vue 的特点，在 vue 中书写高阶组件比较麻烦

##### Mixin

Mixin：原理是对象属性的复制，可以将多个对象的属性进行赋值（_这是继承无法做到的_）

1. 以对象形式混入，可以提高复用
2. 选项合并，且生命周期钩子合并时，是全部合并，而非替换，且在组件钩子之前调用

**劣势**

1. 命名冲突
2. 当项目越来越大时，mixin 导致无法确定是哪个混入文件生效，不利于维护
3. _对组件侵入式修改_

##### Hooks

1. 暴露出的属性都具有明确来源
2. 合成函数任意命名，不存在命名冲突
3. 更好的类型推断
4. 无新的 Vue 实例，提高性能

**缺点**

1. 需要区分 ref 值和纯值
2. 属性在 ref 后，会变得冗长

**Hooks 中 setup 无法获取 this**

#### API

```typescript
ref()
toRefs()
isRef()

reactive()
watch()
computed()
provide()
inject()

watchEffect()


// 生命周期

beforeCreate -> setup()
created -> setup()

beforeMount -> onBeforeMount()
mounted -> onMounted()

beforeUpdate -> onBeforeUpdate()
updated -> onUpdated()

beforeDestroy -> onBeforeUnmount()
destroyed -> onUnmounted()

errorCaptured -> onErrorCaptured()
```

_当生命周期 vue2.0 和 vue3.0 同时运行时，在 **vue2 中优先运行 vue2 的生命周期钩子**，**vue3 中优先运行 vue3 的生命周期钩子**，所以不建议混用，免得版本升级搞混_

## dart

### dart 特性

1. 在 dart 中，**一切皆对象**，任何数据类型都是对象，继承于*Object*，包括 number，String，null 等
2. dart 为强类型语言，但类型是可以推断的，例如`var a = 1;` 推断为 int
3. dart 支持泛型
4. dart 支持函数，且必须包含入口函数 `int main(){}`
5. dart 没有类的访问限制符（public，private，protected），在 dart 中，私有变量以"下划线\_"表示
6. dart 攻击提示两种错误类型，警告、错误。警告只是程序可能错误，但不会阻止程序执行；而错误会阻止程序执行非，分为*编译错误*和*运行错误*

### 变量声明

```
int n = 1;
var n1 = 2;
dynamic n2 = 2;

final n3 = 4; // final值可被修改一次
const n4 = 5; // const值声明后固定不变
```

### 变量类型

1. number
   - double：`double d = 1.0;`
   - int：`int i = 1;`
2. string

   - String：`String str = '';`
   - r'...'：原始字符串 `String str = r'';`
   - '${exp}'：如果exp是变量，则可以简化为'$exp'

   ```
   String exp = 'exp';
   String str = '$exp';
   String str1 = '${exp.toLowerCase()}';
   ```

   - ... xxx ...：多行字符串：`String l = '''long string''';`

3. boolean：bool：`bool b = false;`
4. list/array：`List<int> l = [];`
5. set：`Set<int> s = {1, 2, 3, 4};`
6. map：`Map<int, int> m = {1: 2};`
7. Rune：用来表示字符串中的 UTF-32 编码字符
8. Symbol：程序中声明的运算符或者标识符 `Symbol s = #symbol;`

### 运算符

```dart
+，-，*，/，%
&&，||，!
&，|，^，~，<<，>>
=
==，!=，<= ...

// 级联运算符(..)
.. 可以对同一个对象连续调用，节省临时创建变量的步骤
```

### 函数

```dart
// 位置参数，可选参数
int fn(int x, int y, [int z = 0]) {
  return x + y + z == null ? 0 : z;
}
fn(1, 2);

// 命名参数，可选参数
import 'package:meta/meta.dart';

int fn({@required int x, @required int y, int z = 0,}) {
  return x + y + z;
}
fn(x: 1, y: 2);
```

#### 箭头函数

```dart
int fn(int x, int y) => x + y;
```

#### 匿名函数

```dart
([[Type] param1[, …]]) {
  codeBlock;
};

// 例如
list.forEach((item){
  ...
})
```

### class

```dart
class Test{
  // 构造函数，构造函数的名字可以是 ClassName 或者 ClassName.identifier
  Test(int x, int _y) {
    this.x = x;
    // 或者省略this
    y = _y;
  }
}
```

#### 构造函数

1. 类如果未声明构造函数，则提供默认的构造函数
2. 构造函数不被继承

```dart
class Test {
  int x;

  // 命名构造函数
  Test.X(int _x) {
    x = _x;
  }
}

var t1 = new Test(1, 2):
var t2 = new Test.X(1);
```

#### 继承

```dart
class P {
  int x;
  P(int _x) {
    x = _x;
  }
}

class C extends P {
  C(int _y) : super(_y);
}
```

#### Getter Setter

```dart
class Test {
  String _name = 'Test';

  String get name => _name;
  set name(String name) => _name = name;
}
```

##### 重写

**可以重写父类方法，也可重写运算符**

```dart
class Vector {
  final int x, y;

  Vector(this.x, this.y);

  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);

  // 运算符 == 和 hashCode 部分没有列出。 有关详情，请参考下面的注释。
  // ···
}
```

#### 抽象类型

```
abstract class Test {

}
```

#### 静态属性、方法

```dart
class Test {
  static int i;
}
```

#### Mixin

```dart
// Mixin 是复用类代码的一种途径， 复用的类可以在不同层级，之间可以不存在继承关系
class Maestro extends Person
    with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}
```

#### 对象类型

`test.runtimeType`：返回 Type 对象

### 枚举

```dart
enum Color { RED, BLUE };
```

### 异步支持

Future 是 JS 中的 Promise

```dart
Future checkVersion() async {
  var version = await lookUpVersion();
}
```

### 生成器

```dart
// 同步生成器
Iterable<int> naturalsTo(int n) sync* {
  int k = 0;
  while (k < n) yield k++;
}

// 异步生成器
Stream<int> asynchronousNaturalsTo(int n) async* {
  int k = 0;
  while (k < n) yield k++;
}
```

**对于所有 Dart 代码有两种可用注解：@deprecated 和 @override。 关于 @override 的使用， 参考 扩展类（继承）**

## leetCode

### 搜索旋转非排序数组

思路：二分法 + 确定数组有序

设 起始位 index 为 s, 结束 index 为 e，第 0 位的值为 START，最后一位的值为 END

二分法：使用二分法确定 m = (s + e) >> 1；

确定数组是否有序：

1. 如果 arr[m] >= START，则表示 m 的左边是有序的，如果 target < arr[m] && target >= START，则应该向做移动，否则向右
2. 如果 arr[m] <= END，则表示 m 的右边是有序的，如果 target < arr[m] && target <= END，则应该向右移动，否则向左

```

```

**注意点**

1. 即使判断有序时，也需要与两端的极值做比较，不然会出现如下情况 arr = [4,5,6,1,2,3]，target = 2：_2 < 6，但此时也应该向右移动_

[搜索旋转非排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

### 数组中数字出现的次数

### 山脉数组找出目标值

### 快乐数

```

```
