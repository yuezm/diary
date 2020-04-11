# Vue 响应式数据

## 原理

用一句话来概括: 使用`Object.defineProperty` 劫持对象属性的*getter* 和*setter*。在 getter 触发时，收集依赖，在 setter 触发时派发更新

有几个非常重要的对象

1. Observer: 可观察对象
2. Deps: 收集依赖对象（订阅发布模式的消息中心）
3. Watcher: 观察者

## 实现

**主要实现方法（省略版）**

```typescript
function defineReactive(target: object, key: string, value: any): void {
  const _deps: Deps = new Deps();
  ...
  Object.defineProperty(target, key, {
    get(): any {
      // 由于作用域关系，此处无法访问 *哪一个 Watcher 正在获取属性*，所以使用静态属性来标识
      if (Deps.Target) {
        // 收集依赖
        _deps.append(Deps.Target);
        if (!isEmpty(childOb)) {
          // 该步骤是结合 **数组方法劫持** 来使用的，如果属性值为数组，则调用更新时 ob.deps 而非 闭包中的 _deps
          (childOb as Observer).deps.append(Deps.Target);
        }
      }
      ...
    },
    set(v: any): void {
      // 派发更新
      ...
      _deps.notify();
    },
  });
}
```

**Deps: 存储管理依赖（省略版）**

```typescript
class Deps {
  ...
  public static Target: Watcher | null;
  private subs: Set<Watcher>;

  constructor(){
    this.subs = new Set<Watcher>();
  }

  append(w: Watcher): void{
    ...
    this.subs.add(w);
    ...
  }

  notify(): void{
    for(const w of this.subs){
      w.update();
    }
  }
}
```

**Watcher: 依赖（省略版）**

```typescript
class Watcher(){
  ...
  private value: any;
  private vm: Vue;
  private getter: Function;
  ...

  constructor(options: IWatcherOptions){
    this.vm = options.vm;
    this.getter = options.getter;

    this.get();
  }

  get(){
    Deps.Target = this;
    this.value = this.getter();
    Deps.Target = null;
  }

  update(): void{
    // 更新干点啥
  }
}
```

### 依赖收集

1. 当 watcher 调用 get 时，触发 getter 方法
2. watcher 内的 getter 方法，触发对象属性的 getter 方法
3. 对象属性的 getter 方法，触发 deps 的 append 方法，此时依赖收集成功

`watcher.get() => watcher.getter() => property getter() => deps.append()`

### 派发更新

1. 当属性值改变时，触发属性值的 setter 方法
2. 属性值 setter 方法，触发 deps 的 notify 方法
3. deps 的 notify 方法，触发内部收集的 watcher 的 update 方法

`property value changed => property setter() => deps.notify() => watcher.update()`

[simple-vue observe](https://github.com/yuezm/simple-vue/tree/master/src/observe)

## Vue 中响应式数据简介

### props

`vm.$options.propsData`: 存储父组件传递给子组件的 propsData，例如 {test: "Hello word"};  
`vm.$options.props`: 存放了子组件 props 定义及校验方法，在序列化后，例如 { test:{ type: String, required:false, default: "HELLO China" } }

1. 按照 props 定义进行校验，校验通过后取值
2. 关闭响应式数据（_为啥需要关闭_），并手动调用 `defineReactive(vm._props, key, value)` 绑定响应式数据，绑定完成后开启响应式数据
3. 代理到 vm

在 props 绑定响应式数据时，会执行如下代码

```
toggleObserving(false) => shouldObserve = false => 会关闭 observe，致使无法使用 observe 创建响应式数据
```

**为什么需要关闭响应数据？？？**

在调用 `defineReactive(vm._props, key, value)` 时，会对值继续调用`observe(val)`，此时，shouldObserve 可以阻止属性值继续创建响应式对象，因为 props 是父级传递的数值，不需要在子级再次维护完整的响应式数据，从而节省性能

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
