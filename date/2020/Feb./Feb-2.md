## Javascript

### Webpack

webpack 是构建工具，负责将文件打包，输出为指定的文件

1. **初始化**

   - 合并配置文件、shell 参数
   - 加载 Plugins
   - 初始化 Compiler，Compiler 负责文件监听和启动编译。Compiler 实例中包含了完整的 Webpack 配置，全局只有一个 Compiler 实例
   - 入口文件分析

2. 编译

   - 根据不同文件调用不同的 Loader
   - 寻找依赖，并根据不同文件调用不同的 Loader

3. 输出
   - 合并 Module 为 Chunk
   - 将 Chunk 输出为文件

#### Entry

```
entry: string | string[] | object
```

指定入口文件，webpack 根据入口文件寻找依赖

#### Module

模块，在 webpack 中，每个文件都是模块，从入口文件开始，深度递归寻找依赖（其他模块）

##### rules

匹配规则，根据指定的正则匹配文件名，如果匹配成功，则使用定义的 Loader 处理

```
{
  test: Regexp
  use: string[] | object[]...存在很多种Loader组合方式
  include: string[]
  exclude: string[]
  options: object
  parse: object 对文件进行更加细粒度的控制
}
```

##### noParse

指定不需要解析的文件

#### Loader

**webpack 核心之一**

由于 webpack 原生只支持 Javascript，那么对 CSS、图片等其他资源都无法处理，所以在对非 JS 需要借助对应的 Loader 处理

**Loader 特点**

1. 链式操作，前一个 Loader 执行的结果做为后一个 Loader 的输入，第一个 Loader 接收源码，最后一个 Loader 返回执行完毕的结果
2. 自 use 定义中，从后往前执行
3. 可以在加载时指定 Loader，例如`require('style-loader!css-loader!index.css')`

##### 自定义 Loader

1. 定义 Loader

```
module.exports = function(source){
  // load-util工具包 loaderUtil.getOptions(this); 获取配置
  // return source; 简单Loader
  // this.callback() Webpack 给 Loader 注入的 API，以方便 Loader 和 Webpack 之间通信
  /*
  this.callback(
    // 当无法转换原内容时，给 Webpack 返回一个 Error
    err: Error | null,
    // 原内容转换后的内容
    content: string | Buffer,
    // 用于把转换后的内容得出原内容的 Source Map，方便调试
    sourceMap?: SourceMap,
    // 如果本次转换为原内容生成了 AST 语法树，可以把这个 AST 返回，
    // 以方便之后需要 AST 的 Loader 复用该 AST，以避免重复生成 AST，提升性能
    abstractSyntaxTree?: AST
  );
  * */
  // this.cacheable(false); 清理缓存，webpack默认loader缓存
  // this.callback(null, source); 同步回调
  // callback = this.async(); 异步回调
  return source;
}
```

2. 将 Loader 加入 webpack
   - 通过 resolveLoader 属性指定 Loader 路径
   - 通过 module.rules.use 内指定 Loader 的绝对路径

#### Chunk

由多个 Module 合并为 Chunk

如果 entry 为数组和字符串，则 Chunk 只存在一个，名称为 main；如果 entry 为对象，则 Chunk 可能存在多个，且名称为对象定义的 key 值

#### Plugin

**webpack 核心之一**

webpack 提供插件机制，webpack 依赖 tapable 包，在生命周期中，广播生命周期事件。在插件中，可以监听这些事件，从而插入 webpack 流程中。目的在于解决 Loader 无法解决的事情

##### 自定义插件

### 编写 Plugin

tapable: tapable 包提供插件钩子机制

```
class TestPlugin {
  constructor(options) {
  }

  apply(compiler) {
    // plugin 方法
    compiler.plugin('emit', function (compilation, callback) {
      // console.log(com.chunks);
      compilation.chunks.forEach(function (chunk) {
        chunk.files.forEach(function (filename) {
          // compilation.assets 存放当前所有即将输出的资源
          // 调用一个输出资源的 source() 方法能获取到输出资源的内容
          compilation.assets[ filename ].source();
        });
      });
      callback();
    });


    // 声明周期钩子
    compiler.hooks.emit.tap('TestPlugin', function (compilation) {
      compilation.chunks.forEach(chunk => {
        chunk.files.forEach(file => {
          console.log(compilation.assets[ file ].source());
        });
      })
    })

  }
}
// 初始化时，实例化 TestPlugin，实例调动 apply 方法，获得 compiler 参数后，再调用 compiler.plugin(事件名称, 回调函数) 则可以监听广播事件
```

1. Compiler: 对象包含了 Webpack 环境所有的的配置信息，包含 options，loaders，plugins 这些信息，这个对象在 Webpack 启动时候被实例化，它是全局唯一的，可以简单地把它理解为 Webpack 实例；

   - compiler.apply(): 广播事件
   - compiler.plugin(): 监听事件

2. Compilation: 对象包含了当前的模块资源、编译生成资源、变化的文件等。当 Webpack 以开发模式运行时，每当检测到一个文件变化，一次新的 Compilation 将被创建。Compilation 对象也提供了很多事件回调供插件做扩展。通过 Compilation 也能读取到 Compiler 对象

##### 事件

[compiler 钩子](https://www.webpackjs.com/api/compiler-hooks/)

#### Output

输出文件配置

##### filename

占位符：[hash]-模块标识符 hash, [chunkhash]-chunk 内容的 hash, [name]-模块名称, [id]-模块标识符, [query]-模块的 query，例如，文件名 ? 后面的字符串

##### chunkFilename

chunk 的名称，指定运行过程中，生成的 chunk 的名称，而非最终生成文件名称，在**按需加载中**，必须配置该属性

###### path

输出目录，绝对路径

###### publicPath

加载其他资源路径，例如 cdn 资源

###### libraryTarget 和 library

构建一个可以被其他模块导入使用的配置选项

#### Bundle

打包后的文件，可能存在多个文件

##### 文件加载

###### 同步文件加载

1. 所有的*同步依赖模块*都在自执行函数参数中`modules`声明，模块内部所有加载操作都会由`__webpack_require__(moduleId)`执行，例如

```
// index.js
require('./a'); ==> __webpack_require__("./a.js");
```

2. 通过 `__webpack_require__(moduleId)` 加载模块，加载完毕后，将模块 exports 缓存进 `installedModules[moduleId] = module.exports` 对象中；再次执行加载该模块操作时，直接返回缓模块存。tips: exports 对象里面存储着 CommonJS 的 module.exports 或 exports、ES6 Module 的 export 或 export default
3. 由自执行函数，执行入口文件，从而根据依赖执行整个工程

###### 异步文件加载

1. webpack 识别 `import()` 方法，将该依赖模块单独打包
2. 异步依赖由 `webpackJsonp` 管理
3. 异步依赖由`__webpack__require__.e` 负责加载
   - 创建*script*标签，并设置 src
   - 待加载完毕后，将*script*标签加载到 document.head 中
4. 异步依赖加载完成后，再由 `代码内部__webpack__require__`调用

```
(function(modules){
  var installedModules = {};

  function __webpack_require__(moduleId){
    if (installedModules[moduleId]){
      return installedModules[moduleId].exports
    }
    ...
  }

  __webpack__require__.e = function(chunkId){
    ...
  }
  ....
  __webpack_require__('./index.js'); // 加载入口文件
})({
  './index.js': (fuction(module, exports){
    // 同步模块执行
    eval("...")

    // 如果存在异步模块的
    eval("__webpack_require__.e(0).then(__webpack_require__.bind(null,'./*.js')));....")
  }),
  './*.js': (function(module,exports,__webpack_require){
    eval("...");
  }),
  ...
})
```

```
// 异步模块

(window["webpackJsonp"] = window["webpackJsonp"] || {}).push([[0],{
  "*.js": (function(module, export){
    ...
  }),
}])
```

#### 其他

##### devServer

DevServer 会把 Webpack 构建出的文件保存在内存中（使用 writeToDisk 可将文件写入磁盘）,且 DevServer 不会理会 webpack.config.js 里配置的 output.path 属性

1. historyApiFallback:boolean | object: 方便 History API 使用

```
historyApiFallback: true
这会导致任何请求都会返回 index.html 文件，这只能用于只有一个 HTML 文件的应用。
如果你的应用由多个单页应用组成，这就需要 DevServer 根据不同的请求来返回不同的 HTML 文件，配置如下
historyApiFallback: {
  // 使用正则匹配命中路由
  rewrites: [
    // /user 开头的都返回 user.html
    { from: /^\/user/, to: '/user.html' },
    { from: /^\/game/, to: '/game.html' },
    // 其它的都返回 index.html
    { from: /./, to: '/index.html' },  ]
}
```

2. hot:boolean:
3. contentBase:string: DevServer HTTP 服务器的文件根目录，一般为项目根目录
4. headers:object:
5. host
6. port
7. allowedHosts: 配置一个白名单列表，只有 HTTP 请求的 HOST 在列表里才正常返回，使用如下
8. compress:boolean: 是否启用 gzip 压缩
9. open:boolean: 是否默认打开浏览器

##### resolve

1. alias:object : 别名
2. mainFields:string[]: 根据配置，读取三方模块的，不同环境下的模块
3. extensions:string[]: 解析文件时，尝试的后缀名
4. modules:string[]: 寻找模块的路径，默认为 node_modules
5. descriptionFiles:string[]: 配置描述第三方模块的文件名称，也就是 package.json 文件
6. enforceExtension:boolean: 导入模块是否强制后缀名
7. enforceModuleExtension: 和 enforceExtension 功能相似，但只对 node_modules 下的模块生效

#### webpack 优化

1. 缩小文件搜索范围:
   - module-rules-loader 配置 includes 指定范围
   - resolve 指定范围
2. DllPlugin: 动态链接插件
3. HappyPack: 使 webpack 具有多进程能力（webpack 最耗时的流程可能就是 Loader 对文件的转换操作，可使用 HappyPack 交个其他进程处理）
4. ParallelUglifyPlugin 替代 UglifyJS，具有多进程能力
5. CommonsChunkPlugin: 公共代码提取
6. babel-plugin-syntax-dynamic-import: 异步组件（例如 vue 的路由懒加载）
7. Prepack（prepack-webpack-plugin）: 增加代码运行效率
8. Scope Hoisting（ModuleConcatenationPlugin）: 分析模块依赖，将模块注入另一个模块，减少函数声明消耗
9. 输出分析: webpack --profile(记录下构建过程中的耗时信息) --json(以 JSON 的格式输出构建结果)
   - http://webpack.github.io/analyse/ 官方可视化分析工具
