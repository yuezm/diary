# Vue Slot

## 什么是插槽

根据 Vue 官方文档，插槽介绍如下所示

> Vue 实现了一套内容分发的 API，这套 API 的设计灵感源自 Web Components 规范草案，将 \<slot\> 元素作为承载分发内容的出口

插槽就像主板的内存插口，给你预留了一个位置放入内存条，你可以选择不插入内存条，也可以插入 2G 内存、4G 内存...。回到插槽来说，子组件给你预留了位置放入 VNode，父组件可以传入空，也可以传入 span、div、p 的 VNode

根据我们平时书写的代码，我们将插槽分为两个部分

- _父组件内的 HTML 模板_，也就是父组件传入的 VNode
- _子组件内的 slot 组件_，也就是子组件预留插槽的位置

## 插槽分类

按照官方文档，插槽有如下分类

**按照是否存在名称**

- 具名插槽
- 默认插槽（感觉“匿名插槽”和“具名插槽”比较对应）

**按照是否可以访问子组件数据**

- 普通插槽
- 作用域插槽

接下来我会慢慢分析这个几个插槽，先在这里给出结论，在此之前，明确一点，_组件的产出是 VNode_，基本步骤如下

HTML 模板 ==<sup>编译</sup>==> render 函数 ==<sup>运行</sup>==> VNode

**结论 1** _具名插槽、默认插槽没有区别_，默认插槽就是名称为“default”的具名插槽  
**结论 2** _普通插槽和作用域插槽本质也没有区别_，都是 VNode，只是产生 VNode 的地方不同
**结论 3** 作用域插槽依靠**函数传参**来访问子组件的数据

### 具名插槽、匿名插槽

拿如下代码来举例

```vue
// Parent.Vue

<Child>
  <p slot="default">slots</p>
</Child>
```

```vue
// Child.Vue

<div>
  <slot></slot>
</div>
```

#### 父组件

**step1 编译为 render 函数**

父组件的 HTML 模板编译后的 render 函数如下所示

```javascript
// render 函数
function anonymous() {
  with (this) {
    return _c('Child', [
      _c(
        'p',
        {
          attrs: { slot: 'default' },
          slot: 'default',
        },
        [_v('slots')]
      ),
    ]);
  }
}
```

有些同学可能看的不太习惯，我们转换下写法，改成 `createElement` 写法，但我们要明确一点，`_c` 和 `createElement` 只是相似，不是完全一致的，但是我们可以在此处使用 `createElement` 来转换为我们熟悉的样子，方便理解

```javascript
// 转换后的render函数
render(h) {
  return h('Child', [
      h('p', {
        attrs: { slot: 'default' },
        slot: 'default',
      }, 'slots'),
    ],
  );
}
```

提示

1. 如果不清楚如何查看编译后的 render 函数的话，可以使用如下函数`Vue.compile($str).render.toString();`来查看
2. 如果不太清楚 `_c、_v...`等函数具体是什么，可以看文章最后，有贴

**step2 运行 render 函数**

父组件的 render 函数运行后，得到 VNode，如下代码为简化版，只列举个比较重要的几个参数。

这里将整个父组件模板编译后的 VNode 记为**VNodeParent**，将 Child 标签包裹的元素编译后的 VNode 记为 **VNodeSlots**，方便后续介绍

```javascript
const VNodeSlots = [
  {
    tag: 'p',
    data: { slot: 'default' },
    children: [
      {
        tag: undefined,
        text: 'hello word',
      },
    ],
  };
];

const VNodeParent = {
  tag: 'vue-component-1-Child',
  componentOptions: {
    // 对于Component来说，children存储于VNode.componentOptions.children，而非VNode.children，记住这个位置！！！
    // 具体见 “src/core/vdom/create-component.js” createComponent 函数
    tag: 'child',
    children: VNodeSlots,
  },
};
```

提示，如果不太清楚 render 函数运行时机和运行结果，可以查看 “src/core/instance/lifecycle.js” 文件的 mountComponent 函数

```javascript
// src/core/instance/lifecycle.js

export function mountComponent(vm: Component, el: ?Element, hydrating?: boolean): Component {
...
  updateComponent = () => {
    vm._update(vm._render(), hydrating); // 可以打印 _render函数的值，或debug
  }
}
```

这里值得一提的是，其实 `vm._render` 并不是我们上面编译后的 render 函数，真正 render 函数运行于 “src/core/instance/render.js” 文件内，如果你在此处打印 `vm._render.toString()` 会发现和我们上面列出的 render 函数不一致，但此处我们关心的是结果值 vnode，所以不用在意这些细节

```javascript
// src/core/instance/render.js

export function renderMixin (Vue: Class<Component>) {
  ...
  Vue.prototype._render = function (): VNode {
    const { render, _parentVnode } = vm.$options
    ...
    vnode = render.call(vm._renderProxy, vm.$createElement);
    // 这才是正真 render 函数运行的地方
    // vm._renderProxy 可以看做就是 vm
  }
}
```

至此，父组件分析完毕，进入子组件分析

#### 子组件

**step1 编译 render 函数**

子组件编译后的 render 函数为

```javascript
function anonymous() {
  with (this) {
    return _c('div', [_t('default')], 2);
  }
}
```

我们还是把它转换为我们熟悉的样子

```javascript
render(h) {
  return h('div', [ _t('default') ]); // 暂时忽略参数 2
}
```

这个函数内，我们看到了陌生的函数 `_t('default')`，由于 createElement API 参数性质和 "default" 字符串，我们进行如下猜测

**猜想 1** `_t('default')` 运行后返回一个 VNode  
**猜想 2** 通过传入 "default" 字符串可以获取父组件 HTML 模板中的 “slot="default” 的 VNode，也就是 VNodeSlots

至于真的是否是这样呢？那得分析下 \_t 了，也就是 renderSlot

```javascript
// src/core/instance/render-helpers/render-slot.js
// name slot名称，此处是 default
// 根据_t参数，其余参数均为空，则暂时无视这些参数
export function renderSlot(name: string, fallback: ?Array<VNode>, props: ?Object, bindObject: ?Object): ?Array<VNode> {
 let nodes
  // 此处排除  scopedSlotFn 干扰
  ...
  nodes = this.$slots[ name ] || fallback; // this.$slots 从哪里来的？
  // 此处排除 target干扰
  ...
  return nodes; // 直接返回了nodes，类型为 Array<VNode>，撇开 undefined 不谈哈
}
```

我们由上代码可以看到，\_t 确实是返回的是 VNode，证实了猜想 1，但是无法证实猜想 2。想要证实猜想，必须知道 `this.$slots` 的值，this 指向子组件的 Vue 实例，那 `$slots` 呢？

我们看向 “src/core/instance/render.js” 文件

```javascript
// src/core/instance/render.js

export function initRender(vm: Component) {
  const renderContext = parentVnode && parentVnode.context;
  vm.$slots = resolveSlots(options._renderChildren, renderContext);
}
```

很明显，\$slots 值为 resolveSlots 返回值，由于再分析 resolveSlots 就跑的太远了，我们只简单说下 resolveSlots 函数的参数和作用

```javascript
// src/core/instance/render-helpers/resolve-slots.js
export function resolveSlots(children: ?Array<VNode>, context: ?Component): { [key: string]: Array<VNode> } {
    const slots = {}; // 存储 slot 对象 { [slotName]: [ slotVNode ] }
    for (let i = 0, l = children.length; i < l; i++) {
        const child = children[ i ]
        const data = child.data
      if (
        ...
        data &&
        data.slot != null
      ) {
        // 如果存在 slot 名称，则推入 slots[slotName]，这就是具名插槽
        const name = data.slot;
        const slot = slots[name] || (slots[name] = []);
        ...
        slot.push(child);
      } else {
        // 如果不存在slot名称，则推入 slots.default，这就是默认插槽
        (slots.default || (slots.default = [])).push(child);
      }
    }
    return slots;
}
```

**children 参数** options.\_renderChildren， \_renderChildren 赋值于 “src/core/instance/init.js” 的 initInternalComponent 方法内

```javascript
export function initInternalComponent (vm: Component, options: InternalComponentOptions) {
  ...
  const vnodeComponentOptions = parentVnode.componentOptions;
  opts._renderChildren = vnodeComponentOptions.children;
  ...
}
```

我们在父组件的 HTML 说道过，Component 的 children 是存储于 VNode.componentOptions.
children 中的，所以 `options._renderChildren === parentVnode.componentOptions.children === VNodeSlots`，此处完成父组件到子组件的传值

**context 参数** renderContext，renderContext 为父组件的 Vue 实例

由 resolveSlots 函数我们可得如下

1. 证明**结论 1**
2. resolveSlots 将数组转换为一个对象，表现为 `[ slotVNode ] ==> { [slotName]: [ slotVNode ]}` 的对象

对于我们此处，\$slots 就是一个 `{ default: VNodeSlots }`，至此我们证实了猜想 2

**step2 运行 render 函数**

那么，子组件 render 函数执行完成后，得到 VNode 如下所示

```javascript
{
  tag: 'div',
  children: VNodeSlots,
  // 这里可能有同学会问了，[ _t('default') ] 得出应该是 [ VNodeSlots ]，为什么变成 VNodeSlots 了
  // 其实这里还有一个步骤，感兴趣可以查看 "src/core/vdom/helpers/normalize-children.js" 的 normalizeChildren 方法
}
```

至此父组件传入的 VNode 挂载进子组件内

### 作用域插槽

由于在普通插槽流程分析的比较完整，所以在作用域插槽，不会完整的分析流程，只会对照着普通插槽来分析不同之处

```vue
// Parent.Vue

<Child>
  <template slot-scope="scope">{{ scope.tt }}</template>
</Child>
```

```vue
// Child.Vue

<div>
  <slot tt="slots"></slot>
</div>
```

#### 父组件

**step1 编译 render 函数**

父组件的 HTML 模板编译后的 render 函数如下所示

```javascript
function anonymous() {
  with (this) {
    return _c('Child', {
      scopedSlots: _u([
        {
          key: 'default',
          fn: function (scope) {
            return [_v(_s(scope.tt))];
          },
        },
      ]),
    });
  }
}
```

还是将上面写成 createElement 写法

```javascript
render(h){
  return h('Child',{
    scopedSlots: {
      default: function (scope) {
          return [_v(_s(scope.tt))];
      }
    }
  })
}
```

我们看到作用域插槽和普通插槽似乎有点不同

1. **无 children 参数** 普通插槽 HTML 模板时放于 children 参数，而作用域插槽是放于 data 参数中的 scopedSlots 属性下
2. **传入数据方式** 普通插槽 children 参数是一个 VNode，而作用域插槽 data.scopedSlots 内部的 value 却是一个函数，函数还带了个参数，参数和我们 `slot-scope = "scope"` 的 "scope" 一致

提示，\_u 为 resolveScopedSlots，resolveScopedSlots 函数不过多介绍，函数大概的作用和 resolveSlots 相似，将数组转换为对象，表现为`[key: 'default',fn: HandleFunction}] => { default: HandleFunction }`

#### 子组件

**step1 编译 render 函数**

子组件内 render 函数如下所示

```javascript
function anonymous() {
  with (this) {
    return _c('div', [_t('default', null, { tt: 'slots' })], 2);
  }
}
```

我们还是将上面写成 createElement 写法

```javascript
render(h) {
  return h('div', [ _t('default', null, { tt: 'slots' }) ]); // 暂时忽略参数 2
}
```

我们看到作用域插槽和普通插槽似乎有点不同

1. **多出了参数** 作用域插槽 \_t 传了 3 个参数
2. **第三个参数** 第三个参数为我们在 slot 组件的属性，还记得我们是怎么样写的吗 `<slot tt="slots"></slot> ==> { tt: 'slots' }`

照例，我们继续分析 \_t，也就是 renderSlot

```javascript
// name slot名称，此处是 default
// fallback null
// props slot 组件传值，此时是 { tt: 'slots' }
function renderSlot(name, fallback, props, bindObject) {
  var scopedSlotFn = this.$scopedSlots[name];
  var nodes;


  props = props || {};
  nodes = scopedSlotFn(props) || fallback;

  // 排除 普通插槽、target 干扰
  ...
  return nodes;
}
```

到这一步，已经很明显了，我们只要确定了 this.\$scopedSlots 的值，就能确定 vnodes 的值

我们看向 “src/core/instance/render.js” 文件内的 \_render 方法

```javascript
// src/core/instance/render.js

Vue.prototype._render = function (): VNode {
  ...
  vm.$scopedSlots = normalizeScopedSlots(
    _parentVnode.data.scopedSlots,
    vm.$slots,
    vm.$scopedSlots
  )
 ...
}
```

我们不分析 normalizeScopedSlots，不然有点扯远了，有没有发现一个熟悉的身影 `_parentVnode.data.scopedSlots`，这不是我们在父组件放到 data 属性中的 scopedSlots 嘛，此处完成父组件到子组件的传值

此处，\$scopedSlots 为 `{ default: scopedSlotFn }`

**step2 运行 render 函数**

由上可知，在 renderSlot 函数内，scopedSlotFn 就是我们在父组件中的 fn 函数，那么运行它

```javascript
function scopedSlotFn(scope) {
  return [_v(_s(scope.name))];
}

scopedSlotFn({ tt: 'slots' });
```

现在发现 scope 参数作用了吗，就是函数参数，通过该参数来访问子组件内运行时的传值，由此证明**结论 3**

## 总结

### 对比普通插槽和作用域插槽

```javascript
// 普通插槽
_t('default');

// _t 运行于父组件，VNode 生成于父组件
// 通过 children 参数传输给子组件，子组件只用使用该VNode

// 作用域插槽
{
  scopedSlots: _u([
    {
      key: 'default',
      fn: function (scope) {
        return [_v(_s(scope.tt))];
      },
    },
  ]),
}

//_u运行于父组件，但 _u 运行后是一个函数，并不是一个 VNode，父组件通过 data.scopedSlots 参数传给子组件
// 函数是运行于子组件，VNode 也生成于子组件
```

那么把普通插槽转换为如下代码，那不就普通插槽和作用域插槽的传值方式不就一样了嘛

```javascript
const vn = _t('default');
{
  scopedSlots: {
    default: ()=> vn;
  }
}
```

由此证明**结论 2**

#### Vue2.6 合并插槽 和 v-slot

当然，Vue2.6 的代码不是这样的，但原理差不多

```javascript
// <template><p>slots</p></template>
// 在 Vue2.6 中，render 函数中的 scopedSlots 如下
{
  scopedSlots: {
    default: function(){
      return slots[ key ]; // slots === this.$slots
    }
  }
}
```

这就是 Vue2.6 合并普通插槽和作用域插槽后的 render 函数，普通插槽也是传递函数，且函数运行于子组件了。但对于 VNode 生成的地方依旧没变，作用插槽 VNode 是在子组件运行 scopedSlotFn 时生成；而普通插槽还是在父组件内就生成 VNode

至此 vue2.6 合并普通插槽和作用域插槽结束，至于 v-slot 只是 API 的变化，如果想了解 v-slot 的编译过程，请查看 “src/compiler/parser/index.js” 的 processSlotContent 函数

### 编译作用域

为什么 scopedSlotFn 运行于子组件但是编译作用域却是父组件？？

假设有如下代码

```vue
// Parent.vue

<Child>
  <template v-slot:default="scope">
    <p>{{ scope.tt }}</p>
    <p>{{ name }}</p>
  </template>
</Child>
```

Child.vue 如上不变，编译后的 render 函数如下所示

```javascript
// 父组件 render 函数
function anonymous() {
  with (this) {
    return _c('Child', {
      scopedSlots: _u([
        {
          key: 'default',
          fn: function (scope) {
            return [_c('p', [_v(_s(scope.tt))]), _c('p', [_v(_s(name))])];
          },
        },
      ]),
    });
  }
}
```

```javascript
// 子组件 render 函数
function anonymous() {
  with (this) {
    return _c('div', [_t('default', null, { tt: 'slots' })], 2);
  }
}
```

1. render 函数是以 `render.call(vm._renderProxy, vm.$createElement);` 运行，则确定 anonymous 函数内的 this 指向 vm.\_renderProxy，vm.\_renderProxy 可以认为就是 vm，则各个 anonymous 函数内 this 指向自身组件的 Vue 实例
2. 由于 `with(this){...}` 语句的原因，\_t 函数内部 this 指向 anonymous 的 this，也就是 vm；在 \_t 函数内部运行 scopedSlotFn 时，即运行如下函数

```javascript
function (scope) {
  return [
    _c('p', [_v(_s(scope.tt))]),
    _c('p', [_v(_s(name))]),
  ];
}
```

还记得作用域和闭包吗？拿 name 来说，scopedSlotFn 内部无法寻找到 name，则沿着作用域向上查找，直到 with 语句的 this 为止（this 存在 name 属性），由于 scopedSlotFn 定义于父组件的 render 函数中，那么 this 也就是父组件的 Vue 实例，则获取的自然也是父组件的数据，那么自然它的编译作用域就是父组件

## 备注

```javascript
// src/core/instance/render.js
export function initRender(vm: Component) {
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false);
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true);
}

// src/core/instance/render-helpers/index.js
export function installRenderHelpers(target) {
  target._o = markOnce;
  target._n = toNumber;
  target._s = toString;
  target._l = renderList;
  target._t = renderSlot;
  target._q = looseEqual;
  target._i = looseIndexOf;
  target._m = renderStatic;
  target._f = resolveFilter;
  target._k = checkKeyCodes;
  target._b = bindObjectProps;
  target._v = createTextVNode;
  target._e = createEmptyVNode;
  target._u = resolveScopedSlots;
  target._g = bindObjectListeners;
}
```
