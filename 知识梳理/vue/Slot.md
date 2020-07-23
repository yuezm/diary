# 插槽

## 什么是插槽

插槽是什么？按照 vue 的官方文档

> Vue 实现了一套内容分发的 API，这套 API 的设计灵感源自 Web Components 规范草案，将 <slot> 元素作为承载分发内容的出口...

插槽理解为是一个”占位符“，就像小区的停车位，你把车牌挂载停车位上，表示这个车位被你占了，等你回家后，你可以把车停进去，不管是什么牌子的汽车都可以停进去。而插槽也是一样，使用 slot 占位，表示这里存在一些数据，但是置于数据是什么，他预先也不知道

举个栗子

```vue
<template>
  <test-slot>
    <p>hello slot</p>
  </test-slot>
</template>
```

```vue
<template>
  <slot />
</template>
```

由上 demo 可以比较明显的看出，插槽分为两部分组成 **HTML 模板**和 **slot 组件**

HTML 模板是啥？HTML 模板就是 VNode，VNode 是啥，VNode 就是虚拟 DOM 的虚拟节点，（类比于 DOM 的 Node）
