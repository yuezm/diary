# weak 10.16~10.23

## react

what: 构建「快速响应」的javascript库

how:

何为快速响应，制约快速响应的限制有哪些

1. CPU算力不足
   - 设备老旧：无法解决
   - JS执行时间过长，阻塞GUI线程渲染：将同步的大任务，切分为异步的小任务，分次执行，称为「时间分片」

2. 网络I/O：本质上来说，网络请求的阻塞无法避免，但是可以通过交互来减少用户的感知

React 做的改进

React 原有架构约为如下：Reconciler -> Renderer

Reconciler：协调器

1. 在状态发生变更时，调用render（函数组件），生成VDOM
2. VDOM diff
3. 根据diff，通知Renderer

Renderer：渲染器，利用 react-dom

1. 根据变化，渲染

React 现有架构如下：Schecduler -> Reconciler -> Renderer

Schecduler：任务调度器，判断浏览器当前是否有空闲时间
Reconciler：协调器，根据 Schecduler来判断当前是否执行，在执行时，对需要更新的VDOM打上标记
Renderer：渲染器

优化点

1. 递归 -> 循环，任务可被中断
2. 增加 Schecduler，可在浏览器空闲时调用
3. Schecduler 和Reconciler随时可被高优任务打断，但是Renderer仍然不可打断
