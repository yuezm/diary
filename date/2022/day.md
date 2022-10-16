# weak 10.16~10.23

## react

what: 构建「快速响应」的javascript库

how:

何为快速响应，制约快速响应的限制有哪些

1. CPU算力不足
   - 设备老旧：无法解决
   - JS执行时间过长，阻塞GUI线程渲染：将同步的大任务，切分为异步的小任务，分次执行，称为「时间分片」

2. 网络I/O：本质上来说，网络请求的阻塞无法避免，但是可以通过交互来减少用户的感知

### React 做的改进

React 原有架构约为如下：Reconciler -> Renderer

Reconciler：协调器

1. 在状态发生变更时，调用render() => VDOM
2. VDOM diff
3. 根据diff，通知Renderer

Renderer：渲染器，利用 react-dom，总体流程

jsx -> ReactElement(VNode) -> Reconciler -> Renderer

1. 根据变化，渲染

React 现有架构如下：Schecduler -> Reconciler -> Renderer

Schecduler：任务调度器，判断浏览器当前是否有空闲时间
Reconciler（render阶段）：协调器，根据 Schecduler来判断当前是否执行，在执行时，render() => Fiber，并对需要更新的Fiber打上标记
Renderer（commit阶段）：渲染器

jsx -> ReactElement -> Fiber -> ( Schecduler、Reconciler、Renderer)

优化点

1. 递归 -> 循环，将同步的不大可打断的任务 -> 异步的可被打断的任务
2. 增加 Schecduler，可在浏览器空闲时调用
3. Schecduler 和Reconciler（render阶段）随时可被高优任务打断，但是Renderer（commit阶段）不可打断

### Fiber React实现的代数效应

Fiber 是 React的一套更新机制，支持不同优先级，中断及恢复，且恢复可以记住之前的状态

代数效应是函数式编程的一个概念，即将副作用从函数中转出去，举个例子

```jsx
function App(){
  const user = useGetUser(); // 我完全不知道useGetUser内部是同步还是异步，也不知道内部的实现，既可以完全当做同步函数来使用，即将副作用抽离出去
}
```


**为什么不使用async...await和Generator**

1. 传播性，使用了async或Generator，后续函数必须都是async或Generator
2. 在中途插入高优任务时，复杂度太高了



从某种程度上来说Fiber和VNode相似，例如Fiber和VNode属性都存储了节点的信息，例如type,tag,key等等，也都会对应某个真实的DOM节点，但是Fiber和VNode

1. Fiber描述的是某个任务，所以Fiber包含内当前的节点的操作，任务的优先级等等；而VNode描述的某个节点
2. Fiber以链表形式存储子元素；而VNode靠数组存储子元素


从3个阶段来分析

before
```
jsx -> render function

render function -> ReactElement
```

mount
```
```

update
