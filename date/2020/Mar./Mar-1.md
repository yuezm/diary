## Javascript

### 位运算模仿加法运算

1. 使用 `&` 计算需要进位的数
2. 使用 `^` 计算无需进位的数

```javascript
function bitAdd(n, m) {
  while (n > 0) {
    const _n = (n & m) << 1; // 将需要进位的数取出来并进位
    const _m = n ^ m; // 将无需进位的数加起来

    n = _n;
    m = _m;
  }
  return m;
}
```

#### 不使用加减乘除计算 n 的 7 倍

1. 使用位运算模拟 `bitAdd`
2. 使用七进制位运算

```javascript
function t7(n) {
  return parseInt(n.toString(7) << 1, 7);
}
```

3. 使用 eval

```javascript
function t7(n) {
  return eval('n * 7');
}
```

4. 双层循环，向数组推入值，计算数组长度

```javascript
function t7(n) {
  const arr = [];

  for (let i = 0; i < n; i++) {
    for (let j = 0; j < 7; j++) {
      arr.push(null);
    }
  }
  return arr.length;
}
```

### SPA、SSR、预渲染

#### SPA

单页面应用，使用客户端渲染，即 HTML 是由 Javascript 加载运行后生成

**优点**

1. 用户交互友好，无需重新加载界面
2. 前后端分离明显，利于分工合作
3. 客户端渲染，减少服务器资源

**缺点**

1. SEO: 由于部分搜索引擎不能良好支持 Javascript 运行时，所以对 SEO 不友好
   - 预渲染，对于需要 SEO 的部分界面使用预渲染
   - 服务端渲染
   - 做引导界面
   - 针对 spider 做一套模板
2. 首屏渲染: 由于需要加载 Javascript，由 Javascript 生成 HTML，所以存在白屏时间和首屏渲染时间增长
   - 代码压缩，包含 CSS、HTML、JS 压缩；图片压缩等
   - 路由懒加载
   - 按需加载、**后编译**
   - CDN
   - 使用缓存
   - 服务多入口
3. 浏览器部分功能，例如前进后退按钮无法实现（ps **在支持 history API 情况下，vue-router 已经解决了**）
   - 使用 History API 自身实现

#### 预渲染

在浏览器请求之前，使用无头浏览器执行代码后，生成的静态 HTML

**优点**

1. 配置简单，与 SSR 完美结合
2. SEO
3. 加载速度快

**缺点**

1. _无法动态生成界面_
2. 对于大量路由来说，预渲染时间会很长

#### SSR

服务端渲染，在服务器渲染出 HTML，向浏览器直接传输的 HTML

**优点**

1. SEO
2. 首屏渲染快速

**缺点**

1. 增加服务器资源
2. 增加开发难度，比如某些 API 在 SSR 需要注意

### 跨域解决

#### JSONP

1. 使用 script 标签加载服务器资源
2. 服务器返回由函数包裹的字符串 `"fn({....})"`,浏览器解析后，认为是 _fn 函数执行_，执行参数为所传入的 JSON 数据

**缺点**

1. XSS 攻击: 由于 JSONP 一般来说都不会将函数名写死，而是由前端传递，所以可能出现 XSS 攻击`?jsonpname=<script>alert(1)</script>`

   - 对 JSONP 函数名长度检测
   - 字符转义

2. 非标准方法，属于 hack

#### 跨域资源共享

客户端发起请求时，服务器返回指定的头部信息，如果头部信息符合，则可以跨域

##### 简单请求

请求头部仅为指定的字段，常用的如下所示

```
Method: GET、HEAD、POST
Content-Type: application/x-www-form-urlencoded、multipart/form-data、text/plan
Origin: xx
Accept: xx
Accept-Language: xx
...
```

此时为简单请求，简单请求时，服务只需要范围如下头部，即可进行跨域

```
Access-Control-Allow-Origin: * 或者 客户端发送的Origin值
```

##### 预检请求

请求头部为指定的字段，常用的如下所示

```
Method: PUT、OPTIONS、TRACE、DELETE、CONNECT
Content-Type: 非简单请求指定的三个值，例如 application/json
...
```

1. 如果为预检请求，则客户端先试用 **OPTIONS** 发起预检请求，服务端必须返回如下头部，即可进行跨域

```
Access-Control-Allow-Origin: * 或 客户单发送的Origin值
Access-Control-Allow-Methods: * 或者指定的请求方法，如 "POST,GET"
Access-Control-Allow-Headers: * 或者指定的头部key，如 "Content-Type，Origin"
Access-Control-Max-Age: 秒数 该头部为非必须返回，表示该次预检请求多少秒内有效！！！
```

2. 客户端下次请求会带上如下字段

```
Access-Control-Request-Method: POST
Access-Control-Request-Headers: xx
...
```

3. 服务端必须返回如下头部

```
Access-Control-Allow-Origin:  * 或 客户单发送的Origin值
```

大多数浏览器不支持预检请求的重定向，**但在后续版本中废弃该行为**

1. 浏览器去掉对预检请求的重定向
2. 将预检请求变为简单请求

#### 反向代理

使用 nginx 代理

#### window.postMessage

新 API,使用 window.postMessage 可以对其他域的的 window 发送消息

```
window.postMessage(message, targetOrigin, [transfer]);

window.on('message',ev=>{
  // ev.data
  // ev.origin
  // ev.source
})
```

### MVVM 和 MVC

#### MVC

分离视图和模型

1. View 发生变化时，通知 Controller
2. Controller 将数据简单处理后，交于 Model
3. Model 处理完毕后，使用**观察者模式**通知 View
4. _View 获得通知后，向 **Model** 获得更新_

**优点**

1. 视图和业务逻辑分离，较好维护
2. 观察者模式可以做到多个视图更新

**缺点**

1. View 和 Model 耦合较多，对于抽取模块较为困难
2. 测试困难，View 存在于 UI 下，而对 Controller 单元测试时，不存在 UI，有些逻辑无法验证

#### MVVM

1. View 发生改变，通知 Virtual Model
2. Virtual Model 通知 Model 处理
3. Model 处理完成后，使用**Virtual Model Binder**使 View 发生改变

**优点**

1. View 和 Model 解耦，可单独抽离模块，
2. View 不依赖于 Model，可单独测试
3. 自动更新（使用 Binder，预先对 View 和 Model 建立勒绑定关系），增加可维护性

**缺点**

1. 模型较大，对于小应用不适合
2. 对于大型系统而言，视图状态较多，VM 维护成本较高

## 性能

### 首屏优化

1. 代码压缩（包含图片压缩）
   - Vue Cli 提供 css、js、html 压缩
   - 使用 *webpack-image-loader*压缩图片
   - 使用 postcss 合并小图片
2. 按需加载、后编译
   - babel-plugin-import 插件
3. 路由懒加载
4. CDN
5. 多入口拆分
6. 缓存: 1. webpack 缓存拆分、大宝；2. 浏览器缓存；3. 使用 prefetch 和 preload 优化
7. 预渲染
8. 服务端渲染
9. 菊花图和骨架屏

后编译: 现在的代码大多数都是在 ES6+下开发的，由 webpack+babel 编译为 ES5 代码，其它包也一样。但在打包时，会产生**冗余代码**，且这些冗余代码无法经过 Tree Shaking 去除。所以提出**不要单独编译依赖包，而在工程编译时，一同编译**

### 重排、重绘及优化

浏览器加载路线

1. 解析 HTML，生成 DOM Tree
2. 解析 CSS，生成 CSS Tree
3. 合并 CSS Tree + DOM Tree，生成 Render Tree
4. 经过 **Layout(reLayout 重排)**，计算元素的定位和布局
5. 经过 **Paint(rePaint 重绘)**，计算元素的颜色，背景
6. 经过 Display，GPU 渲染显示

**重排:** 元素布局改变，需要重新计算元素的定位和布局。例如盒子大小，字体大小，浮动等

**重绘:** 元素布局未改变，无需重绘计算布局，但需要重新计算像素点。例如颜色改变

**优化**

1. 尽量将多次合并，例如插入 DOM 时，可以使用 Fragment，暂存，然后一步放入 DOM 中
2. 对频繁操作时，可创建复合图层(translate3d，translateZ，opacity)
3. 对于大量 DOM 操作，可以先将元素`display: none`，操作完成后再设置`display: block`，当元素隐藏时，任何操作都不触发重排、重绘
4. 不要使用 table 布局，table 布局很特殊，他的重排和重绘成本高于普通元素
5. 对于引发重排、重绘的元素，对其缓存
   - offsetTop、offsetLeft、offsetWidth、offsetHeight
   - scrollTop、scrollLeft、scrollHeight、scrollWidth
   - client...
   - getComputedStyle
6. 浏览器对重排和重绘也存在优化，会将任务放入队列中，刷新队列，而不是单独执行

### clientWidth、Height，offsetWidth、Height，scrollWidth、Height

```
clientWidth、Height: 可视区域宽高，包含 content + padding
offsetWidth、Height: 元素宽高，包含边框
scrollWidth、Height: 元素宽高，包含看不见的部分

clientLeft、Top: 指到可视区的距离（以父级的content为基准）
offsetLeft、Top: 以最近的定位父级的距离（以父级的padding为基准）
scrollLeft、Top: 父级的滚动距离（子元素超出时，滚动条出现在父级，设置也是父级的滚动距离）
```

## 浏览器

### 缓存

浏览器缓存一般分为四种: Service Worker 缓存，Memory 缓存，Disk 缓存，HTTP2 Push 缓存

#### Service Worker Cache

专属于 Service Worker 缓存，主要特点如下

1. 缓存受 Service Worker 控制，可以手动控制是否缓存和是否使用缓存
2. Service Worker 可拦截浏览器请求，强制使用缓存
3. 只存在于 Service Worker 中，且在 Service Worker 注册激活后可使用

#### Memory Cache

1. 存储于内存中，读取速度快，但存储量有限
2. 无法手动控制缓存，由浏览器控制
3. 关闭 Tab 时，清除内存缓存

#### Disk Cache

1. 存储于磁盘中，存储速度较内存慢，但存储量大
2. 无法手动控制缓存，由浏览器控制
3. 关闭 Tab 不会清除硬盘缓存

_一般来说，大文件都存储于 Disk Cache 中，而当 Memory Cache 满了也会将文件存储于 Disk Cache 中_

#### Http2 Push Cache

1. 存在于 HTTP2 会话内，由客户端和服务端各自维护自身缓存
2. 随着 HTTP2 断开，缓存清除
3. 无法手动控制缓存，由浏览器控制
4. 缓存时效性很短，且缓存只可以使用一次

#### 缓存头部

缓存主要分为: 强缓存，协商缓存

1. **强缓存 > 协商缓存**
2. 强缓存: pragma: no-cache > Cache-Control > Expires
3. 协商缓存: ETag > Last-Modified

##### 强缓存

强缓存: 在缓存有效期内，浏览器无需和服务器通信，直接使用缓存

```
Expires: 时间字符串
Cache-Control: public、private、no-cache、no-store、max-age
```

###### Expires

HTTP1.0 产物，用于设置浏览器缓存时间

Expires: Wed, 21 Oct 2015 07:28:00 GMT

**优点**

1. 在不支持 HTTP1.1 的浏览器可使用

**缺点**

1. Expires 是根据客户端时间判断的，而客户端时间可能与服务端时间不一致
2. 无法设置相对时间

###### Cache-Control

HTTP1.1 新增头部，**Cache-Control 优先级高于 Expires**

- public: 任何客户端都可以缓存
- private: 只有目标客户端可缓存，中间代理不允许缓存
- no-cache: **客户端可缓存**，但是否使用需要和服务端通信
- no-store: 不可缓存
- max-age: 秒数
- max-stale: 秒数，表示可使用过期资源，但过期时间不能大于该时间

此时响应头状态码为 200（from memory cache 或 from disk cache）

**优点**

1. 更多的控制选项，且可以设置相对过期时间

**缺点**

1. 在无法支持 HTTP1.1 浏览器无法使用

##### 协商缓存

###### Last-Modified

标记文件最后的修改时间，如果在改时间后文件进行了修改，则需要更新

1. 服务端返回头部: Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
2. 客户端请求时，需带上请求头 If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
3. 如果服务端确认在此时间后，文件无修改则返回状态码 304；否则返回新的文件，状态码为 200

**If-Modified-Since 和 If-Unmodified-Since**

If-Modified-Since: 向服务器确认资源是否改变，改变则传输(200)；未改变则返返回 304  
If-Unmodified-Since: 向服务器确认该资源是否未改变，**改变则返回 412**；未改变则传输(200)

If-Unmodified-Since: 主要使用于**范围传输**。举个例子，如果你想获得文件 1000-2000 字节的数据，但此时文件发生了改变，那么此次获取就毫无意义

**优点**

1. 相较于 E-Tag，Last-Modified 不需要额外的结算消耗

**缺点**

1. 时间只能精确到秒，如果再 1 秒内文件改变了，Last-Modified 无法正确判断
2. 无法判断文件是否修改了，例如只是打开文件而不做修改（新增数据然后又删掉），Last-Modified 也会变更

###### ETag

由服务器根据文件内容计算出 Hash 值，传输给客户端，只有资源存在变更，则 Hash 值也会变更，**E-Tag 在服务端校验时，优先级高于 Last-Modified**

1. 服务端返回头部 ETag: xxxx
2. 客户端再请求时，需带上请求头 If-None-Match: xxx
3. 服务器会比较当前 Hash 值与文件计算所得 Hash 值是否相等，相等则返回 304；不相等则返回文件内容(200)

**If-None-Match 和 If-Match**

If-None-Match: 如果文件未改变则返回 304；改变则返回文件(200)  
If-Match: 如果文件改变则返货 412；未改变则继续传输

**优点**

1. 控制粒度较 Last-Modified 精细

**缺点**

1. 需要额外的计算资源

#### 实际缓存策略

1. 如果**未设置缓存过期时间**，浏览器不会马上发起协商请求，而是采用**启发式算法**，**取 Date 的值 - Last-Modified 时间的 10% 做为缓存时间**，tips ETag 不存在启发式算法
2. 对于频繁变动资源，使用 Cache-Control:no-cache + 协商缓存
3. 对于不经常变动资源: Cache-Control: max-age=31536000
4. 用户在 Tab 页面刷新时
   - 普通刷新: 会优先读取 Memory Cache
   - 强制刷新: 请求头设置 Cache-Control: no-cache，并强制请求服务器
5. 用户离开后，输入 URL 访问: 优先读取 Disk Cache，否则发送网络请求

## Node

### hack require

**模块加载调用**

_每个模块会被 Wrapper 包裹，function(require, exports, module, **filename, **dirname){ xx }_

在调用 require 时，调用的是 Wrapper 传入的 require，而该 require 是由 `var require = internalModule.makeRequireFunction.call(this)` 创建

```
function makeRequireFunction() {
  const Module = this.constructor;
  const self = this;

  function require(path) {
    try {
      exports.requireDepth += 1;
      return self.require(path); // 真正调用
    } finally {
      exports.requireDepth -= 1;
    }
  }
  ...
}
```

这是 require 调用顺序，`require => Module.prototype.require => Module.\_load => { ... Module.\_resolveFileName ... }`，所以有很多地方可以劫持函数

例如劫持 Module.prototype.require

```javascript
const Module = require('module');
const path = require('path');
const pathsStore = require('./package.json').alias;

for (const attr in pathsStore) {
  pathsStore[attr] = path.resolve(pathsStore[attr]); // 需要注意路径的问题
}

const _require = Module.prototype.require;
Module.prototype.require = function(path) {
  const firstPath = path.includes('/') ? path.splice('/')[0] : '';
  if (firstPath !== '' && pathsStore[firstPath]) {
    path = path.replace(firstPath, pathsStore[firstPath]);
  }

  return _require(path);
};
```

**注意点**

1. 注意 hack require 的位置
2. 注意路径问题，由于一般配置都使用的**相对路径**，在路径加载时容易出现问题，干脆转为绝对路径

## CSS

### 伪元素、伪类

伪元素和伪类都是对**文档树中不存在的部分**进行修饰

伪元素: 对 DOM 中不存在的元素进行操作，例如`::before`且会**在浏览器创建新的元素**  
伪类: 对文档树中不存在的状态进行操作，且该状态一般由用户触发，例如`:hover`等。**不会创建新元素**

**优先级: 伪类 > 伪元素**

### 层叠上下文

层叠上下文是一种特殊的渲染方式。和 BFC 一样，层叠上下文中的元素层叠水平与外部无关

#### 层叠水平

在同层叠上下文中，表示该元素的显示层次大小，越大则显示越靠前

`层叠上下文 < z-index: 负值 < block 盒子 < 浮动盒子 < inline、inline-block 盒子 < z-index: 0、auto < z-index: 正值`

1. 同一个层叠上下文中
   - 层叠水平越大，则靠前
   - 同层叠水平，则后面的元素靠前
2. 不同层叠上下文，元素无比较意义

tips: **元素的层叠水平本来就存在，而 z-index 仅仅是可以修改定位元素和 flex 布局元素的层叠水平而已**

tips: z-index: 0 和 z-index: auto 在**层叠水平表现一致**；但 **z-index:0 会创建层叠上下文**，而 **z-index: auto 不会创建层叠上下文**

[层叠上下文](https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)

## Vue

### vuex

vuex 提供一个大仓库来存储数据，他与普通数据存储的区别

1. vuex 内部数据为响应式的
2. vuex 提供 mutations 机制，只有 mutations 可以修改 state

vuex 响应式原理: 在 vuex 内部，存在一个 Vue 实例，state 数据是放在该 Vue 实例中，当数据发生变化时，则对 Watcher 派发更新

vuex 简单实现

```
import Vue from "vue";

class SimpleStore {
  constructor({ state, mutations }) {
    this.mutations = mutations;
    this._vm = new Vue({
      data: {
        $$state: state
      }
    });
  }

  get state() {
    return this._vm.$data.$$state;
  }

  set state(v) {
    console.warn("不允许直接修改state");
  }

  commit(event, v) {
    this.mutations[event](this._vm.$data.$$state, v);
  }
}

export default {
  install(Vue) {
    Vue.mixin({
      beforeCreate() {
        if (this.$options.store) {
          this.$store = this.$options.store;
        } else {
          this.$store = (this.$parent && this.$parent.$store) || {};
        }
      }
    });
  },
  Store: SimpleStore
};
```

#### vuex DIY store 插件

vuex 插件机制: 使用 plugins 提供的 Store 实例，订阅 mutation 触发，而 vuex 数据时必须通过 mutations 触发的，所以可以监测到数据

1. 在事件触发时，将数据存储于 localStorage 中，使用*localforage 替代*
2. 在初始化时，将存储的数据读取，并合并到 state 中，tips: **必须要使用 store.replaceState()**

```

```
