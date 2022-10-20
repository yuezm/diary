# day

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

what: 构建「快速响应」的 javascript 库

how:

何为快速响应，制约快速响应的限制有哪些

1. CPU 算力不足

   - 设备老旧：无法解决
   - JS 执行时间过长，阻塞 GUI 线程渲染：将同步的大任务，切分为异步的小任务，分次执行，称为「时间分片」

2. 网络 I/O：本质上来说，网络请求的阻塞无法避免，但是可以通过交互来减少用户的感知

### React 做的改进

React 原有架构约为如下：Reconciler -> Renderer

Reconciler：协调器

1. 在状态发生变更时，调用 render() => VDOM
2. VDOM diff
3. 根据 diff，通知 Renderer

Renderer：渲染器，利用 react-dom，总体流程

jsx -> ReactElement(VNode) -> Reconciler -> Renderer

1. 根据变化，渲染

React 现有架构如下：Schecduler -> Reconciler -> Renderer

Schecduler：任务调度器，判断浏览器当前是否有空闲时间
Reconciler（render 阶段）：协调器，根据 Schecduler 来判断当前是否执行，在执行时，render() => Fiber，并对需要更新的 Fiber 打上标记
Renderer（commit 阶段）：渲染器

jsx -> ReactElement -> Fiber -> ( Schecduler、Reconciler、Renderer)

优化点

1. 递归 -> 循环，将同步的不大可打断的任务 -> 异步的可被打断的任务
2. 增加 Schecduler，可在浏览器空闲时调用
3. Schecduler 和 Reconciler（render 阶段）随时可被高优任务打断，但是 Renderer（commit 阶段）不可打断

### React 从 createRoot 到组件更新的流程

前置知识，后续全部以函数组件、HTML 标签为例，class 组件不分析

```js
函数组件会被编译为 ReactElement，在React17前以createElement运行，在React17后，编译器自动从jsx报引入，无需在手动引入
```

1. ReactDOM.createRoot 创建 FiberRootNode 和 rootFiber

   ```js
   FiberRootNode.current = rootFiber;
   rootFiber.stateNode = FiberRootNode;
   ```

2. 调用 render

   ```js
   render(<App />)  --> updateContainer
   ```

3. updateContainer

   ```js
   创建 Update 对象，并调度更新
   ```

4. 进入 render 根据 `performSyncWorkOnRoot` 或 `performConcurrentWorkOnRoot`

   ```js
   // 根据所选模式，调用，差别就是会检查浏览器忙线，shouldYield()
   // 简单来说，就是将递归改为 while循环，且可以随时根据shouldYield()中断，但单个Fiber内是无法中断的
   [performSyncWorkOnRoot | performConcurrentWorkOnRoot] --> [workLoopSync,workLoopConcurrent]

    // workLoopConcurrent
    function workLoopSync() {
      while (workInProgress !== null) {
        // 执行某些操作
        workInProgress = beginWork(); // beginWork 返回子Fiber

        if (workInProgress === null) {
          completeWork();
        }
      }
    }
   ```

5. render - beginWork

   根据当前传入 Fiber，生成子 Fiber

   update：是否可以复用 fiber

   - 复用成功：bailoutOnAlreadyFinishedWork -> (判断子树是否需要更新 ? cloneChildFibers : null)
   - 如果复用失败，则进入创建 Fiber 逻辑，并进入 reconcileChildren（创建 Fiber 并 Diff），并给需要操作的 Fiber 赋予 effectTag

   mount：进入创建 Fiber 逻辑，根据传入的 Fiber tag，执行不同的函数，如 FunctionComponent 在执行 renderWithHooks 时，进入 reconcileChildren(创建 Fiber)，

6. render - completeWork

   当 DFS 子 Fiber 结束后，执行当前 Fiber 的 completeWork，也是根据不同的 Fiber tag 调用不同的函数，拿 HostComponent 来说

   update：updateHostComponent
   mount：创建 DOM --> appendAllChildren(并将子元素挂载于该 DOM 上) --> finalizeInitialChildren

   updateHostComponent 和 finalizeInitialChildren 主要是处理 DOM 的一些事件，style 等等

7. commit - before mutation(commitBeforeMutationEffects)

   在 before mutation 主函数之前 调度 useEffect
   执行 before mutation 主函数，其中包含 getSnapshotBeforeUpdate 的生命周期

8. commit - mutation(commitMutationEffects)
   根据 effectTag 处理真实的 DOM 操作

   1. HostComponent

      - Placement(插入): 操作 DOM API
      - Update(插入)：
      - Deletion（删除）：删除节点

   2. FunctionComponent

      - Placement(插入): 操作 DOM API
      - Update(插入)：同步执行 useLayoutEffect 销毁函数
      - Deletion（删除）：调度 useEffect 的销毁函数

   切换 fiberRootNode.current 到 work

9. commit - layout(commitLayoutEffects)

   1. 对于 FunctionComponent：调用 useLayoutEffect 回调函数（commitHookLayoutEffects），调度 useEffect 的销毁与回调函数(flushPassiveEffectsImpl)
   2. ref 处理，对 ref 赋予 DOM 元素的值、ClassComponent 实例、FunctionComponent 的值

10. 如 setState 触发更新，通过 dispatchAction 来触发更新

    ```js
    dispatchAction = dispatch.bind(null, fiber, hooks.queue);

    dispatchAction --> 创建Update对象 --> hooks.queue携带Update对象 --> 递归 get rootFiber --> 调度更新
    ```

### Scheduler

#### 可实现调度的 API

1. setTimeout: 在浏览器端存在 4ms 最小延迟
2. RAF：RAF 调用时间间隔不确定
3. requestIdleCallback：兼容性问题；如果浏览器在空闲时，requestIdleCallback(callback) 回调函数的执行间隔是 50ms（W3C 规定）
4. MessageChannel：较为适合
5. Promise、MutationObserver：微任务递归调用，会导致卡在清空微任务阶段，导致无法进入浏览器的 layout

由于浏览器每一帧需要进行如下操作，执行宏任务 --> 清空微任务 --> 执行 RAF --> layout、paint --> requestIdleCallback

#### react 如何实现调度，关键方法 schedulerCallback 和 shouldYield

React 和 Scheduler 交互简介

1. Scheduler 提供 schedulerCallback，可以插入一个任务（例如一次更新）
2. Scheduler 调度该任务（port1.postMessage，在 port2.onmessage 时 从任务队列中取出任务并执行，执行前需要判断是否可以执行 task && !shouldYield）
3. Reconciler 基于 Fiber 执行更新，在执行前询问 Scheduler 提供的 shouldYield ，是否需要暂停
4. 如果不需要暂停则继续更新；否则暂停更新，并且 Scheduler 在下一个合适的时间（创建一个新的宏任务）继续调度

具体步骤

调用 schedulerCallback(lane,cb)

1. 根据任务优先级计算出 timeout，此时间仅用于计算任务优先级，优先级越高，timeout 越小
2. 创建一个任务
3. 判断当前任务是否为延迟任务，如果是延迟任务，则推送 timerQueue；否则推入 taskQueue，并根据 startTime + time 递增排序（插入排序），以保证高优任务现行执行
4. port.postMessage，创建一个宏任务
5. port1.onmessage = cb
6. 执行 cb --> flushWork --> workLoop
7. 会遍历 timerQueue，将需要执行的任务移动到 taskQueue，并保证排序
8. 以 while 形式从 taskQueue 取出任务，判断任务是否 shouldYieldToHost

   - 需要 shouldYieldToHost，则判断当前任务是否结束，如果未结束，则再次 port.postMessage 创建一个宏任务，等待下一次调度（performWorkUntilDeadline 的 hasMoreWork）
   - 不需要的话进入下一步

9. 执行任务，在执行任务时，可能会因为 shouldYield 而中断，此时再次 port.postMessage 创建一个宏任务，等待下一次调度（performWorkUntilDeadline 的 hasMoreWork）

```js
shouldYieldToHost 判断 currentTime - startTime < 预设值（5ms），如果满足则继续执行；不满足则中断，即每帧React只要求有5ms的执行时间
```

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

```js
Fiber {
  updateQueue: UpdateQueue
}

UpdateQueue {
  baseState: 本次更新前fiber的状态
  firstBaseUpdate，lastBaseUpdate: 因优先级不够，无法执行的 Update
  shared: { pending:null }：本次更新的 Update

}

Update {
  lane: 优先级
  tag: 更新类型
  payload: 挂载的数据
  callback: 更新后的回调函数
  lastEffect: Effect，在函数组件中，还存在，单链表
  next
}

Effect {
  tag: HookFlags,
  create: () => (() => void) | void,
  destroy: (() => void) | void,
  deps: Array<mixed> | void | null,
  next: Effect,
}
```

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

### Hooks

#### Fiber React 实现的代数效应

Fiber 是 React 的一套更新机制，支持不同优先级，中断及恢复，且恢复可以记住之前的状态

代数效应是函数式编程的一个概念，即将副作用从函数中转出去，举个例子

```jsx
function App() {
  const user = useGetUser(); // 我完全不知道useGetUser内部是同步还是异步，也不知道内部的实现，既可以完全当做同步函数来使用，即将副作用抽离出去
}
```

**为什么不使用 async...await 和 Generator**

1. 传播性，使用了 async 或 Generator，后续函数必须都是 async 或 Generator
2. 在中途插入高优任务时，复杂度太高了

从某种程度上来说 Fiber 和 VNode 相似，例如 Fiber 和 VNode 属性都存储了节点的信息，例如 type,tag,key 等等，也都会对应某个真实的 DOM 节点，但是 Fiber 和 VNode

1. Fiber 描述的是某个任务，所以 Fiber 包含内当前的节点的操作，任务的优先级等等；而 VNode 描述的某个节点
2. Fiber 以链表形式存储子元素；而 VNode 靠数组存储子元素

hooks 存储在 fiber 中，由 memoizedState 存储的单链表

#### useState，useReducer

在 mount 的 render 阶段时，在调用 useState 时，会创建一个 Hooks 对象，以单链表形式记录在 Fiber.memoizedState 上，顺序和 hooks 使用的顺序一致，并返回 [hooks.memoizedState, dispatch]

此时如果 dispatch 触发，则创建一个 Update 对象，以循环单链表形式记录于 Hooks.queue.pending 上，开启调度

在 update 的阶段时，在调用 useState 时，根据 Fiber.memoizedState 找到对应的 Hooks 节点，执行 Hooks.queue.pending 上的所有更新

此时有个注意点，如果在 render 阶段可再次触发更新，react 以一个 didScheduleRenderPhaseUpdate 来标记

```js
Hooks {
  memoizedState: 当前hooks的值，根据Hooks不同，保存不同的值，例如useState保存的state的值
  baseState: 当前hooks的值
  baseQueue: 上一次更新，由于优先级的关系未执行的Update
  queue: Queue
  next: 指针
}

Queue {
  pending: Update的循环链表
  lanes: 优先级
  dispatch: 等于useState的dispatch，调用约为 dispatch.bind(null, fiber, queue);
  lastRenderedReducer: 上一次render时使用的reducer
  lastRenderedState: 上一次render时的state
}

Update {
  lane: Lane, // 优先级
  action: A, // 传递的参数
  hasEagerState: boolean,
  eagerState: S | null,
  next: Update<S, A>,
}
```

#### useEffect、useLayoutEffect

在 mount-render 时，执行 useEffect，创建一个 hooks 对象，hooks 对象保存了回调函数、销毁函数、依赖等等，存储于 Fiber.memoizedState，并通过 pushEffect 向 fiber.updateQueue.lastEffect，以单链表环存储

flushPassiveEffects -> flushPassiveEffectsImpl

在 mount - commit 时，调度执行 fiber.updateQueue.lastEffect 的函数，先全部调用 unmount，在全部调用 mount

在 update 时，执行 useEffect 时，判断当前依赖是否改变，如果改变了则继续执行 pushEffect，否则不执行

#### useRef

在 mount-render 时，执行 useRef，创建一个 hooks 对象，hooks.memoizedState 对象保存了 { current: null }
在 mount-render 时，对包含 ref 的 fiber，赋予 Ref effectTag 标识
在 mount-commit 时，如果出现了 DOM 删除，则需要清空 ref，否则对 ref 进行赋值

#### useMemo、useCallback

在 mount-render 时，执行 useMemo，创建一个 hooks 对象，hooks.memoizedState 对象保存了回调函数和依赖
在 update-render 时，执行 useMemo，对比依赖，如果没变化则返回原值，否则返回新值

### memo

1. memo 返回一个 对象，包含 $$type 标记为 memo ReactElement，和 compare 方法
2. 在 beginWork 时，进入 MemoComponent 分支比较，如果满足一定条件例如 compare 为 null 等等，进入 SimpleMemoComponent
3. MemoComponent -> 对比 props；SimpleMemoComponent --> 无需对比 props，但需要比较 context 是否更新

### lazy

1. import() 借助 webpack 实现代码分割
2. lazy：返回一个对象，$$type 标记 lazy ReactElement，及加载组件的方法（lazyInitializer）

   ```js
   lazyInitializer;

   // 判断当前组件的加载状态，如果未加载过，咋执行 ctor() 去加载，当加载完毕后，修改加载状态，并保存加载结果；如果已加载了，则直接返回结果
   // 如果最终状态还是为"未加载"，则抛出错误，否则返回加载结果
   ```

3. 在 beginWork 时，进入 LazyComponent 分支，调用 mountLazyComponent，获取加载完成的结果，并根据组件类型在生成 Fiber
4. Suspense，类似于一种错误捕获机制了，在发生错误时，展示 fallback

## redux

1. 中心化的结构，以一个大对象存储所有的数据，无法拆解领域
2. 支持异步的话，需要另外的手段，例如 dva（dispatch 满天飞）

## recoil

1. 初始化

   在 atom 调用后，生成一个 node 对象，且由 get，set 等方法，并通过 registerNode 向全局 nodes 注册（nodes.set(node.key, node)）

2. 如何存储数据

   RecoilRoot 内以 Context<AppContext.Provider value={storeRef}> 实现

   storeRef.getState() --> storeStateRef

   storeStateRef 是一种特殊的数据结构（StoreState），其包含 prevTree、currentTree、nextTree，类型都为 TreeState，拿 currentTree 来说，存在树形 atomValues，这个就是存储当前 recoil 状态的地方

3. 如何修改数据 `useSetRecoilState(recoilState)`

   通过 Context API 获取到 storeRef（这也就是 recoil 只能访问最新的 RecoilRoot 的原因），根据 storeRef，可以获取到 storeStateRef
   期间存在一些列的调用链，最终调用到关键函数 store.replaceState，有如下功能

   - 拷贝老的值 --> 新值
   - 对新值执行更新 --> 以 recoilState 获取到 key，从而根据 key 向 nodes 获取到 node，调用 node.set 尝试校验合法性（值重复或者想要重置值，但是无需要重置的值），写入 storeStateRef 中（需要注意的时，值并非一次性写入 storeState，而类似是 commit->push，node.set 先 commit，最终在 push）
   - 返回新值
   - 通知 Batcher 组件更新（通过 setState({})，迫使组件更新）

4. 如何获取值并响应更新 `useRecoilValue(recoilState)`

   通过 Context API 获取到 storeRef（这也就是 recoil 只能访问最新的 RecoilRoot 的原因），根据 storeRef，可以获取到 storeStateRef
   以 recoilState 获取到 key，从而根据 key 向 nodes 获取到 node，调用 node.get(store, state)

5. 如何更新 `useRecoilValue(recoilState)`
   在调用 useRecoilValue hooks 时， subscribeToRecoilValue 想当前的 state.nodeToComponentSubscriptions 注册事件，并设置回调函数
   当设置数据时，会致使 Batcher 更新，在 Batcher 更新时，调用 endBatch --> sendEndOfBatchNotifications
   sendEndOfBatchNotifications 中派发 storeState.nodeTransactionSubscriptions 的更新
   次吃由于在调用 useRecoilValue(recoilState) 时已经向 state.nodeToComponentSubscriptions 注册了事件，此时收到更新后，执行回调函数
   回调函数中，再次判断是否需要更新（newValue === oldValue?），如果值不一致，则执行更新（通过 setState([])）
