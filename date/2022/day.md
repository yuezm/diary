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

```js
type IsEqual<T1, T2> = (<T>() => T extends T1 ? 1 : 0) extends (<T>() => T extends T2 ? 1 : 0) ? 1: 0;
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

React 现有架构如下：Scheduler -> Reconciler -> Renderer

Scheduler：任务调度器，判断浏览器当前是否有空闲时间
Reconciler（render 阶段）：协调器，找出变化的组件。根据 Scheduler 来判断当前是否执行，在执行时，render() => Fiber，并对需要更新的 Fiber 打上标记
Renderer（commit 阶段）：渲染器，将变化的组件渲染出来。

jsx -> ReactElement -> Fiber -> ( Scheduler、Reconciler、Renderer)

优化点

1. 递归 -> 循环，将同步的不大可打断的任务 -> 异步的可被打断的任务
2. 增加 Scheduler，可在浏览器空闲时调用
3. Scheduler 和 Reconciler（render 阶段）随时可被高优任务打断，但是 Renderer（commit 阶段）不可打断

Fiber 如何更新

### React 从 createRoot 到组件更新的流程

前置知识，后续全部以函数组件、HTML 标签为例，class 组件不分析

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

   根据当前传入 Fiber，创建或者复用 Fiber

   update：是否可以复用 fiber

   - 复用成功：bailoutOnAlreadyFinishedWork -> (判断子树是否需要更新 ? cloneChildFibers : null)
   - 如果复用失败，则进入创建 Fiber 逻辑，并进入 reconcileChildren（创建 Fiber 并 Diff），并给需要操作的 Fiber 赋予 effectTag

   mount：进入创建 Fiber 逻辑，根据传入的 Fiber tag，执行不同的函数，如 FunctionComponent 在执行 renderWithHooks 时，进入 reconcileChildren(创建 Fiber)，

   ```js
   updateFunctionComponent(){
    ...
    renderWithHooks()
    ...
    reconcileChildren();
   };
   ```

6. render - completeWork

   当 DFS 子 Fiber 结束后，执行当前 Fiber 的 completeWork，也是根据不同的 Fiber tag 调用不同的函数，拿 HostComponent 来说

   update：updateHostComponent
   mount：创建 DOM --> appendAllChildren(并将子元素挂载于该 DOM 上) --> finalizeInitialChildren

   updateHostComponent 和 finalizeInitialChildren 主要是处理 DOM 的一些事件，style 等等

7. commit - before mutation(commitBeforeMutationEffects)

   1. 处理 DOM 节点删除后的一些 auto focus，blur 逻辑
   2. 在 before mutation 主函数之前 调度 useEffect
   3. 执行 before mutation 主函数，其中包含 getSnapshotBeforeUpdate 的生命周期

8. commit - mutation(commitMutationEffects)

   1. 根据 effectTag 处理真实的 DOM 操作
   2. componentWillUnmount
   3. 调度 useEffect 销毁函数（删除操作时）
   4. useLayoutEffect 销毁函数

   5. HostComponent

      - Placement(插入): 操作 DOM API
      - Update(插入)：
      - Deletion（删除）：删除节点

   6. FunctionComponent

      - Placement(插入): 操作 DOM API
      - Update(插入)：同步执行 useLayoutEffect 销毁函数
      - Deletion（删除）：调度 useEffect 的销毁函数

   切换 fiberRootNode.current 到 work

9. commit - layout(commitLayoutEffects)

   1. 对于 FunctionComponent：调用 useLayoutEffect 回调函数（commitHookLayoutEffects），调度 useEffect 的销毁与回调函数(flushPassiveEffectsImpl)
   2. ref 处理，对 ref 赋予 DOM 元素的值、ClassComponent 实例、FunctionComponent 的值
   3. componentDidMount、componentDidUpdate

10. 如 setState 触发更新，通过 dispatchAction 来触发更新

    ```js
    dispatchAction = dispatch.bind(null, fiber, hooks.queue);

    dispatchAction --> 创建Update对象 --> hooks.queue携带Update对象 --> 递归 get rootFiber --> 调度更新
    ```

### Scheduler

#### 可实现调度的 API，为何选择 MessageChannel

由于浏览器每一帧需要进行如下操作，执行宏任务 --> 清空微任务 --> 执行 RAF --> layout、paint --> requestIdleCallback

1. setTimeout: 在浏览器端存在 4ms 最小延迟
2. RAF：RAF 调用时间间隔不确定
3. requestIdleCallback：兼容性问题；如果浏览器在空闲时；requestIdleCallback(callback) 回调函数的执行间隔是 50ms（W3C 规定）
4. MessageChannel：较为适合
5. Promise、MutationObserver：微任务递归调用，会导致卡在清空微任务阶段，导致无法进入浏览器的 layout

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
3. 判断当前任务是否为延迟任务，如果是延迟任务，则推送 timerQueue；否则推入 taskQueue，并根据 sortIndex = startTime + time 为依据，实现二叉堆，以保证高优任务现行执行
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
  firstBaseUpdate，lastBaseUpdate:  Update，因优先级不够，无法执行的 Update
  shared: { pending:null }：本次更新的 Update

}

Update {
  lane: 优先级
  tag: 更新类型
  payload: 挂载的数据
  callback: 更新后的回调函数
  lastEffect: Effect，在函数组件中，还存在，单链表
  next: Update
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
2. 在中途插入高优任务时，复杂度较高。例如当执行完毕 doExpensiveWorkA 和 doExpensiveWorkB 后，发现 B 组件存在一个高优更新，此时你必须从头再执行一遍 doWork()，而不能复用中途已经执行完毕的 doExpensiveWorkA

   ```js
   function* doWork(A, B, C) {
     var x = doExpensiveWorkA(A);
     yield;
     var y = x + doExpensiveWorkB(B);
     yield;
     var z = y + doExpensiveWorkC(C);
     return z;
   }
   ```

从某种程度上来说 Fiber 和 VNode 相似，例如 Fiber 和 VNode 属性都存储了节点的信息，例如 type,tag,key 等等，也都会对应某个真实的 DOM 节点，但是 Fiber 和 VNode

1. Fiber 描述的是某个任务，所以 Fiber 包含内当前的节点的操作，任务的优先级等等；而 VNode 描述的某个节点
2. Fiber 以链表形式存储子元素；而 VNode 靠数组存储子元素

hooks 存储在 fiber 中，由 memoizedState 存储的单链表

#### useState，useReducer

在 mount 的 render 阶段时，在调用 useState 时，会创建一个 Hooks 对象，以单链表形式记录在 Fiber.memoizedState 上，顺序和 hooks 使用的顺序一致，并返回 [hooks.memoizedState, dispatch]

此时如果 dispatch 触发，则创建一个 Update 对象，以循环单链表形式记录于 Hooks.queue.pending 上，开启调度

在 update 的阶段时，在调用 useState 时，根据 Fiber.memoizedState 找到对应的 Hooks 节点，将 Hooks.queue.pending 拼接到 baseQueue 上， 并执行所有更新

此时有个注意点，如果在 render 阶段可再次触发更新，react 以一个 didScheduleRenderPhaseUpdate 来标记

```js
Hooks {
  memoizedState: 当前hooks的值，根据Hooks不同，保存不同的值，例如useState保存的state的值
  baseState: 当前hooks的值
  baseQueue: Update, 上一次更新，由于优先级的关系未执行的Update
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

1. memo 返回一个 ReactElement，包含 $$type 标记为 memo ReactElement，和 compare 方法
2. 在 beginWork 时，进入 MemoComponent 分支比较，如果满足一定条件例如 compare 为 null 等等，进入 SimpleMemoComponent
3. MemoComponent -> 对比 props；SimpleMemoComponent --> 无需对比 props，但需要比较 context 是否更新

### lazy

1. import() 借助 webpack 实现代码分割
2. lazy：返回一个 ReactElement，$$type 标记 lazy ReactElement，及加载组件的方法（lazyInitializer）

   ```js
   lazyInitializer;

   // 判断当前组件的加载状态，如果未加载过，咋执行 ctor() 去加载，当加载完毕后，修改加载状态，并保存加载结果；如果已加载了，则直接返回结果
   // 如果最终状态还是为"未加载"，则抛出错误，否则返回加载结果
   ```

3. 在 beginWork 时，进入 LazyComponent 分支，调用 mountLazyComponent，获取加载完成的结果，并根据组件类型在生成 Fiber
4. Suspense，类似于一种错误捕获机制了，在发生错误时，展示 fallback

### Context

1. 创建 Context

   ```js
   const AppContext = createContext(); // 创建一个context对象，且将value数据存储于 _currentValue、_currentValue2 支持多个渲染核心，如ReactDOM 为主渲染核心
   <AppContext.Provider value={}><AppContext.Provider>
   ```

2. 在组件内调用

   - 进入 beginWork --> updateFunctionComponent --> reconcileChildren --> useContext
   - 创建一个 Dependencies 对象，包含当前 ctx 和当前 ctx 的值 memoizedValue
   - 向当前 Fiber 注册依赖，数据以链表存放于 Fiber.dependencies.firstContext
   - 根据 ctx 取出存储的值 \_currentValue、\_currentValue2 并返回

   ```js
   const ctx = useContext(AppContext); // 调用readContext
   ```

3. 何时更新，拿函数组件来说（beginWork）

   - 进入 beginWork
   - checkScheduledUpdateOrContext 校验，对比所有 Fiber.dependencies.firstContext 的新老值（memoizedValue 和 ctx.\_currentValue），如果存在一个不相等的，则进入 updateFunctionComponent
   - 在 updateFunctionComponent 内，还会根据 Fiber.dependencies.lanes, renderLanes，对比是否需要更新，如果需要更新，则会修改更新标识，并删除 Fiber.dependencies.firstContext
   - 如果修改了更新标识 didReceiveUpdate = true，则无法复用当前 Fiber，进入 reconcileChildren

4. 如何使用 Context.Provider，使用 valueStack （栈的存储结构）

    - 在 beginWork 时，将 _currentValue 推入栈中，并将当前的值赋予_currentValue
    - 在 completeWork 时，将 _currentValue 赋予从栈顶的值，并退栈

  以栈的数据结构，来保证

- 组件必须处于 context.Provider 内才可以获取到当前 Provider 的数据
- 如果存在多个Provider，则读取的是最近的那个Provider的数据

## recoil

1. 初始化

   在 atom() 调用后，生成一个 node 对象，且由 get，set 等方法，并通过 registerNode 向全局 nodes 注册（nodes.set(node.key, node)）

2. 如何存储数据

   ```js
   <RecoilRoot />
   ```

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

4. 如何获取值 `useRecoilValue(recoilState)`

   通过 Context API 获取到 storeRef（这也就是 recoil 只能访问最新的 RecoilRoot 的原因），根据 storeRef，可以获取到 storeStateRef
   以 recoilState 获取到 key，从而根据 key 向 nodes 获取到 node，调用 node.get(store, state)

5. 如何更新 `useRecoilValue(recoilState)`
   在调用 useRecoilValue hooks 时， subscribeToRecoilValue 想当前的 state.nodeToComponentSubscriptions 注册事件，并设置回调函数
   当设置数据时，会致使 Batcher 更新，在 Batcher 更新时，调用 endBatch --> sendEndOfBatchNotifications
   sendEndOfBatchNotifications 中派发 storeState.nodeTransactionSubscriptions 的更新
   次吃由于在调用 useRecoilValue(recoilState) 时已经向 state.nodeToComponentSubscriptions 注册了事件，此时收到更新后，执行回调函数
   回调函数中，再次判断是否需要更新（newValue === oldValue?），如果值不一致，则执行更新（通过 setState([])）

## redux和 react-redux

### redux

redux 是一个纯粹的状态管理库

1. 单一数据源，redux state 是一个大对象存储的
2. state 是只读的，无法被开发者私自修改
3. 可以通过 dispatch + reducer 进行修改 state

defect

1. 中心化的结构，以一个大对象存储所有的数据，无法拆解领域
2. 支持异步的话，需要另外的手段，例如 dva（dispatch 满天飞）

### react-redux

#### Subscription

react-redux 很重要的对象

1. 可收集依赖，以双向环形链表存储于 Listener 对象中
2. 可派发更新，对 Listener 搜集的依赖，派发更新
3. 可向 store(redux) 或者其他 Subscription 注册，当 store 或其他Subscription发生变化时，触发回调

#### Provider

1. 创建顶层的 Subscription 对象，监听 store 变化，当 store 变化时 notify
2. 创建 Context，将变量存入其中，方便后续访问，store，subscription

该 Context 可以自身传入，也可以使用默认的 ReactReduxContext

#### connect

1. 接收 mapStateToProps，mapDispatchToProps传参，并对参数处理，最终成为一个selector的形式，简单来说 (store.getState(), wrapperProps) => props;
2. 通过执行 useSyncExternalStore hooks，当值发生改变时，subscribeForReact 回调会被调用

    ```js
      actualChildProps = useSyncExternalStore(
        subscribeForReact,
        actualChildPropsSelector,
        getServerState
          ? () => childPropsSelector(getServerState(), wrapperProps)
          : actualChildPropsSelector
     )
    ```

3. 调用 subscribeForReact -> subscribeUpdates

    ```js
    // 1. 创建 Subscribition 对象，在组件中，如果父组件传入 store 对象，则向该 sotre 订阅变化；否则向 Context(ReactReduxContext)的 Subscription 订阅变化
    // 2. 当 store 更新时，直接通知当前 Subscription 或者由 Context 的根 Subscription 通知当前的 Subscription
    // 3. 当接收到变化时，需比较新旧props是否相同，如果不同则强中更新当前组件；如果相同则继续派发更新，让子组件判断是否需要更新
    ```

4. 将 actualChildProps 传给子组件

## React 16 -> 17 -> 18

17 的重要更新

1. React 合成事件的变化
2. 函数组件会被编译为 ReactElement，在 React17 前以 createElement 运行，在 React17 后，编译器自动从 jsx 报引入，无需在手动引入

18 的重要更

1. 合并更新，在 18 以前，只可以对 React 事件处理合并更新，在 18 之后，可以对 Promise，setTimeout 等等处理合并更新，可以由 flushSync 拒绝合并更新
2. render API，默认开启的 ConCurrent Mode
3. 组件返回可以为 undefined，18 前只能为 null
4. Strict Mode 的日志打印，18 后悔打印两次日志，一次为灰色
5. Suspense 的影响

   - React16 的时候，Suspense 是立即挂载于 DOM，并触发生命周期，但元素不可见（display: hidden）
   - React18 的时候，Suspense 遵从 resolve 后再渲染到 DOM 中，声明周期也是在目标懒加载组件解析后才触发

6. 新 API

   - flushSync
   - useId
   - startTransition 在大任务下，也可以保持响应，就是被 startTransition 回调包裹的 setState 触发的渲染被标记为不紧急渲染，这些渲染可能被其他紧急渲染所抢占

     ```js
     startTransition(() => {
       setState();
     });
     ```

   - useDeferredValue 返回一个延迟获取的值，只有在没有紧急更新时，返回值才会是新的值

     ```js
     const [value, setValue] = useState();
     const newValue = useDeferredValue(value);
     ```

## hydrate & island

hydrate 就是在 SSR 时，由服务端生成 HTML，但同时需要注入客户端所需要执行的脚本，此为注水
island 是一种性能优化的方式，将需要 hydrate 和静态组件拆分，对需要 hydrate 组件操作，对静态组件直接下发

## router

### 初始化

1. 创建路由，监听 history 变化,并执行首次跳转。需要注意的是已经没有 hashchange 监听了

   ```js
   const router = createBrowserRouter(); // createHashRouter
   ```

   - 创建 history，根据 browser 还是 hash 创建不同的 history（最终调用的也是同一个方法）
   - 创建 router，并立刻调用 initialize
     - 1. 调用 listen，监听 popstate 事件，并设置 callback startNavigation【从这有步骤开始，当 history 出现变化时，都会执行 startNavigation】
     - 2. 首次主动调用 startNavigation

2. RouterProvider 内注册 router 状态变化的 callback，简单来说就是在收到状态变化后，比对一下值，如果出现变化则 forceUpdate

3. 数据存储
   存储数据，使用 Context 存储数据，自定义 Hooks 读取

   ```js
   <RouterProvider router={router}>
   ```

### 响应变化

#### startNavigation

1. 路由匹配

   ```js
   1. 将路由拍平，并计算出路由的得分，得分大致规则如下
        - 根据 / 切割为数组，基础得分为数组的长度
        - 如果存在 * 匹配符，则减分
        - 如果存在 index 则加分
        - 最后再将数字每一项（除去 * ）的得分加起来（还是有规则 根据 :id、""、其他 得分不同）

    2. 排序
        - 如果路由得分不同，则降序
        - 否则，判断路由是否为相似路由（只有最后一位字符不同才为相似路由，例如 "/abc/d" vs "/abc/a"），如果为相似路由则升序；否则保持位置不变

    3. 匹配
        - 匹配第一个match的路由
   ```

2. NotFound 匹配

   当未匹配到路由时，会去寻找展示 notFoundMatches

3. 判断是否为 hash 路由

   - 如果是 hash 路由，则直接进入 completeNavigation
   - 否则会进去 action，loader 等等再进去 completeNavigation

#### completeNavigation

1. 路由状态更新，更新 router 的 state，更新时会广播状态的变更
2. 同步到 history

#### 组件渲染

1. RouterProvider 接收到广播，并强制更新
2. useRoutes 内 匹配一次路由, matches
3. renderedMatches
4. \_renderMatches

   \_renderMatches 简单来说，就是将匹配的 matches 从右侧连接起来

   ```jsx
   <RouteContext.Provider value={
     outlet, // reduce的第一个传参，表示从右侧开始到这加一起的组件
     matches: parentMatches.concat(renderedMatches.slice(0, index + 1)),
   }>
     {children}
   </RouteContext.Provider>
   ```

5. 处理 `<Outlet/>` 获取 OutletProps 传递的 Context，从 context 获取 outlet 属性（组件）并渲染

## 微前端

1. 优势
2. 目前普遍的实现
3. qiankun

### qiankun

和 single-spa 相比

1. HTML entry
   - JS entry 改造过大，而且一些常见的优化例如拆包，按需加载都无效了
   -
2. css 沙箱
3. js 沙箱
4. 预加载

#### 注册

调用 qiankun API registerMicroApps => single-sap 的 registerApplication

注册时，并声明了当前的 loadApp 方法

```js
function registerMicroApps() {
  registerApplication({
    name: App_Name,
    app: () => {
      // qiankun 的 loadApp 方法
    },
  });
}
```

#### 匹配 & 加载

```js
url change

reroute
  - getAppChanges()
  - performAppChanges()
      - unmountAllPromise
      - toLoadPromise -> loadApp
      - tryToBootstrapAndMount
        - bootstrap
        - unmountAllPromise 执行完毕
        - mount

```

single-spa 对事件劫持（hashchange、popstate），在 url 变化时，调用 getAppChanges() 对比所注册的 apps「activeRule -> activeWhen 与当前 URL 对比」、「当前 app 的状态」，来获取当前需要进行操作的子应用，例如

- 对于 load，如果 URL 匹配，则进入 load 流程（已 mount 的子应用会进入 unMount 流程），则进入 lappsToLoad 数组
- 对于 unMount，如果 URL 不匹配，且当前处于 MOUNTED 状态，则进入 appsToUnmount 数组

```js
const appsToLoad = [];
appsToLoad.push(app);

// appsToUnload: appsToUnload,
// appsToUnmount: appsToUnmount,
// appsToLoad: appsToLoad,
// appsToMount: appsToMount
```

single App 等待 app 加载完毕后，就会去执行各个阶段的生命周期 performAppChanges()

1. 以 promise 方式执行所有的卸载事件，unload,unmount，联合为一个 Promise，unmountAllPromise

2. 对于 appsToLoad ，是会调用 toLoadPromise(app);

   single spa 根据 appsToLoad 调用 loadApps 去加载，最终调用的是 app.loadApp 的方法

   single spa loadApps => app.loadApp => qiankun loadApp，会返回一个包含各个生命周期的对象，根据 single-spa 调用链的生命周期进行调用

   1. 下载匹配的子应用 entry，并序列化（import-html-entry 会将模板拆分）

      - template： 模板
      - script：外部的 script
      - style：外部的样式
      - execScripts：js entry

   2. 将 template 渲染至 DOM
   3. 创建沙箱（后续运行时会传入沙箱实例）
   4. 获取 js entry 的生命周期函数
   5. 返回自定义生命周期

3. 执行 bootstrap 生命周期

4. 等到 unmountAllPromise 完毕后，执行 mount 生命周期，完成渲染

   - beforeUnmount
   - 子应用生命周期 mount
   - 将当前沙箱置为 active
   - 区域渲染
   - afterUnmount

#### 卸载

执行卸载生命周期

- beforeUnmount
- 子应用的 unmount
- 恢复 global 状态，将当前的沙箱置为 inactive
- afterUnmount
- 清除 DOM 区域的 template

#### CSS 沙箱

1. scoped css
2. shadow DOM

#### JS 沙箱的作用及实现

##### SnapshotSandbox

mount 时将 window 浅拷贝，并原来的数据备份；当 unmount 时，恢复原来备份的数据

##### LegacySandbox

SnapshotSandbox 在恢复时，需要比对所有的 key，效率较低，LegacySandbox 使用 Proxy 代理，在 set 时，记录子应用改变的属性，记录哪些数据变了来减少对比

- 如果是新增属性，那么存到 addedMap 里
- 如果是更新属性，那么把原来的键值存到 prevMap，把新的键值存到 newMap

##### ProxySandbox

以上两种沙箱只能运行于单例模式，本质也是操作 window，如果同时存在两个子应用则无法满足

ProxySandbox 利用 Proxy 将 window 赋值出来，称为 fakeWindow，即每个子应用可对应一个 fakeWindow

写数据时

1. fakeWindow 没有，但 globalWindow 有，则写入 globalWindow，并同时写入到 fakeWindow
2. 否则直接写入 fakeWindow

读数据时

1. 对属性是有判断的，例如 top、parent 等从 globalWindow 获取，否则从 fakeWindow

##### 如何实现隔离

通过 wrapper 包裹后，进行传参（可想象 cjs）

```js
function (window){
  // 以下函数运行的时候，都是参数传递的 window
}
```

### micro app

### wujie

1. 使用 iframe 做为 JS 沙箱，来解决 JS 沙箱性能的问题
2. 使用 web-component 来隔绝各个应用
3. 应用级别的 keep-alive

## 打包工具

webpack

1. 生态较好
2. 开发体验较差
3. 构建速度较慢
4. webpack 也可以使用 esbuild-loader 来提速，用于替换 babel-loader 和 ts-loader¬

vite

1. 生态较 webpack 差
2. 构建速度快（esbuild 快，go 的天然优势，esbuild 中途减少了 AST 的转换，尽可能的复用 AST）
3. 开发环境工具（esbuild）和生产环境工具(rollup)不一致
4. 在大文件或者较多网络请求时，表现较差

why rollup：相较于 webpack 生态一般，但相较于 webpack

- 更加轻量，打包出来的代码更简洁
- 使用 esm 标准模块（通过插件可以导入现有的 cjs）

### webpack

#### 启动

会走一遍完成的 bundle 的过程

file string => module => AST(dependence 分析) => module

1. resolve：根据 entry 定位入口文件路径，创建一个 Module 对象
2. load：读取入口文件
3. transform：调用 loader 处理
4. parse：将处理好的文件解析为 AST，并分析 AST 找出依赖，并对依赖处理（从步骤 1 循环，知道依赖处理完毕）

对 module graph 处理

1. tree shaking
2. module 分类，根据静态加载还是动态加载，将 module 分为 init chunk 和 async chunk
3. 优化 init chunk 和 async chunk 重复 module
4. 分离 chunk，例如处理三方依赖
5. 根据 chunk 获取到 template，并根据 template 输出

#### 热更新

watch 文件变化，根据变化的文件，再走一次 bundle 流程，当 bundle 完后，通过 ws 通知浏览器获取新的打包文件

1. 启动时，dev server 通过启动一个服务

   - 提供两种服务，一个是 HTTP 服务，返回静态资源，二是 Socket 服务；向前端插入 Socket 代码，可以和浏览器通信
   - 向模块注入 createModuleHotObject 函数，创建 Module 时，会进行调用

   ```js
   module.hot = createModuleHotObject();

   createModuleHotObject = () => {
     var hot = {
       _requireSelf: function () {
         currentParents = me.parents.slice();
         currentChildModule = _main ? undefined : moduleId;
         __webpack_require__(moduleId);
       },

       accept() {},

       check() {},
     };

     return hot;
   };
   ```

2. webpack 监听文件修改，当文件修改时，触发从新编译
3. webpack 监听编译完成，向浏览器通过 socket 发送 hash 事件
4. 浏览器接收到推送，调用 hotCheck，向服务端获取补丁文件，根据 hash -> hash.json -> hash.js
5. 模块替换

   - 根据 module id 找到旧模块 cache，删除掉。并删除 children 中的依赖(parents)

   ```js
   delete __webpack_require__.c[moduleId];
   ```

   - 加载新的模块，写入 cache

   ```js
   outdatedSelfAcceptedModules.push({
     module: outdatedModuleId,
     require: module.hot._requireSelf, // 调用此方法，会调用一次 __webpack_require__(moduleId);
     errorHandler: module.hot._selfAccepted,
   });

   // 修改 module
   __webpack_require__.m[updateModuleId] = appliedUpdate[updateModuleId];
   ```

#### react 热更新（react-refresh）

可以实现组件级别的刷新，且可以保存原组件的状态

- webpack 的 HRM，hot-update.js
- react-refresh
- react-refresh-webpack-plugin
- react reconciler

1. 在获取 hot-update.js 中，有段代码 `__webpack_require__.$Refresh$.register(_c, "Home");`，调用的是 react-refresh 中的 register

   ```js
   register(Home, 'home');
   // register 向 allFamiliesByID 中存储组件，如果组件不存在，则存储，如果组件存在，则插入 pendingUpdates
   // pendingUpdates = [ [{ current: Home },Home] ]
   ```

2. 在 hot-update.js 中，$ReactRefreshModuleRuntime$() -> executeRuntime() 方法

   ```js
   // executeRuntime 方法中
   function executeRuntime() {
     if (isReactRefreshBoundary(moduleExports)) {
       enqueueUpdate(); // 局部刷新
     } else {
       webpackHot.invalidate(); // 全局刷新
     }
   }
   ```

3. enqueueUpdate 最终调用的是 react-refresh performReactRefresh 方法

   ```js
    function performReactRefresh(){
     updates.forEach(([family, nextType]) => {
       const prevType = family.current;

       updatedFamiliesByType.set(prevType, family); // WeakMap
       updatedFamiliesByType.set(nextType, family);

       family.current = nextType;

       if (canPreserveStateBetween(prevType, nextType)) {
         updatedFamilies.add(family);
       } else {
         staleFamilies.add(family);
       }
     });

     mountedRootsSnapshot.forEach(root => {
       ...

       helpers.scheduleRefresh(root, update); // root即为 filerRootNode

       ...
     });
    }
   ```

4. reconciler 的 scheduleRefresh 调用 scheduleFibersWithFamiliesRecursively，scheduleFibersWithFamiliesRecursively 内会根据 updatedFamilies 和 staleFamilies 找到对应的 Fiber

   ```js
   flushSync(() => {
     scheduleFibersWithFamiliesRecursively(
       root.current,
       updatedFamilies,
       staleFamilies
     );
   });
   ```

5. 最终调用 `scheduleUpdateOnFiber(_root, fiber, SyncLane, NoTimestamp);`

6. 然后在 beginWork 中，创建 Fiber 时

   ```js
     function createWorkInProgress(){
       ...

       if (__DEV__) {
         workInProgress._debugNeedsRemount = current._debugNeedsRemount;
         switch (workInProgress.tag) {
           case IndeterminateComponent:
           case FunctionComponent:
           case SimpleMemoComponent:
             workInProgress.type = resolveFunctionForHotReloading(current.type);
             break;

             ...
         }
       }
     }
   ```

7. resolveFunctionForHotReloading

   ```js
   // reconciler ReactFiberHotReloading.new.js
   function resolveFunctionForHotReloading(){
     ...
     const family = resolveFamily(type); // resolveFamily 通过 setRefreshHandler 在 react-refresh写入
     ...
   }

    // refresh ReactFreshRuntime.js
   function resolveFamily(type) {
     return updatedFamiliesByType.get(type);
   }
   ```

### vite

开发环境

1. 使用 esbuild 打包外部的依赖
2. 响应浏览器的 `<script type="module">` 发起的请求

生产环境

1. 使用 rollup 打包

#### 启动

vite 没有启动 bundle 这一个步骤，他是通过浏览器请求并 hack 此次请求，通过对浏览器请求的资源进行 resolve -> load -> transform -> parse

#### 热更新

watch 文件变化，通知浏览器去加载变化的文件，然后自动会走 resolve -> load -> transform -> parse 的流程

### rollup

### turbopack

1. rust 天然优势
2. incremental computation
   - 增量计算
   - 利用 turbo 实现并行的使用多个核心
   - 缓存，对函数的调度运行结果进行缓存，避免二次计算
3. Lazy bundling，按需打包（dev）

## PWA & Service Worker

what、why、how

## 浏览器 JS 代码执行

编译语言：源代码 => AST => 字节码 => 二进制代码
JS：源代码 => AST => Ignition => 字节码 => 机器码（其中 Ignition 会收集代码的执行过程，发现如果有热代码，则使用 Turbofan 编译为机器码）

为什么需要使用字节码？原来架构中是没有字节码的，直接将 AST 转化为机器码执行，但是随着手机的流行，V8 需要大量的内存去存储转换后的机器码，手机内存可能没这么大，所以对架构进行了改进。相对来说，字节码比机器码占用空间少很多

## 报警方案

1. 接入日志平台

   - 平台建设
   - SDK 建设

2. 信噪比

   - 明确报警等级
   - 消除无用报警，或消除暂时不关注的报警
   - 聚合同类型报警
   - 合理降级错误
   - 确定报警的阈值

3. 报警信息

   - 日志上报规范
   - 明确报警等级
   - 关键节点日志（开发宣讲，CR）
   - info 日志

4. 跟进、复盘

   - 出现报警的跟进人
   - 每周的报警分析
   - 分析阈值的合理性

## npm、yarn、pnpm

扁平化

advantages

1. 层级较浅
2. 重复复用包，减少包的下载

disadvantages

1. 层级不稳定
2. 扁平化算法的时间成本
3. 幽灵依赖

pnpm

1. 下载速度快
2. 高效缓存，磁盘利用率：针对同一个版本的包，只会存储一份，后续使用「硬链接」；针对不同版本的包，会尽可能复用之前的版本代码
3. 解决幽灵依赖：

   - 将文件下载安装于 node_modules/.pnpm 中，在 node_modules 下只会有当前 packages.json 中声明的包，且使用「软连接」链接到 node_modules/.pnpm
   - node_modules/.pnpm 下的包有一定的结构性

     ```txt
     .pnpm
      - koa2
        - node_modules
          - koa-compose 「软链接」到 .pnpm/koa-compose
      - koa-compose
     ```

4. 支持 monorepo

## diff 算法

### vue

1. 双端比较
   通过 oldStart，oldEnd，newStart，newEnd 来比较

   1. 比较出现相等，则按照比较的交换位置
   2. 比较无法相等

      - 如果 new key 可以在 old 中找到，则人工构建一个相等出来（把 old key 中的节点移动 oldStart 去，构造出 oldStart === newStart）
      - 否则新增节点

   3. 最后判断是否存在新增和删除的节点

   ```js
   // [5,1,2,3,4,6] => [1,2,3,4] 不太好处理如下情况，但lastIndex算法较好处理
   ```

2. 前缀后缀（插头去尾） + lastIndex + 最大增子序列

   1. 去除可复用的前缀和后缀
   2. 删除 old 中存在，但 new 中不存在的节点
   3. 通过 lastIndex 算法，判断节点是否需要移动
   4. 如果需要移动，则使用最大增子序列算法，算出无需要移动的节点
      - 需要移动：遍历新的节点；新增节点则直接添加；否则如果当前节点索引为 lis，则无需移动；否则将节点移动到此
      - 无需移动：判断下是否存在新增节点

### react

1. 第一次遍历，掐头
2. lastPlacedIndex

   通过 lastPlacedIndex 来计算老的节点是否需要移动

   ```js
   // [1,2,3,4] => [4,1,2,3] // 不太好处理如下情况，但双端比较较好处理
   ```

3. 遍历一次，删除无用节点
