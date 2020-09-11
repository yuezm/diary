## Javascript

### Number(n: any)

Number 构造函数转换

1. 对于字符串，转换为数字
2. 对于数字，转换为数字
3. 对于 bool 值，转换为 0、1
4. 对于对象，调动 valueOf 或 toString 方法转换基本类型，在按照 1、2、3 规则转换
5. null 转为 0
6. Symbol 转换报错
7. 其他都为 NaN

#### parseInt(s: string, radix: number)

parseInt 转换的时字符串，如果传入非字符串，则**先转换为字符串**，例如对象调动 toString()；radix 表明进制数

parseInt 从第一位开始转换:

- 如果第一位就不能转换，或者 s 为空: NaN
- 如果后续位无法转换: 截断

1. radix===0 或 radix===undefined: 以 s 的前缀判断，如果前缀为*0X*，则为 16 进制，否则为 10 进制
2. radix===1 或 radix >=37: NaN
3. 2<= radix <=36: 按照进制数转换，且按照普通规则转换

#### parseFloat(s: string)

parseFloat 转换的时字符串，如果传入非字符串，则**先转换为字符串**，例如对象调动 toString()；radix 表明进制数

parseFloat 从第一位开始转换:

- 如果第一位就不能转换，或者 s 为空: NaN
- 如果后续位无法转换: 截断

#### toString(radix: number)

将数字转换为其他进制数

### Array

**Javascript 数组特点**

1. Javascript 数组并非传统意义上的数组，内部实现是基于 Object 的，所以数组的下标也不是传统数组的索引，虽然看上去是数字，但可以看成对象的 key（或者 HashMap 的 key）
2. 但在新的引擎中，对数组进行了优化，会开辟连续内存存储数组。如果数组内存储数据相同，则引擎会进行优化，否则还是按照老数组存储
3. 新的数组结构 TypedArray 是类似于传统的数组结构，且可以直接操作内存

#### push

push 方法完全是基于数组对象的 length 属性的

1. length 属性先转换为数字（其他类型也可以，只要能转换为数字的类型即可）；如果无法转为数字、或 length 属性不存在，则 length 为 0
2. 基于 length 增加元素 `array[length] = element; length++;`
3. 返回数组新的 length

#### Set、WeakSet

`Set<any>、WeakSet<object>` 都是类数组存储结构:

1. 内部元素无法重复，比较方式**除去 NaN 相等外，类似于"==="比较**
2. 内部元素无序
3. 内部 v8 存储结构为 JSCollection

WeakSet:

1. 内部只能存储对象
2. WeakSet 内部存储的对象不计算引用，所以可能被随时回收
3. WeakSet 由于弱引用的原因，无法遍历，无法计算长度

#### Map、WeakMap

`Map<any, any>、WeakMap<object, any>`都是类 HashMap 存储结构，也是基于 JSCollection 实现

1. 原 Object key 值只能为字符串，而 Map 可以为任意类型，内部键比较视**除 NaN 视为一个键以外，以"==="比较**

WeakMap:

1. key 只能为对象
2. WeakMap key 值对象不计算引用，所以可能被随时回收
3. WeakMap 由于弱引用，无法遍历，无法计算元素个数

### 节流防抖

#### 节流

使事件在一段时间内只触发一次

#### 防抖

使事件在一段时间内只触发一次，再次触发事件会清除上一个事件。_理想情况下，防抖可以永远不触发，而节流会按频率触发_

```
function debounce(handler, ms = 500) {
  let timer = null;
  return function handle() {
    clearTimeout(timer);
    timer = setTimeout(() => {
      handler(); // 如果需要处理this，则使用call等处理
    }, ms);
  };
}

function throttle(handler, ms = 500) {
  let onOff = true;
  return function() {
    if (!onOff) return;
    onOff = false;
    handler();

    setTimeout(() => {
      onOff = true;
    }, ms);
  };
}
```

### class

#### new.target: ConstructorFunction | undefined

判断当前构造函数是否为 new 调用，如果为`new Test() => new.target === Test`，而非 new 调用则`new.target === undefined`

**ES6 Class 和 ES5 "Class"区别**

1. 是否必须 new 调用: ES6 必须 new 调用；而 ES5 不必须
2. 是否变量提升: ES6 不会变量提升（TDZ）；ES5 会变量提升
3. 是否自动严格模式: ES6 自动严格模式；而 ES5 不是
4. 内部函数特点: ES6 内部所有函数无 prototype、constructor 且**不可遍历**；而 ES5 不是
5. 内部是否能操作 Class 名:ES6 内部无法改变 Class 名；ES5 无限制

**ES6 Class 继承 和 ES5 "Class" 继承区别**

1. 继承原理: ES6 继承方式，将 `C.prototype.__proto__ = P.prototype`；ES5 继承（构造函数+原型继承）`C.prototype = new P()`
2. 静态属性、方法继承: ES6 会继承静态方法、属性；ES5 在一般写法是没有继承
3. 调用顺序: ES6 先实例化父级`super()`，再实例化子集修饰；ES5 先实例化子集

### let、const、var

1. 变量提升: let、const 不会变量提升(TDZ)；var 会
2. 全局变量绑定 window: let、const 不会绑定；var 会
3. 变量再次声明: let、const 不允许再次声明；var 可以
4. 块作用域: let、const 存在块作用域；var 没有
5. 未声明使用: let、const 必须先声明再使用；var 不用，如果未声明则自动声明为全局变量

TDZ（Time Dead Zone）: Javascript 在词法解析时，如果遇到变量声明或函数声明，则会在作用域内创建变量，**函数声明直接进行赋值运算；var 声明进行初始化，但不进行赋值运算；let、const 只创建变量，不进行初始化**。let、const 在未进行赋值运算时，访问该变量会抛出`ReferenceError`。**在变量声明和变量正真能访问的这一段时间，称为 TDZ**

### Javascript 类型判断 typeof、instanceof、Object.prototype.toString.call()、Array.isArray

1. typeof: 根据存储数据的标识判断；缺点:
   - 只能判断 Javascript 类型
   - 无法对 Object 准确判断
2. instanceof: 根据原型链判断；缺点
   - 只能判断对象，无法对基本类型判断
   - `所有对象 instanceof Object === true`
   - `Symbol.hasInstance` 可以修改
3. Object.prototype.toString.call(): Javascript 所有对象都是基于 Object，如果对象的 toString 没有被重写的话，都是返回`[object Type]`，例如`[object Array]`；缺点：

   - 只能判断 Javascript 内置类型
   - `Symbol.toStringTag` 可以修改

4. Array.isArray: 判断数组类型，且可以判断 Frames；缺点：
   - 只能判断数组

### 模块化发展历程

#### IIFE

使用自执行函数解决命名冲突的问题，每个文件都是一个 IIFE 函数

#### ES6 Module

ES6 Module 使用 import 导入模块，使用 export、export default 导出模块

1. ES6 Module 内部自动严格模式
2. ES6 Module 加载的为**模块的引用**，即模块后续变化会产生影响
3. ES6 Module 无需解决循环引用的问题，只需要在模块使用时能找到即可
4. ES6 Module import 必须置于顶级作用域
5. ES6 Module import 为静态加载，无法使用变量，即无法在运行时确定（`import()可以`）

#### CommonJS

CommonJS 主要适用于服务端（Node.js），以 require 导入模块，以 module.exports、exports 导出模块，所有模块都包裹在

1. CommonJS 内部为松散模式
2. CommonJS 加载的是**模块的缓存**，模块执行后，后续所有加载都是从缓存读取
3. CommonJS 需要解决循环引用的问题
4. require 可以任意嵌套作用域
5. require 可以在运行时确定

```
function(require, module, exports = module.exports, __dirname, __filename){
  // 所有模块代码
}

```

### call、apply、bind

1. 传参: call、bind 需分开传参；apply 可以数组传参；且 this 参数如果为 null、undefined 话，在非严格模式下为 window
2. 返回值: call、apply 直接运行函数；bind 返回 Bind 函数，该函数是怪异函数，且 new 调用时，**忽略传入的 this 参数，且传入其他参数**，表现为直接传参调用原函数
3. 性能: call 性能较 apply 高一点

```
Function.prototype.call = function (_this, ...args) {
  if (_this === null || _this === undefined) {
    _this = window;
  }

  if (typeof _this !== 'object') {
    _this = new Object(_this);
  }

  _this.fn = this;
  _this.fn(...args);
};
```

```
Function.prototype.bind = function (_this, ...args) {
  if (_this === null || _this === undefined) {
    _this = window;
  }

  if (typeof _this !== 'object') {
    _this = new Object(_this);
  }

  _this.handle = this;

  return function BindFunction(...bindArgs) {
    return _this.handle(...args.concat(bindArgs));
  }
};
```

### 箭头函数和普通函数区别

1. this: 普通函数 this 为谁调用该函数；箭头函数按作用域确定
2. arguments: 普通函数存在 arguments 属性；箭头函数按作用域确定
3. [[constructor]]、prototype: 普通函数存在该俩属性，且可以 new 调用；箭头函数不存在该属性，且不可以 new 调用
4. yield: 普通函数称为 Generator 函数；箭头不可以

### input 防抖和中文输入

```
let timer = null;
input.addEventListener('input', ev => {
  clearTimeout(timer);
  timer = setTimeout(() => {
    // 进行一些操作
  }, 200);
});
```

中文输入触发: compositionstart、compositionupdate、compositionend 事件。composition\*事件只在需要再次输入确认时触发，例如中文选词，语音输入，备用词等

```
let tag = true;
let timer = null;
i.addEventListener('compositionstart', function () { tag = false });

i.addEventListener('compositionupdate', function (ev) { console.log('正在输入...'); })

i.addEventListener('compositionend', function (e) {
  tag = true;
  const ev = document.createEvent('HTMLEvents');
  ev.initEvent('input');
  e.target.dispatchEvent(ev);
});

i.addEventListener('input',function(ev){
  // 此时才出发输入事件
  if(tag){

    // 那么如果不支持composition事件勒，那么input事件就会一直触发，所以这里开启防抖
    clearTimeout(timer);
    timer = setTimeout(() => {
        // 干点啥
    }, 200);
  }else{
    ev.preventDefault();
  }
})
```

### 前端常见加密场景和加密方式

1. base64 编码，骗骗不懂行的人
2. XOR 加密，利用 `A ^ B ^ B === A`的原理，剂型异或加密，可用于传输密码等操作
3. Hash 算法，对密码等隐秘信息显示、日志、存储时，需要进行 Hash 运算，不可明文
4. 数字证书、非对称加密、对称加密: HTTPS
5. 反爬虫混淆: 例如展示数据为 "ABC"，此时后端返回数据为 `CBA => { C: 'A'，B:'B',A:'C' } => ABC`

### JS "True、False" 规则

**False**

1. false => false
2. null、undefined => false
3. ''、'0' => false
4. 0、NaN => false

**True**

其余都为 True，所有对象都为 True(尤其是[])

### 对象 valueOf、toString

1. 基本对象的包装对象 Number、Boolean、String: valueOf 对应的基本类型；toString 对应的基本类型的字符串表示
2. Date: valueOf 对应的时间戳；toString 对应自身表示的标准时间字符串
3. Array: valueOf 对应自身；toString 表示数组形式字符串，**注意，不存在"[", "]"**
4. RegExp: valueOf 自身；toString 表示正则形式的字符串 "/xxx/"
5. ...
6. 其余的未重写 valueOf 方法和 toString 方法的对象: valueOf 自身；toString [object Object]

## Node

### npm

#### npm 依赖

##### dependencies

生产依赖，在执行 `npm install --production`下载包

##### devDependencies

开发依赖，在执行`npm install --only=dev` 可以只下载开发依赖

##### peerDependencies

对等依赖: 表示当前项目、包只在 peerDependencies 声明的包环境下有效

##### optionalDependencies

可选依赖: 表示当前项目、包即使在 optionalDependencies 依赖安装失败下，也可以正常构建和使用

##### bundledDependencies

打包依赖: 在执行`npm pack`后，将项目打成压缩包，也可以下载的依赖

##### npm 依赖版本号

npm 版本号分为: 主版本号.次版本.补丁号，以正整数来命名

1. 以符号表示

   - ~: 表示 对补丁号的约束，^1.1.1
   - ^: 表示 对次版本号约束 ~1.1.1
   - -: 表示范围区间 1.1.1 - 1.2.1

2. 以运算符号表示

   - <、<= : 小于、小于等于:
   - _> 、>=_ : 大于、大于等于
   - =: 等于某个版本

3. 无特殊符号: 表示完全匹配版本，且如果版本号为标记完整，则以"x"补充，例如 `1.0.0、 1 表示的是 1.x.x，也可表示为 ^1.0.0 或 >=1.0.0 <2.0.0`

## CSS

### BFC

BFC: 块级格式上下文。BFC 是一个封闭的容器，内部元素如何排列均匀外部无关

BFC 特点:

1. 容器封闭、内部元素排列与外部无关
2. 内部垂直 margin 折叠
3. BFC 可以获得浮动元素高度

创建 BFC:

1. html、body 元素
2. overflow 非 visible
3. float 非 none
4. display: flex、inline-block
5. position: absolute，fixed
6. ...

### 元素居中

1. 子元素: position + margin 负值
2. 子元素: position + transform 负值
3. 父元素: flex 布局
4. 父元素: grid 布局
5. 父元素: flex 布局; 子元素: margin: auto
6. 父元素: text-align: center; line-height: 父元素高度; 子元素: display: inline-block

#### Flex

弹性布局，将空间按主轴切割，子元素在主轴分开排列

```css
display: flex、inline-flex;

justify-content: 指定主轴对齐方式
align-items: 交叉轴元素对齐方式
...
```

#### Grid

网格布局，将空间切分为网格，将元素放入网格中

```css
display: grid、inline-grid
justify-content: 指定主轴对齐方式，横向
align-items: 交叉轴元素对齐方式，纵向

grid-template-columns: 定义一列的宽度，可使用数值，百分比，repeat，fr，auto 等
grid-template-rows: 定义一行的高度，~
```

GRID 实现 Nav，Header，Content，Footer 经典布局

```
<style>
 body, html {
      height: 100%;
    }

    .container {
      height: 100%;
      width: 100%;

      display: grid;
      grid-template-columns: 200px auto;
    }

    .right {
      display: grid;
      grid-template-rows: 100px auto 100px;
    }

    .content {
      border: 1px solid #000;
    }
</style>
<div class="container">
  <div class="left"></div>
  <div class="right">
    <div class="top"></div>
    <div class="content"></div>
    <div class="footer"></div>
  </div>
</div>
```

#### opacity:0；visibility:hidden；display:none

1. 实现原理: opacity 修改元素透明度；visibility 将元素从 render 树删除，但未从 DOM 节点删除；display 从 DOM 节点删除
2. 元素占位: opacity 元素占位；visibility 元素占位；display 元素不占位，彻底删除
3. 子集影响: opacity 子集强制生效；visibility 子集默认生效，但可以通过 visibility: visible 显示；display 子集强制生效
4. 性能: opacity 启动硬件加速，性能最佳；visibility 触发重绘；display 触发重排

#### 实现文本增加...

1. css 实现单行文本

```
overflow:hidden;
text-overflow: ellipsis;
white-space: nowrap;
```

2. JS 实现多行文本

原理是通过，clientHeight 和 scrollHeight 比较

```
  let words = '';

  words += '...';
  while (p.clientHeight < p.scrollHeight) {
    p.innerHTML = words.slice(0, -4) + '...';
  }
```
