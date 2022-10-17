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


## 监控系统

### 埋点

1. 手动埋点
2. 系统埋点
3. 全埋点

### 信息捕获

错误信息，性能信息

1. try...catch
2. window.onerror
3. window.onunhandledrejection
4. 各个框架的错误捕获
5. 白屏捕获
6. 内存泄漏监控，window.performance.memory
7. 性能指标
8. PV、UV、用户停留时间

## css 自定义属性

## ts

### 如何判断 readonly

readonly 比较难判断，只有在泛型尚未赋值时，extends 后的值必须一致，如下所示

```typescript
type IsEqual<T1, T2> = (<T>() => T extends T1 ? 1 : 0) extends <
  T
>() => T extends T2 ? 1 : 0
  ? 1
  : 0;
```

### never 作用

1. 类型计算，例如 `string & number`
2. Exhaustive Check，例如 if...else，switch...case，提示小伙伴加新的类型时，进行处理

```typescript

```

## react

```js
createRoot
  1. 调用 createContainer -> createFiberRoot 生成一个 rootFiber
  2. 返回一个 ReactDOMRoot，并 ReactDOMRoot._internalRoot = rootFiber

ReactDOMRoot.render
  1. 将 render 传入的ReactElement，updateContainer 到该 rootFiber，updateContainer(ReactElement, fiber)
```

### Reconciler(render)

以 while 循环，通过 workInProgress 指针，指向当前所处理的 Fiber，来实现可中断的 DFS

```js
// workLoopConcurrent
function workLoopSync() {
  while (workInProgress !== null) {
    // 执行某些操作
    workInProgress = beginWork();

    if (workInProgress === null) {
      completeWork();
    }
  }
}
```

#### beginWork

根据当前传入 Fiber，生成子 Fiber

update：是否可以复用 fiber

- 复用成功：bailoutOnAlreadyFinishedWork -> (判断子树是否需要更新 ? cloneChildFibers : null)
- 如果复用失败，则进入创建 Fiber 逻辑，并进入 reconcileChildren（创建 Fiber 并 Diff），并给需要操作的 Fiber 赋予 effectTag

mount：进入创建 Fiber 逻辑，根据传入的 Fiber tag，执行不同的函数，如 FunctionComponent 在执行 renderWithHooks 时，进入 reconcileChildren(创建 Fiber)，

#### completeWork

当 DFS 子 Fiber 结束后，执行当前 Fiber 的 completeWork，也是根据不同的 Fiber tag 调用不同的函数，拿 HostComponent 来说

update：updateHostComponent
mount：创建 DOM --> appendAllChildren(并将子元素挂载于该 DOM 上) --> finalizeInitialChildren

updateHostComponent 和 finalizeInitialChildren 主要是处理 DOM 的一些事件，style 等等

### Renderer(commit)

#### before mutation 阶段

在 before mutation 主函数之前 调度 useEffect
执行 before mutation 主函数，其中包含 getSnapshotBeforeUpdate 的生命周期

#### mutation 阶段

根据 effectTag 处理真实的 DOM 操作

1. HostComponent

   - Placement(插入): 操作 DOM API
   - Update(插入)：
   - Deletion（删除）：删除节点

2. FunctionComponent

   - Placement(插入): 操作 DOM API
   - Update(插入)：同步执行 useLayoutEffect 销毁函数
   - Deletion（删除）：调度 useEffect 的销毁函数

#### layout 阶段

commit layout 主函数

调用生命周期函数

对于 FunctionComponent：调用 useLayoutEffect 回调函数，调度 useEffect 的销毁与回调函数
对于类组件，可根据 current 来调用 componentDidMount 和 componentDidUpdate

赋值 ref

### diff

用于计算出哪些 Fiber 可以复用，用于节省性能。由于深度对比两个树的话，所需要的时间较为巨大，以如下约束

1. 同级别对比，不同层级不可复用
2. 以 key 来辅助提高效率，key 不同，则不可复用
3. 以 type 描述节点，不同 type 不可复用

#### 单节点

对于单节点，直接进行比较，

1. current 是否存在，不存在直接创建新的 Fiber
2. 判断 key、type 是否相同，相同则可以复用，返回当前 Fiber 的副本，否则不可复用，创建新的 Fiber

#### 多节点

两次遍历

第一次遍历节点

遍历 nextChildren 和 oldFiber

1. 如果 child 和 oldFiber key 不一致，则结束遍历
2. 如果 child 和 oldFiber key 一致，但 type 不一致，则标记 oldFiber 删除
3. 否则持续遍历，直到 nextChildren 或 oldFiber 结束

第二次 lastIndex，当结束第一次遍历时，可能存在如下集中情况

1. nextChildren 和 oldFiber 结束，直接更新
2. nextChildren 结束，删除 oldFiber 后续节点
3. oldFiber 结束，需要新增节点
4. 都没结束，采用 lastIndex 算法

lastPlacedIndex lastPlacedIndex = 0

1. 将 oldFiber 遍历，生成 map = { [key] : Fiber }
2. 遍历剩余的 nextChildren 第 i 个时，key 为 j，去 map 内查找 j 的是否存在

   - 存在，可以被复用，判断此时 i >= lastPlacedIndex ? "无需移动该节点": "需要向右移动节点";lastPlacedIndex = Math.max(lastPlacedIndex, i);
   - 不存在，不可被复用，需要新增节点

缺点，如果出现换位的话，更新效率较低，例如

```text
abcd
dabc
```

### 更新

Update 触发分为三类，ReactDOM、ClassComponent，FunctionComponent，每个 Update 都有几个关键字段 lane、payload、next

触发更新 -> 创建 Update -> 从 Fiber 到 RootFiber -> 调度更新 -> render 时由 processUpdateQueue 完成 Update -> commit

创建好的 Update 存储于 updateQueue 中，包含如下关键字段

- baseState：执行时的初始状态
- firstBaseUpdate、lastBaseUpdate：因优先级不够，无法执行的 Update
- shared：本次更新的 Update

例如此次有三个任务 A1 -> B2 -> C1 -> D2，显示为 ABCD

```js
// 连续插入四个任务 A1 B2 C1 D2
// 先存储于shared.pending中 D2 -> C1 -> B2 -> A1
baseState = '';
firstBaseUpdate = null;
lastBaseUpdate = null

// render 时，将pending环剪开，拼接到 lastBaseUpdate 后，当 render 时，由于优先级问题，会先执行 A1 -> C1
baseState = '';
//  A1 -> B2 -> C1 -> D2
firstBaseUpdate = A1;
lastBaseUpdate = D2

展示为 AC

// 接下来，触发低优先级任务
baseState = 'A'; // 由于B2被跳过了，此时baseState为'A'
firstBaseUpdate=B2 // 由于B2被跳过了，为了依赖完成，必须再次执行B2
lastBaseUpdate = D2
```
