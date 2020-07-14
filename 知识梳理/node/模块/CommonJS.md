# CommonJS

## 什么是 CommonJS

CommonJS 是一套 Javascript 模块规范，我们常说的 CommonJS 也是指的这一套规范。

CommonJS 是越来越多的标准集合，包含

1. Modules
2. 二进制字符串或 Buffer
3. 二进制、Buffer 以及文本 I/O 流
4. File I/O
5. 系统进程参数、环境、流
6. 文件系统接口
7. Socket Stream
8. 测试、断言
9. WEB 服务器网关
10. 本地包和远程包的管理

...

## 为什么会出现 CommonJS

在 ES6 后，Javascript 有了自己的模块规范，但是在当时是缺乏普遍可接受的 Javascript 模块机制，CommonJS 应运而生

CommonJS API 将通过定义满足许多常见应用程序需求的 API ，来提供与 Python，Ruby 和 Java 一样丰富的标准库。目的是使应用程序开发人员能够使用 CommonJS API 编写应用程序，然后在不同的 JavaScript 解释器和主机环境中运行该应用程序。在与 CommonJS 兼容的系统中，可以使用 JavaScript 编写

1. 服务端 Javascript 应用程序
2. 命令行工具
3. 桌面 GUI 应用
4. 混合应用

值得一提的是，CommonJS 不仅仅是在服务端可以使用，也可以在浏览器端（客户端）、桌面端使用，可能是因为 Node 太火了，所以导致很多人都认为 CommonJS 只能运行于服务端

> 这个项目由 Mozilla 工程师 Kevin Dangoor 于 2009 年 1 月发起，最初名为 ServerJS。在 2009 年 8 月，这个项目被改名为“CommonJS”来展示其 API 的广泛的应用性。有关规定在一个开放进程中被创建和认可，一个规定只有在已经被多个实现完成之后才被认为是最终的。 CommonJS 不隶属于致力于 ECMAScript 的 Ecma 国际的工作组 TC39，但是 TC39 的一些成员参与了这个项目

> 在 2013 年 5 月，Node.js 包管理器 npm 的作者 Isaac Z. Schlueter，宣布 Node.js 已经废弃了 CommonJS，Node.js 核心开发者应避免使用它。[Breaking the CommonJS standardization impasse](https://github.com/nodejs/node-v0.x-archive/issues/5132#issuecomment-15432598)

### CommonJS 优势

1. CommonJS 一个文件即为一个模块，模块内定义的变量、函数...等都是私有的
2. 作用域隔离，不同模块命名不会冲突，且如果不是如下写法，不会污染全局作用域

   ```javascript
   a = 2; // 非严格模式下，不声明直接使用
   global.b = 3; // 全局变量直接赋值

   var c = 2; // 这样写在浏览器端污染全局作用域，但node不会
   ```

3. 文件缓存，模块重复使用
4. 模块按照代码出现的顺序加载，不需要再考虑 script 先后顺序问题

**以下的代码和内容都是 Node 的观点出发**（关键别的我也不知道啊）

## Module

```javascript
// node-master/lib/internal/modules/cjs/loader.js
function Module(id = '', parent) {
  this.id = id;
  this.path = path.dirname(id); // 目录名
  this.exports = {}; // 模块对外的值
  this.parent = parent; // 父模块，如果为main模块，则父模块为null
  updateChildren(parent, this, false);
  this.filename = null; // 文件名，文件的绝对路径
  this.loaded = false; // 文件是否加载完毕
  this.children = []; // 子模块
}
```

Node 中 每个模块都是 Module 的实例，表现为 `const module = new Module(filename, parent);`

这里值得一提的是，模块内部会提供一些对象，例如 `require，module，exports，__filename，dirname`，这些都可以在模块内直接使用，但**它们不是挂载 global 对象上的全局属性、方法**，至于他们是如何产生的，下面会详细讲解

### exports

exports 属性为 Module 实例的 exports 属性，表现为 `exports = module.exports;`

1. 当 exports 未重新赋值时，例如如下使用 `exports.name = 2;`，exports 和 module.exports 相等
2. 当 exports 重新赋值时，例如如下使用 `exports = {};`，exports 和 module.exports 不相等

请注意，无论 exports 如何变化，无论指向谁，模块最终对外的数据永远是 module.exports，而不是 exports

```javascript
// node-master/lib/internal/modules/cjs/loader.js
Module._load = function (request, parent, isMain) {
  {
    ...
     return cachedModule.exports;
  }
  ...
  {
    ...
    return cachedModule.exports;
  }
  ...
  return module.exports;
};
```

#### require

require 函数主要用来加载模块，表现为 `const mod = require(modulePath)`，加载完成后，返回模块的 exports 属性

由于此文不是主要分析模块加载，所以不会对 require 详细分析，即不会分析 makeRequireFunction 函数，而是直接取其中的核心代码 Module.prototype.require，也不会介绍 路径分析 和文件定位，这个在 [node 文档有详细的说明](http://nodejs.cn/latest-api/modules.html#modules_modules)

```javascript
Module.prototype.require = function (id) {
  validateString(id, 'id');
  ...
  return Module._load(id, this, /* isMain */ false);
  ...
};
```

自 require 方法校验完 id（模块名）后，进入 `Module._load` 时，一般分为如下几个步骤

1. 路径分析
2. 文件定位
3. 文件运行并缓存

由于一步步介绍起来过于复杂，所以会省略部分中间步骤

直接从文件编译运行开始说起

1. 会根据文件类型，去执行相应的方法编译，对于 js 来说，运行如下代码

```javascript
Module._extensions[extension](this, filename);

Module._extensions['.js'] = function (module, filename) {
  ...
  const content = fs.readFileSync(filename, 'utf8');
  module._compile(stripBOM(content), filename); // stripBOM 删除字节顺序标记
};
```

2. 此处正式进入 Javascript 模块编译阶段

   - 调用 CPP CompileFunction 方法，返回对象 compiled { function: compiledWrapper, cacheKey: xx }

```javascript
Module.prototype._compile = function (content, filename) {
  ...
  compiled = compileFunction(
    content,
    filename,
    0,
    0,
    undefined,
    false,
    undefined,
    [],
    ['exports', 'require', 'module', '__filename', '__dirname']
  );
  ...
  compiledWrapper = compiled.function;
};
```

- 编译完成后，准备运行参数

```javascript
  ...
  const dirname = path.dirname(filename);
  const require = makeRequireFunction(this, redirects); // 暂时不分析 makeRequireFunction
  var result;
  const exports = this.exports; // 所以说 exports = module.exports勒
  const thisValue = exports;
  const module = this;
```

- 运行

```javascript
...
// 由于call传入thisArgs为thisValue，所以模块内部顶级作用域的 this === module.exports
// 所以有时候，你可以使用 this 来赋值，但不建议
result = compiledWrapper.call(
  thisValue,
  exports,
  require,
  module,
  filename,
  dirname
);
...
```

- 运行完毕后，由于引用赋值的原因，对于 exports/module.exports 的赋值

```cpp
// node-master/src/node_contextify.cc
void ContextifyContext::CompileFunction(const FunctionCallbackInfo <Value> &args) {
  // params 就是 compileFunction 传的  ['exports', 'require', 'module', '__filename', '__dirname']
  // source 就是 content
  ...
  Local <ScriptOrModule> script;
  MaybeLocal <Function> maybe_fn = ScriptCompiler::CompileFunctionInContext(
      parsing_context, &source, params.size(), params.data(),
      context_extensions.size(), context_extensions.data(), options,
      v8::ScriptCompiler::NoCacheReason::kNoCacheNoReason, &script);

  ...

  // 经过 ScriptCompiler::CompileFunctionInContext 生成函数句柄
  Local <Function> fn = maybe_fn.ToLocalChecked();

  ...

  // 创建 result 对象句柄 const result = {};
  Local <Object> result = Object::New(isolate);

  // result.function = fn;
  if (result->Set(parsing_context, env->function_string(), fn).IsNothing())return;

  // result.cacheKey = cache_key;
  if (result->Set(parsing_context, env->cache_key_string(), cache_key).IsNothing())return;

  // 返回 result
  args.GetReturnValue().Set(result);
}
```

3. 进入缓存

```javascript
Module._load = function (request, parent, isMain) {
  ...
  Module._cache[filename] = module;
  ...
  return module.exports;
};
```

讲真，Module.\_cache[filename] = module; 其实运行前于 module.load(filename)，但还是将缓存放在勒最后一步，感觉比较符合正常思路

4. 返回 exports

至此为止，require 加载完毕

[commonjs-effort-sets-javascript-on-path-for-world-domination](https://arstechnica.com/information-technology/2009/12/commonjs-effort-sets-javascript-on-path-for-world-domination/)  
[commonjs.org](http://www.commonjs.org/)  
[CommonJS 规范](https://javascript.ruanyifeng.com/nodejs/module.html)
