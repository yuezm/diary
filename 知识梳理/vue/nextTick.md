# Vue 异步更新和 nextTick

什么是异步更新？在 vue 状态发生改变的时候，在更新 DOM 的时候是异步的，举个例子

```vue
<template>
  <p ref="asyncP">
    {{ i }}
  </p>
</template>

<script>
export default {
  data() {
    return {
      i: 0,
    };
  },
  mounted() {
    this.i = 1;

    console.log(this.$refs.asyncP.innerHTML); // 0

    this.$nextTick(() => {
      console.log(this.$refs.asyncP.innerHTML); // 1
    });
  },
};
</script>
```

如上代码所示，当 vue 状态发生变更，i 由 0 变为 1，但是 DOM 未立即更新

> 可能你还没有注意到，Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的

## 为什么需要异步更新

如 vue 文档所写，vue 采用异步更新主要是为了 1. 去除重复数据；2. 避免多次操作 DOM

举个如下例子

```vue
<template>
  <p>
    {{ i }}
    {{ j }}
  </p>
</template>

<script>
export default {
  data() {
    return {
      i: 0,
    };
  },
  mounted() {
    for (let x = 1; x <= 10; x++) {
      this.i = x;
    }
    this.j = 10;
  },
};
</script>
```

1. i 发生 10 次改变，如果不为异步更新，即 i 状态改变后直接触发 DOM 更新，则需要进行 10 次 DOM 更新，采用异步则 1 次更新就够了
2. i、j 同时发生改变，如果不为异步更新，则 i 状态改变需要一次 DOM 更新，j 状态改变则也需要一次 DOM 更新，采用异步则 1 次更新就够了

## Vue 如何实现异步更新

首先说下状态改变后的各种操作

1. 属性 setter 触发 Watcher (Watcher.update)
2. Watcher.update 将当前 Watcher 插入更新队列，并做去重处理，此时可以避免同一个 Watcher 被多次插入队列，避免不必要的更新
3. nextTick 中 flush 队列
4. nextTick flush 队列致使 Watcher 开始 flush 队列
5. Watcher.run
6. 触发生命周期钩子
7. 执行完毕

## nextTick

1. nextTick 插入回调函数如队列
2. 异步 flush 队列
   - Promise
   - MutationObservable
   - setImmediate
   - setTimeout
