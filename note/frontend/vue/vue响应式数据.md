# Vue 响应式数据

## 原理

用一句话来概括: 使用`Object.defineProperty` 劫持对象属性的*getter* 和*setter*。在 getter 触发时，收集依赖，在 setter 触发时派发更新

有几个非常重要的对象

1. Observer: 可观察对象
2. Deps: 收集依赖对象（订阅发布模式的消息中心）
3. Watcher: 观察者

## 实现

主要逻辑

```javascript
function defineReactive(target, key, value) {
  ...
  Object.defineProperty(target, key, {
    get(): any {
      // 收集依赖
    },
    set(v: any): void {
      // 派发更新
    },
  });
}
```

[simple-vue observe](https://github.com/yuezm/simple-vue/tree/master/src/observe)

### props

`vm.$options.propsData`: 存储父组件传递给子组件的 propsData，例如 {test: "Hello word"};  
`vm.$options.props`: 存放了子组件 props 定义及校验方法，在序列化后，例如 { test:{ type: String, required:false, default: "HELLO China" } }

1. 按照 props 定义进行校验，校验通过后取值
2. 关闭响应式数据（_为啥需要关闭_），并手动调用 `defineReactive(vm._props, key, value)` 绑定响应式数据，绑定完成后开启响应式数据
3. 代理到 vm

**为什么需要关闭响应数据**

```
toggleObserving(false) => shouldObserve = false => 会关闭 observe，致使无法使用 observe 创建响应式数据
```

在调用 `defineReactive(vm._props, key, value)` 时，会继续调用`observe(val)`，此时，shouldObserve 可以阻止属性值继续创建响应式对象，因为 props 是父级传递的数值，不需要在子级再次维护完整的响应式数据，从而节省性能

### data

1. 调用 `observe(data)`，需要注意的是，如果 data 是一个函数，则调用`observe(data.call(vm,vm))`
2. 代理到 vm

### watch

1. 调用 `vm.$watch` 监听，本质也是创建一个 Watcher，该 watcher 和 render watcher 不同，属于用户定义 watcher

### computed

computed 也是使用 Watcher 类实现的，只是在初始化时传入了参数`lazy: true`

1. Watcher 实例化时，在 lazy 下，不会执行*获取属性值*的操作
2. computed 在获取值时，根据 dirty 属性判断
   - 如果 dirty===true: 则重新计算值，并缓存，且将 dirty 置为 false
   - 如果 dirty===false: 则取缓存值
3. 在 Watcher 更新时，将 dirty 置为 true，即表明需要重新获取值

由以上三步，完成 computed 的惰性计算
