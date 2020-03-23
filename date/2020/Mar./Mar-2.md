## JS

### 执行上下文

执行上下文: 描述当前环境的执行状态。主要分为*全局执行上下文*和*函数执行上下文*

#### 分类

全局执行上下文: 在程序执行时，将全局执行上下文压入执行栈。全局执行上下文在整个执行程序中是唯一的，随着程序执行完毕后弹出  
函数执行上下文: 在进行函数调用时，会创建函数执行上下文，压入执行栈，待函数执行完毕后弹出  
eval 执行上下文

#### 阶段

**创建**

1. 初始化变量环境组件，例如变量声明、函数声明
2. 初始化作用域链
3. 决定 this 指向

**执行**

按照代码顺序，依次执行

**销毁**

从执行栈中弹出

### History API

History API 可以操作浏览器历史记录，例如控制浏览器前进后退按钮

#### pushState

```
history.pushState(state, title[, url]);
```

向浏览器推入历史记录，且不触发 popstate 事件

#### replaceState

```
history.replaceState(state, title[, url]);
```

使用目标记录，覆盖当前的浏览器历史记录，且不触发 popstate 事件

#### go、back、forward

```
history.go(n);
history.back() === history.go(-1);
history.forward() === history(1);
```

操作浏览器历史记录前进后退

### preload、prefetch、defer、async

preload、prefetch、defer、async: 都可以异步加载资源

#### preload 和 prefetch 区别

1. 语义: preload 为预加载，表示该资源当前界面会使用；prefetch 为预取，表示该资源可能会使用
2. 加载时机: preload 表示立即加载，加载完毕后不会立即解析；prefetch 为预取，是否加载资源由浏览器实现
3. 资源优先级: preload 使用 as 控制优先级*style>script>font>image>audio>video*，如果 as 为空，则认为是 XHR，优先级最低；prefeth 无优先级控制

**不要 preload 和 prefetch 对同一个资源使用，会出现重复加载**；**prealod 虽然会立即加载，但并不会立即执行，而是等到需要的时候才会执行**

如果想立即执行 css

```
<link rel="preload" href="xx" as="style" onload="this.rel='stylesheet'">
```

#### async 和 defer 区别

1. 语义: async 为异步加载；defer 为延时加载
2. 执行时机: async 加载完毕后立即执行，且会阻塞 HTML 解析；defer 等待 DOM 解析完毕后执行，**规定在 DOMContentLoaded 前执行完毕**
3. 是否有序: async 无序；defer **规定是有序**

#### preload 和 defer、async 异同

1. 加载时机: 都是立即开始下载
2. 执行时机: preload 在使用时才会执行；async 加载完毕后立即执行；defer 在 DOMContentLoaded 前执行完毕
3. 资源类型: preload 可以加载脚本、css、font 等等；async、defer 只能用于脚本
4. 资源优先级: preload 可以控制优先级；async、defer 不可以控制

## 性能

### Vue 优化实践

#### 编译优化

**dev 编译优化**

1. 使用 DLL 加速本地编译，将常用的、不经常变化的包，例如*vue,vue-router,vuex,axios...*打成一个 DLL

**prod 编译优化**

1. 代码压缩，JS、CSS、HTML 压缩由 vue-cli 已经完成，加入了 image 压缩（_image-webpack-loader_）

_使用 mozjpeg, optipng 等，需要安装新的包，请按照使用安装包 ps: npm install mozjpeg_

```
config.module
  .rule('images')
  .use('image-webpack-loader')
  .loader('image-webpack-loader')
  .options({
    mozjpeg: {
      progressive: true,
      quality: 65,
    },
    optipng: {
      enabled: false,
    },
    pngquant: {
      quality: [0.65, 0.9],
      speed: 4,
    },
    gifsicle: {
      interlaced: false,
    },
    // the webp option will enable WEBP
    webp: {
      quality: 75,
    },
  })
  .end();
```

2. **代码切割**
   - optimization.runtimeChunk: 抽离运行时代码
   - optimization.usedExports: tree-shaking，prod 默认启用
   - optimization.splitChunks: 抽离公共代码，将 node_modules 代码抽离为
   - HashedModuleIdsPlugin(webpack.HashedModuleIdsPlugin): 对 vendor hash 修正
   - prepack-webpack-plugin: 预执行，可以优化代码，**在使用时，如果包的 module.export 不为对象，则会报错** ps 使用 jquery 时

[缓存](https://webpack.docschina.org/guides/caching/)

3. **三方模块按需加载**

4. **使用 CDN**

- _index.html_ 部署于自身 nginx 服务器上，使用协商缓存；其余所有文件，按照路径部署于七牛云对象存储，使用强制缓存
- CDN 服务器配置，max-age 设置一年，如果文件出现修改，则根据 webpack 配置，文件名 hash 值会随之更改；开启 HTTP2

5. **外部资源 preload，prefetch**

大部分资源都是使用 webpack 打包后放入 CDN 的，也有少部分资源是直接使用 CDN，例如 JQuery，所以对于直接使用的外部资源进行判断:

- head 中使用 preload 获取，加载资源

6. 多入口
7. 预渲染
8. 服务端渲染

## CSS

### link 和 @import

**相同**

link 和 @import 都可以加载 css

**不同**

1. 加载资源: link 可以加载其余资源；@import 只能加载 css
2. 从属关系: link 属于 HTML 标签；@import 属于 css 模块语法
3. 兼容性问题: link 无兼容性问题；@import 在 css2.1 的语法
4. 资源优先级: link 可以使用 as 控制优先级；@import 无法控制资源优先级
5. 加载时机: link 可以随着页面解析而加载；@import 在页面加载完毕后才开始加载

_虽然 @import 是页面加载完毕后才开始加载，但是 @import 一般都在 css 文件头部使用，即使 @import 是处于最后才加载的，但 css 是头部引入的，会被后续 css 覆盖_

## 浏览器

### Cookie

HTTP 协议是无状态协议

**优势**: 服务端无序记录客户端状态，减少资源消耗，提高性能；**劣势**: 无法记录客户端状态，无法判断当前用户

所以使用 Cookie 来识别客户端

#### sameSite

用于限制 cookie 传输的策略，可以有效防止 CSRF。sameSite 主要分为三个值

1. **Strict**: 严格限制，**除去与当前页面相同的域名**，其他域名请求时,均不带 cookie
2. **Lax**: 在 Strict 稍微松一点，除去导航的连接，其他请求时，均不会带上 cookie。例如超链接，GET 表单跳转，preload 会带上 cookie
3. **None**:无安全策略，但在 Google 浏览器中且支持 sameSite 时，如果设置 sameSite 为 None，则 cookie 必须为 secure

#### secure

安全 cookie，cookie 只在安全模式下传输**HTTPS**，在 HTTP 下不传输

#### httpOnly

cookie 只能以服务端操作，客户端无法使用 JS 操作和获取

## XHR

### withCredentials

在进行跨域请求时，一般响应返回*Set-Cookie*都输无效的，即无法设置 cookie, 无论 Access-Allow-\*设置为什么

但是，如果 XHR 请求属性中，设置 withCredentials: true，则跨域请求可以设置 cookie

## Vue

### Vue3.0 优化

1. 响应式优化 `Object.defineProperty() => Proxy`

   - 更好的监听数组的变化
   - 更好的监听属性，Proxy 无需遍历整个属性，但是如果想*监听深层属性变化，还是需要递归*，对于嵌套属性，虽然父级的 proxy 的 get 会触发，但无法准确描述是哪个子属性触发的
   - 新增子属性也可以被监听

2. 编译优化

3. 虚拟 DOM 优化

   - 基于 inferno 进行改进，例如将组件划类型划分更加细致，DIFF 算法优化
   - 更新性能与动态指令相关(vue2.0 中与整体模板大小相关，与动态指令无关)

4. Composition API
5. 代码量更小

## 网络

### HTTP

#### HTTP GET、POST

1. 语义: GET 获取资源；POST 传输实体
2. 数据大小限制: 浏览器对 GET URL 存在限制；POST 无限制
3. 编码: GET 为 ASCII 编码，如果 URL 存在非 ASCII 编码，则会进行转换；POST 多种编码
4. 缓存: GET 请求可缓存；POST 无法缓存
5. 安全: GET URL 直接获取，相较于 POST 来说不安全，虽然他们都不安全

## webpack

### webpack 编译速度优化

1. 多进程: 使用 thread-loader 或 happy pack，利用计算机多核优势
2. 缓存: cache-loader，将结果缓存到磁盘。保存和读取缓存会有额外的消耗，所以对开销较大的 Loader 使用较为合适，_vue-cli3 自带_
3. DLL: 动态链接库，将不经常变动的打包为链接库，打包时，链接进来

## LeetCode

### 零钱对换

### 最大公因数

### 字符串最大公因子

### 多数组

### 岛屿最大面积

```

```
