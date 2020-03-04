## CSS

## Javascript

### 深度拷贝

```
function deepCopy(data) {
  // 深度遍历
  const deepCopyMap = new WeakMap();
  return _deepCopy(data);

  function _deepCopy(v) {
    if (isBaseType(v)) {
      return v;
    } else if (isFunction(v)) {
      return eval(v.toString());
    } else {
      // 去重和防止循环引用
      if (deepCopyMap.has(v)) {
        return deepCopyMap.get(v);
      }
      const res = Array.isArray(v) ? [] : {};

      deepCopyMap.set(v, res);
      for (let attr in v) {
        res[ attr ] = _deepCopy(v[ attr ]);
      }
      return res;
    }
  }
}

function deepCopyForWhile(data) {
  // 层次遍历
  if (isBaseType(data)) return data;
  if (isFunction(data)) return eval(data.toString());

  const queue = [];
  const deepCopyMap = new WeakMap();
  const res = Array.isArray(data) ? [] : {};

  // 加入起始队列
  for (let key in data) {
    queue.push({
      p: res,
      k: key,
      v: data[ key ],
    });
  }
  deepCopyMap.set(data, res);

  while (queue.length > 0) {
    const { p, k, v } = queue.shift();

    let r = null;
    if (isBaseType(v)) {
      r = v;
    } else if (isFunction(v)) {
      r = eval(v.toString());
    } else {
      if (deepCopyMap.has(v)) {
        p[ k ] = deepCopyMap.get(v);
        continue;
      }
      r = Array.isArray(v) ? [] : {};
      deepCopyMap.set(v, r);

      for (const key in v) {
        queue.push({
          p: r,
          k: key,
          v: v[ key ],
        });
      }
    }
    p[ k ] = r;
  }
  return res;
}

function isBaseType(data) {
  return !(data instanceof Object);
}

function isFunction(data) {
  return typeof data === 'function';
}
```

## Vue

new Vue => Init(\_init) => Mount(\$mount) => Render(\_render) => Update(\_update)

### Init

执行`_init()`

#### beforeCreate 前

1. 初始化生命周期: 初始化变量和变量赋值
2. 初始化事件中心: `vm._events`赋值，且更新 vm 中的事件，简单来说就是将父级监听的时间放入`vm._events`中，等待触发
3. 初始化渲染： 初始化`vm.$slots、vm.$scopeSlots、vm._c、vm.$createElement`等方法，并赋值`$attrs、$listeners`为响应式数据
4. 执行 beforeCreate 钩子

#### beforeCreate 后 created 前

1. 初始化 inject:
   - inject 寻找 provider 的属性值，如果属性未寻找到，则继续向父级寻找，直到根 Vue，如果再根 Vue 未找到，则发出警告
   - inject 初始化时,`toggleObserving(false)` 关闭依赖收集，但使用`defineReactive(vm, key, result[key])`构造响应式数据
   - inject 虽然响应数据，但在开发环境下，提供自定义 setter，在修改值时会发出警告，生产环境不会
2. 初始化状态

   - 初始化 props:
     1. props 规范化，将所有的 props 转换为标准，例如 `{name: { type: String }}`
     2. 根基子组件定义的规则，对父组件的传值校验
     3. props 使用 `defineReactive(props, key, value)` 绑定为响应式数据，且自定义 setter，在开发环境赋值时，发出警告
     4. 代理到 vm
   - 初始化 methods
     1. function 校验
     2. 根据 props keys 去重校验
     3. 代理到 vm
   - 初始化 data
     1. data 赋值，如果是函数则运行，否则赋值（**由于对象是直接赋值，所以在组件内必须以函数形式存在，防止引用赋值的问题**）
     2. 根据 props、methods keys 去重校验
     3. 构造响应式数据
     4. 代理到 vm
   - 初始化 computed
     1. 根据 vm.\$data、vm.\$options.props keys 去重
     2. 构造 watcher，和其他区别，该 watcher 的 **lazy** 传值，所以 computed 是懒计算，只有再获取值才计算，且如果依赖无变化也不会计算，直接返回值
   - 初始化 watch: 根据参数构造 watcher

3. 初始化 provider
4. 执行 created 钩子

### Mount

调用\$mount 方法，且\$mount 随着平台的不同，有不同的定义

分析带 compiler 的\$mount，地址为*src/platform/web/entry-runtime-with-compiler.js*

1. 重写`$mount`
2. 获取挂载节点
3. 判断不能为 body，不能为 document
4. 获取模板，进入编译模板阶段
5. 执行原`$mount`，起始就是执行`mountComponent()函数`

mountComponent 方法存在于路径*src/core/instance/lifecycle.js*

```
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}

new Watcher(vm, updateComponent, noop, {
  before() {
    if (vm._isMounted && !vm._isDestroyed) {
      callHook(vm, 'beforeUpdate')
    }
  }
}, true)
```

1. 执行 beforeMount 钩子
2. 创建 `updateComponent()`
3. 新建 Watcher，将 `updateComponent`作为更新获取值函数传入，定义 beforeUpdate 钩子在 Watcher before 函数中。**对于一个 Vue 实例来说， 整个页面用的同一个 Watcher，派发更新时，会遍历该 Vue 的整个 VNode**
4. 执行 mounted 钩子

#### Compile

`compileToFunctions()`: 该函数则是编译函数，将模板编译为`render, staticRenderFns`函数，而 render 函数，则是在执行 `render()` 的重中之重

#### Render

`_render()`，定义于*src/core/instance/render.js*

1. 初始化 \$scopeSlots
2. 执行`vm.$createElement()`生成 VNode

createElement 定义于 _src/core/vdom/create-element.js_

1. 对 children 规范化，将子节点也变为 VNode 节点（functional 组件序列化后还是函数，**非 VNode**）
2. 创建 VNode，此时需要区分是否为组件，如果为组件，则执行`createComponent()` 否则执行`new VNode()`

**所以整个 render 过程是一个深度遍历的过程，先创建子项 VNode，再将子项的 VNode 塞入父级中，组合成大的 VNode**

### Update

`_update()`，定义于*src/core/instance/lifecycle.js*

将 VNode(虚拟 DOM)转换为真正的 DOM，执行`vm.__patch__()`，再完成更新后，执行 updated 钩子（ps: 和 beforeUpdate 一样都有 `vm._isMounted` 限制）

### 响应式数据

#### get

在触发 getter 函数时，将依赖（Watcher）收集到 Deps 中

#### set

再出发 setter 函数时，Deps 派发更新，触发 Watcher 的 update 方法

ps: 如果数据为懒获取`lazy===true`，则在 update 时，使用`dirty=true` 标记，表示如果需要获取数据，需要重新计算，否则直接返回现有值，计算完后`dirty=false`;

否则，普通更新为，将 Watcher 加入刷新队列，一起执行`flushSchedulerQueue()`执行队列刷新，所以 Vue 更新也不是实时更新，从而导致渲染也不是实时渲染

#### proxy

```
function proxy(target, ob) {
  for (const key of Object.keys(ob)) {
    Object.defineProperty(target, key, {
      get() {
        return ob[key];
      },
      set(v) {
        ob[key] = v;
      },
    });
  }
}
```

### 组件化

组件最终在 createElement 中，执行的是`createComponent()`

createComponent 定义于*src/core/vdom/create-component.js*

1. 使用`Vue.extend`生成子 CVue
2. 异步组件解决`resolveAsyncComponent()`
3. 生成 VNode，**此时 CVue 并未实例化**
4. 在 update 时，调用 `createElm()`构建正真的 DOM，此时才实例化 CVue，且**先创建子节点 DOM，在加入父节点，也是一个深度遍历过程**

#### 全局组件

```
Vue.component(); 可以声明全局组件
```

全局组件放在 Vue.options.components 中，在实例化或扩展到子集的`vm.$options.components`中，所以子集可以任意使用

#### 局部组件

存放于`vm.$options.components`，在寻找组件时，根据 name 可以寻找到该组件

#### 异步组件

1. 借助 webpack 对于`import()`会自动打包为新的 Chunk，并提供`__webpack_require__.e`来加载异步组件
2. vue 中，使用`resolveAsyncComponent()`加载异步组件，定义于*src/core/vdom/helpers/resolve-async-component.js*

从 Promise 来说把，因为 vue-router 常用的时 Promise 组件

```
Vue.component('ASYNC_COM',()=>import(xx)); // 定义异步组件，tips 路由的异步组件由 vue-router 提供解决方案
```

1. 调用`__webpack_require__.e()` 加载该异步组件模块，加载完毕后返回 Promise
2. 调用`__webpack_require()` 加载该异步模块，获取该模块
3. 下载安装完成后，在 resolveAsyncComponent 内通过`then(resolve,reject)`调用模块，并调动`forceRender() => $forceUpdate()`

### nextTick

nextTick 在下一次 DOM 更新结束后执行回调

1. 将 nextTick 回调函数添加进 nextTick callback 队列
2. 执行 callback 队列
   - 如果存在 Promise，则使用 `Promise.resolve().then(run callbacks)`
   - 如果存在 MutationObserver，则使用 MutationObserver
   ```
    let c = 0;
    const ot = document.createTextNode(c.toString());
    const ob = new MutationObserver((m, obs) => {
      // run callbacks
    });
    ob.observe(ot, { characterDataOldValue: true });
    setTimeout(() => { ot.data = (++c).toString(); }, 500);
   ```
   - 如果存在 setImmediate，则`setImmediate(run callbacks)`
   - 以上都不存在时，则使用`setTimeout(run callbacks)`

### 插槽

slot，scopeSlot 在 slot 上，增加了向父组件传值

#### slot

1. 在使用`compileToFunctions()` 编译时，会编译为 `_vm._t("default")`，如果具名插槽，则编译为`_vm._t("name")`
2. `_t === renderSlot`，定义于 _packages/vue-server-renderer/build.dev.js_
3. 调用`renderSlot()`，根据名称获取`this.$slots`内部的值，
4. **如果 props 出现 slot 传值**，则 作为`$createElement('template',props, children)` ，创建并返回 VNode
5. 否则直接返回`this.$slots`存储的 VNode

#### scopeSlot

1. 在使用`compileToFunctions()` 编译时，会编译为 `_t("default",null,{ key: _vm.key })`，如果具名插槽，则编译为`_t("name",null,{})`
2. 其余和 slot 保持一致

**新的 API,`v-slot` 只支持 template 使用**

区别：

1. slots 无法传值；scopeSlots 可以传值
2. slots 在 initRender 初始化并赋值，在 beforeCreated 前；而 scopeSlots 虽然这 beforeCreated 初始化，但在 \_render() 阶段赋值

### keep-alive

keep-alive 是 Vue 提供的内置组件，可以对其内部组件缓存，定义于*src/core/components/keep-alive.js*

keep-alive 是抽象组件，组件属性 `abstract === true`,组件父子关系将忽略该组件

1. keep-alive 在 created 钩子中，创建`this.cache = Object.create(null)、this,keys = [];` cache 用作以 key 值存储缓存，keys 用于计算缓存个数、LRU 算法队列
2. keep-alive 自定义 render 方法
   - 获取 slots 节点，获取 slots 传值中第一个**组件节点**，作为当前渲染组件。ps 例如*div*这样的节点时无效的
   - 获取当前组件节点 VNode
   - 计算当前组件 key 值
   - 根据 key 去 cache 寻找，如果**缓存命中**: 则使用该缓存，并将该 key 值移到 keys 末尾；如果**缓存未命中**: 根据 key 值推入 cache，将 key 值放入 keys 末尾*此时，如果 keys 长度超过 max，则执行清除缓存*
   - 清除缓存，删除 keys 头部保存的 key 值，且删除 key 对应的缓存

LRU: 以 keys 来保存组件的 key 值，活跃组件放置在 keys 末尾，（如果从非活跃组件变为活跃组件，也需要移动到末尾，清除算法从头部开始清除

#### 渲染

首次渲染和普通渲染一致

**缓存渲染**

1. 从`Component`组件来说，当 is 值出现变化时
2. 子 VNode 发生变化，此时需要执行 patch 算法，计算新旧 VNode 的区别
3. patch 执行前，先调用 `prepatch()` => `updateChildComponent()`updateChildComponent 定义于 _src/core/instance/lifecycle.js_

```
// resolve slots + force update if has children
if (needsForceUpdate) {
  vm.$slots = resolveSlots(renderChildren, parentVnode.context);
  vm.$forceUpdate()
}
```

4. 解析完 slots 后，执行`$forceUpdate() => 执行kee-alive render()`
5. 缓存计算，如果命中缓存，则在 `createComponent()` 方法出现分支，createComponent 定义于 _rc/core/vdom/patch.js_

```
const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
...
if (isTrue(isReactivated)) {
  reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm)
}
```

6. reactivateComponent 将 子 DOM 节点塞入父 DOM 节点，完成更新

**简单来说，当 keep-alive 更新子组件时，计算缓存，如果缓存命中，直接将存储的 DOM 节点塞入父 DOM 节点**

**为何不会触发生命周期钩子**

```
const componentVNodeHooks = {
  init (vnode: VNodeWithData, hydrating: boolean): ?boolean {
    if ( vnode.componentInstance && !vnode.componentInstance._isDestroyed && vnode.data.keepAlive) {
      // kept-alive components, treat as a patch
      const mountedNode: any = vnode // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode)
    } else {
      const child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance
      )
      child.$mount(hydrating ? vnode.elm : undefined, hydrating)
    }
  },
  ...
}
```

当使用 keep-alive 时，不会触发组件`_init() 以及 $mount()`，则不会触发 create 和 mount 钩子

### vue-router

#### Vue.use

1. 使用 Vue 插件方式在 beforeCreate 和 destroyed 钩子中加入 vue-router 操作
2. 其中 beforeCreate
   - 调用 vue-router 初始化方法 `init()`
   - 将**根 Vue 实例的 route 绑定为响应式对象**，tips: 为了实现组件之间、渲染
   - 将非根 VUe 实例，赋值 `vm.__routerRoot`

#### new VueRouter

实例化 VueRouter

1. 生成**Matcher**实例，并生成 Matcher 实例同时，会序列化 _routes_，生成三个对象，用于路由匹配

   - pathList: 数组，用于存放所有 routes 路径
   - pathMap: 对象，用于存放 path 和 路由匹配结果的映射(`path => RouterRecord`)
   - nameMap: 对象，用于存储 name 路由匹配结果的映射(`path => RouterRecord`)

**tips: 序列化还有个重要的过程，将 path 路径转换为正则，用于后续匹配校验**

2. 根据所选 mode，生成不同的 History 对象，例如 HashHistory，HTML5History

#### new Vue

1. `new Vue() => beforeCreate钩子`，执行使用 Vue.use(VueRouter)额外增加的 beforeCreate 钩子
2. VueRouter 初始化
   - this.apps 赋值，将 Vue 实例存储
   - 根据不同的 mode，调用不同的首次跳转

##### HashHistory 首次跳转

1. 声明成功回调、失败回调
2. 调用`transitionTo()`，该方法是 VueRouter 跳转方法，包括后续 `push、replace` 都是调用该方法
   - 路由匹配
   - 调用 `confirmTransition()` 该方法是 transitionTo 内部正真实现跳转的方法
   - transitionTo 成功后执行回调
3. `setupHashListener() => history.setupListeners()` 定义于*src/history/hash.js*
   - 如果支持 History API 则监听 _popstate_，否则监听 _hashchange_ ,当检测到 hash 值变化时，再次执行 transitionTo 函数

##### HTML5History 首次跳转

1. 在 HTML5History 实例化，即刻监听浏览器 popstate 事件，如果触发则执行 transitionTo 函数
2. 首次跳转时，执行 History Base 的 自定义 transitionTo 方法，其实和 transitionTo 函数差不多

#### path 匹配

路径匹配主要有 Matcher 对象和 match 方法完成

1. 在实例化时，即完成 routes 的序列化
2. 执行 match 方法
   - 将目标路径格式化，因为存在很多种 path 路径，例如`/test、{path: '/test'}、{ name:'Test'}`，返回 Location 对象
   - 如果存在 name，则执行 name 匹配，使用 name 在 nameMap 中查询，如果查询到，则返回由该 RouteRecord 构造的 Route，否则进入 path 匹配
   - 遍历 pathList，使用 Location.path 和 pathMap 中保存的 RouteRecord 正则进行匹配，如果匹配，则返回由该 RouteRecord 构造的 Route
   - 如果都没有匹配到，则返回由 null 构造的 Route

#### 路由跳转

1. 根据 path 匹配算出 newRoute，和 oldRoute 比较，如果相同，则中断
2. 根据 newRoute，oldRoute 计算出 **update: 需要更新的组件；activate: 需要重新安装激活的组件；deactivate: 需要删除的组件**
3. 创建执行任务队列

   - 任务 1: 卸载 deactivate 内组件，触发 beforeRouterLeave
   - 任务 2: 执行全局 beforeEach 钩子
   - 任务 3: 更新 update 内组件，触发 beforeRouterUpdate
   - 任务 4: 执行路由内部钩子 beforeEnter
   - 任务 5: 激活 activate 内组件，`resolveAsyncComponents()`

4. resolveAsyncComponents

   - 创建组件数量计数
   - 通过`__webpack_require__.e、__webpack_require__`加载该组件，例如常用的异步组件，加载完成后执行 then 方法
   - 将返回结果值缓存`match.components[key] = resolvedDef`
   - 计数减 1，如果计数为 0，则执行下一步骤

5. 任务执行完毕后，触发任务回调

   - 创建新的任务队列
   - **收集 beforeRouterEnter next 回调**，合并**激活新组件任务**与 beforeResolve 钩子回调，且*激活新组件任务在前*
   - 执行新的队列，并在执行激活新组件时，执行 beforeRouterEnter 钩子
   - 执行 beforeResolve 钩子
   - 完成后回调，在 nextTick 中执行 beforeRouterEnter 回调，定义于**src/history/base.js**的`extractEnterGuards() => bindEnterGuard()`，由于是在渲染后进行执行回调，所以可以访问 vm

6. confirmTransition 执行成功 => 触发成功回调 => 执行 afterEach 钩子

7. 执行 transitionTo 回调

#### 重新渲染

```
1. 绑定根Vue的(vm._route === vm.$route)为响应式数据: Vue.util.defineReactive(this, '_route', this._router.history.current);
2. history 监听 cb 回调，对根Vue
history.listen((route)=>{
  this.apps.forEach(app=>{
    app._route = route
  }
});
);
3. 在confirmTransition 成功回调中执行 cb

this && this.cb(route)

- 由于vm._route为响应式数据，且在 router-view => render 函数中，访问了该数据，则vm._route更新时，会对此派发更新
- 当 route-view 更新时，重新执行 render 方法，获取子节点，执行更新算法
```

#### push、replace 等 API

push、replace API 相差不大，由不同 History 对象调用自身实现的方法

1. 都是使用 transitionTo 实现跳转，成功后执行回调
2. 回调时修改 URL，此时 HTML5History 由 History API 实现（_push API 做了个错误处理_）;而 HashHistory 做了个兼容，支持 History API 才使用，否则使用 window 自带的 API

如下为伪代码

```
// push
if(supportHistory){
  pushState(path);
}else{
  window.location.hash = path;
}

// replace
if(supportHistory){
  pushState(url, true)
}else{
  window.location.replace(path);
}
```

#### router-link

router-link 为 vue-router 提供组件，在点击时执行 push 方法

1. 对 history mode 和 hash mode 完全兼容，这点 a 标签无法保证
2. 如果设置 base path，则可以没必要将路径写全，它会自动读取

#### router-view

router-view 为 vue-router 提供组件，且 router-view 是函数式组件

1. 在 beforeCreate 钩子中，将根 Vue 的 route 绑定为响应式数据 `Vue.util.defineReactive(this, '_route', this._router.history.current)`
2. 当路由跳转时，会修改该对象，由此派发更新

## HTTP

### HTTPS 握手

HTTPS 为 HTTP + SSL + TCP，在 HTTP 和 TCP 中间，使用 SSL/TLS 对数据加密

1. 客户端 => 服务端: 客户端发送 1. 所选 SSL 协议版本；2. 支持的加密协议；3. 随机生成的秘钥 Key1；4. 支持的压缩算法
2. 服务端 => 客户端: 服务端返回 1. 确定 SSL 协议版本；2. 所选加密方式；3. 证书；4.随机生成的秘钥 Key2；5. 所选压缩算法
3. 客户端: 客户端对证书检查 1. 检查颁发机构；2. 检查证书是否合法；3.检查证书是否过期；4.检查证书域名是否与所访问域名相同...
4. 客户端 => 服务端: 如果检查通过，则发送: 1. 使用证书的公钥加密的随机秘钥 Key3；2. 公钥加密的前面握手数据，让服务端检查；3. 告诉服务端，后面通信都使用（Key1+Key2+Key3 计算的**会话秘钥加密**）
5. 服务单: 1. 使用私钥解密 Key3；2. 校验服务单传输的握手数据；3. 计算会话秘钥

_自此开始，后续所有加密，均采用会话秘钥_

### HTTPS session 恢复

由于未知原因，可能断开 HTTPS，如果想再次恢复通话的，**再次握手成本显然过高**，可以采用如下方式恢复

**session-id 恢复**

由 Session-ID 记录会话，客户端再再次发起连接时，可以带上该 Session-ID，如果服务端对此有记录的话，则直接创建对话，无序再次握手

**session-ticket**

由于 Session-ID 只能保存在一台主机上，如果与多台主机通信，则无法使用 Session-ID，可以使用 Session-Ticket

Session-Ticket 由服务端生成、加密后传输给客户端，内部包含 Session-ID，会话秘钥等等信息，且**只能被服务端解密**，客户端再次发起连接时，带上该 Session-Ticket，服务端解密后即可使用当前秘钥创建会话，无序从新握手

### HTTP 301 302 303 307 及 SEO 影响

1. 301 为永久重定向

   - 搜索引擎会将老网址的排名赋予新的网址
   - 搜索引擎会使用新地址替换老地址

2. 302 临时重定向

   - 不会替换老地址
   - 新的访问也不会计入老地址访问量，**由于网站访问量是排名关键因素**，所以 302 会对 SEO 造成影响

3. 303 临时重定向: 规定重定向后，必须以 GET 方式获取资源**虽然 302 规定不允许改变请求方式，但大部分浏览器都是以 GET 方式获取的**
4. 307 临时重定向: 由于 302 历史原因，所以采用 307 来规定不允许改变方法

## NPM

### NPM 包下载机制

执行 npm install 步骤

1. 序列化参数，例如合并 .npmrc
2. 构建当前 node_modules 依赖树
3. 构建构建目标依赖树（根据 shrinkwrap.json、package-lock.json、package.json），**次数需要从远程仓库获取依赖包信息**
4. 比新树、老树，找出不同的依赖
5. 模块扁平化，如果遇到相同模块时，如果版本符合，则忽略，否则放入自身模块依赖下**原则是尽量将模块放入第一层，便于重用**
6. 根据依赖查询缓存，如果缓存匹配则直接使用缓存，否则向远程仓库获取
7. 检验包的完整性: 通过，放入缓存中，按照结构解压到项目 node_modules 中；不通过：重新下载
8. 生成 package-lock.json

### NPM 缓存机制

1. 缓存目录为 `npm config get cache`，Unix 为`~/.npm/_cacache`
2. index-v5 存储缓存索引
3. content-v2 存储缓存内容
4. npm 缓存无法自动清除

### npm 缓存查找方式

1. 以 _package.lock.json_ 中 resolved 和 integrity 属性为准，"pacote:range-manifest:{$resolved}:{$integrity}"以 SHA256 取 Hash 值
2. 在 ~/.npm/\_cacache/index-v5 下按照 hash.slice(0,2) / hash.slice(2,4) / hash.slice(4) 目录下查找该包的数据
3. 在当前文件下找到包元数据中的 `metadata.manifest._shasum` 属性
4. 在 ~/.npm/\_cacache/content-v2/sha1、sha512 目录下查找文件，请注意 sha1 目录还是 sha512 目录，根据 integrity _-_ 前指定
5. 根据 \_shasum.slice(0,2) / \_shasum.slice(2,4) / \_shasum.slice(4) 查找到该包的缓存
