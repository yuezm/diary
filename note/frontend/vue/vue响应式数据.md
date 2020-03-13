# Vue 响应式数据

## 原理

用一句话来概括: 使用`Object.defineProperty` 劫持对象属性的*getter* 和*setter*。在 getter 触发时，收集依赖，在 setter 触发时派发更新

有几个非常重要的对象

1. Observer: 可观察对象
2. Deps: 收集依赖对象（观察者模式的消息中心）
3. Watcher: 观察者

## 实现

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

[simple-vue reactive]()

## computed

computed 也是使用 Watcher 类实现的，只是在初始化时传入了参数`lazy: true`

1. Watcher 实例化时，在 lazy 下，不会执行*获取属性值*的操作
2. computed 在获取值时，根据 dirty 属性判断
   - 如果 dirty===true: 则重新计算值，并缓存，且将 dirty 置为 false
   - 如果 dirty===false: 则取缓存值
3. 在 Watcher 更新时，将 dirty 置为 true，即表明需要重新获取值

由以上三步，完成 computed 的惰性计算
