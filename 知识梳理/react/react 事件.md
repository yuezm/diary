---
theme: channing-cyan
---

# React 事件

React 事件是在原生事件的基础上再封装了一层，且模拟实现了原生事件的捕获、冒泡阶段。我们可以将整体拆分为三个阶段来学习 `事件的绑定`、`事件的触发及收集`、`回调函数执行`

提示

- 文章的源代码是基于 React 18.2.0
- 文章内贴的源代码都只是保留关键部分，完整的代码可以根据代码位置自行查阅

## 事件的绑定

React 将事件分为`可以委托的事件`（可以冒泡的事件）和`不可以委托的事件`（不可以冒泡的事件）。可以委托的事绑定于 `rootContainerElement`（这个是 React 挂载的根元素）上，例如 click 事件；不可以委托的事件绑定于对应的元素上，例如 img 的 load 事件

### 对于可以委托的事件

对于可以冒泡的事件，在 `createRoot` 方法内，通过调用 `listenToAllSupportedEvents` 方法进行绑定

#### listenToAllSupportedEvents

```ts
/**
 * 代码位置 packages/react-dom/src/events/DOMPluginEventSystem.js
 */
export function listenToAllSupportedEvents(rootContainerElement: EventTarget) {
  ...
  // allNativeEvents保存了一系列的事件名称，当然也包括click事件，其成员可见simpleEventPluginEvents
  allNativeEvents.forEach((domEventName) => {
    // 将不可冒泡的事件排除在外
    if (!nonDelegatedEvents.has(domEventName)) {
      // 冒泡阶段的事件监听
      listenToNativeEvent(domEventName, false, rootContainerElement);
    }
    // 捕获阶段的事件监听
    listenToNativeEvent(domEventName, true, rootContainerElement);
  });
  ...
}
```

#### listenToNativeEvent

```ts
/**
 * 代码位置 packages/react-dom/src/events/DOMPluginEventSystem.js
 */
export function listenToNativeEvent(
  domEventName: DOMEventName,
  isCapturePhaseListener: boolean,
  target: EventTarget
): void {
  ...
  let eventSystemFlags = 0;
  // 添加捕获事件标识
  if (isCapturePhaseListener) {
    eventSystemFlags |= IS_CAPTURE_PHASE;
  }

  addTrappedEventListener(
    target,
    domEventName,
    eventSystemFlags,
    isCapturePhaseListener
  );
  ...
}
```

#### addTrappedEventListener

addTrappedEventListener 是事件绑定的核心方法，后续的不可以冒泡事件也是由此方法进行绑定

```ts
/**
 * 代码位置 packages/react-dom/src/events/DOMPluginEventSystem.js
 */
function addTrappedEventListener(
  targetContainer: EventTarget,
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  isCapturePhaseListener: boolean,
  isDeferredListenerForLegacyFBSupport?: boolean
) {
  ...
  // 这个才是绑定到DOM元素上的事件处理函数
  let listener = createEventListenerWrapperWithPriority(
    targetContainer,
    domEventName,
    eventSystemFlags
  );

  if (isCapturePhaseListener) {
    // 捕获阶段的事件监听
    unsubscribeListener = addEventCaptureListener(
      targetContainer,
      domEventName,
      listener
    );
  } else {
    // 冒泡阶段的事件监听
    unsubscribeListener = addEventBubbleListener(
      targetContainer,
      domEventName,
      listener
    );
  }
...
}
```

- addEventCaptureListener、addEventBubbleListener 内部都是调用的原生的 addEventListener，只是针对捕获、冒泡阶段及 passive 传递参数不同
- 这里的 listener 才是元素触发事件时的回调函数（而非我们在 JSX 中写入的事件回调函数），即当 rootContainerElement 触发事件时（例如 click 事件），就会执行该函数

至此针对可冒泡事件就绑定完毕了

### 对于不可冒泡事件

对于不可冒泡事件，在 reconcile 的 `completeWork` 阶段，执行 `finalizeInitialChildren` 方法进行绑定

#### finalizeInitialChildren

```ts
/*
 * 代码位置 packages/react-dom/src/client/ReactDOMHostConfig.js
 */
export function finalizeInitialChildren(
  domElement: Instance,
  type: string,
  props: Props,
  rootContainerInstance: Container,
  hostContext: HostContext
): boolean {
  ...
  setInitialProperties(domElement, type, props, rootContainerInstance);
  ...
}
```

#### setInitialProperties

根据不同元素，来绑定不同的事件

- listenToNonDelegatedEvent 最终也是通过 addTrappedEventListener 方法来为元素绑定事件，addTrappedEventListener 方法在上面已经介绍过了
- 可以看到此时是没有判断 props 传值来绑定事件的，表示即使你在书写 JSX 时 `<img>` 标签即使不传入 onLoad 等等，也是会绑定 load 事件的

```ts
/*
 * 代码位置 packages/react-dom/src/client/ReactDOMComponent.js
 */
export function setInitialProperties(
  domElement: Element,
  tag: string,
  rawProps: Object,
  rootContainerElement: Element | Document | DocumentFragment
): void {
  ...
  // 根据不同的标签，来绑定不同的事件
  switch (tag) {
    case 'dialog':
      listenToNonDelegatedEvent('cancel', domElement);
      listenToNonDelegatedEvent('close', domElement);
      props = rawProps;
      break;

    case 'img':
    case 'image':
    case 'link':
      listenToNonDelegatedEvent('error', domElement);
      listenToNonDelegatedEvent('load', domElement);
      props = rawProps;
      break;
  }
  ...
}
```

## 事件的触发及依赖收集

当元素触发事件后，对于可以冒泡的事件，即冒泡到 rootContainerElement 后触发事件，并执行回调；对于不可冒泡事件，则直接执行回调。步骤如下

1. 通过 JS 原生事件的 event.target 找到触发的元素，并根据元素获取到对应的 Fiber 节点
2. 找到对应的 Fiber 节点后，向上遍历，收集事件 & 事件处理函数（JSX 中传入的事件回调），插入队列中
3. 当收集完毕后，执行刚刚收集的事件处理函数

假设我们先有如下组件，绑定了如下事件，并最终用鼠标点击了一次 img 元素

```jsx
function App() {
  return (
    <div onClick={() => console.log('click div')}>
      <img src="xx"
        onLoad={() => { console.log('load img') }}
        onClick={() => console.log('click image')}
      />
    </div>
  )
}
}
```

### 元素触发事件并执行回调函数

我们刚刚点击了一下 img 元素，当事件进入冒泡阶段时， rootContainerElement 触发 click 事件，并执行由 addTrappedEventListener 绑定 listener。listener 由 createEventListenerWrapperWithPriority 创建而得

#### createEventListenerWrapperWithPriority

```ts
/**
 * 代码位置 packages/react-dom/src/events/ReactDOMEventListener.js
 */
export function createEventListenerWrapperWithPriority(
  targetContainer: EventTarget, // rootContainerElement
  domEventName: DOMEventName, // click
  eventSystemFlags: EventSystemFlags // 0
): Function {
  ...
  const eventPriority = getEventPriority(domEventName);
  let listenerWrapper;

  // 区分优先级
  switch (eventPriority) {
    case DiscreteEventPriority:
      listenerWrapper = dispatchDiscreteEvent;
      break;
    case ContinuousEventPriority:
      listenerWrapper = dispatchContinuousEvent;
      break;
    case DefaultEventPriority:
    default:
      listenerWrapper = dispatchEvent;
      break;
  }
  return listenerWrapper.bind(
    null,
    domEventName,
    eventSystemFlags,
    targetContainer
  );
  ...
}
```

- 这里进行了优先级的判断，会根据事件的名称来赋予不同的优先级，例如 click 事件，他的优先级就是 SyncLane，除去优先级，其内部调用的方法相同，最终都是调用的 dispatchEvent

#### dispatchEventWithEnableCapturePhaseSelectiveHydrationWithoutDiscreteEventReplay

dispatchEvent 方法比较简单，所以没有列出，其核心就是调用 dispatchEventWithEnableCapturePhaseSelectiveHydrationWithoutDiscreteEventReplay 方法

```ts
/**
 * 代码位置 packages/react-dom/src/events/ReactDOMEventListener.js
 */
function dispatchEventWithEnableCapturePhaseSelectiveHydrationWithoutDiscreteEventReplay(
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  targetContainer: EventTarget,
  nativeEvent: AnyNativeEvent
) {
  ...
  // 找到 event.target 所对应的 Fiber
  let blockedOn = findInstanceBlockingEvent(
    domEventName,
    eventSystemFlags,
    targetContainer,
    nativeEvent
  );

  if (blockedOn === null) {
    // 执行依赖收集函数
    dispatchEventForPluginEventSystem(
      domEventName,
      eventSystemFlags,
      nativeEvent,
      return_targetInst,
      targetContainer
    );
  }
  ...
}
```

- 在每个 Fiber 中，我们通过 startNode 来可以访问对用的 DOM。当然在对应的 DOM 中，我们也可以通过变量 `internalInstanceKey` 来访问对应的 Fiber，由此我们通过 event.target 找到触发当前事件的 Fiber。此次 event.target 表示的 img 元素，return_targetInst 表示的 img 对应的 Fiber

### 收集事件 & 事件处理函数

dispatchEventForPluginEventSystem 方法比较简单，所以没有列出，其核心就是调用了 dispatchEventsForPlugins 方法

#### dispatchEventsForPlugins

```ts
/**
 * 代码位置 packages/react-dom/src/events/DOMPluginEventSystem.js
 */
function dispatchEventsForPlugins(
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  nativeEvent: AnyNativeEvent,
  targetInst: null | Fiber,
  targetContainer: EventTarget
): void {
  ...
  const nativeEventTarget = getEventTarget(nativeEvent);

  // 存储的事件 & 事件处理函数的队列
  const dispatchQueue: DispatchQueue = [];

  // 根据当前Fiber去收集事件 & 事件处理函数
  extractEvents(
    dispatchQueue,
    domEventName,
    targetInst,
    nativeEvent,
    nativeEventTarget,
    eventSystemFlags,
    targetContainer
  );

  // 依赖收集完毕后开始执行，后续再说
  processDispatchQueue(dispatchQueue, eventSystemFlags);
}
```

#### extractEvents

extractEvents 方法内核心是调用了 SimpleEventPlugin.extractEvents，所以没有单独列出 extractEvents 的源代码

```ts
/**
 *代码位置 packages/react-dom/src/events/plugins/SimpleEventPlugin.js
 */
function extractEvents(
  dispatchQueue: DispatchQueue,
  domEventName: DOMEventName,
  targetInst: null | Fiber,
  nativeEvent: AnyNativeEvent,
  nativeEventTarget: null | EventTarget,
  eventSystemFlags: EventSystemFlags,
  targetContainer: EventTarget
) {
  ...
  // 收集事件，并对事件进行封装，以及其他兼容性的处理，这个也就是后续我们在 JSX 回调函数中可以获取到的事件
  let SyntheticEventCtor = SyntheticEvent;
  switch (domEventName) {
    case 'click':
      SyntheticEventCtor = SyntheticMouseEvent;
      break;
  }

  // 收集事件的处理函数
  const listeners = accumulateSinglePhaseListeners(
    targetInst,
    reactName,
    nativeEvent.type,
    inCapturePhase,
    accumulateTargetOnly,
    nativeEvent
  );

  if (listeners.length > 0) {
    const event = new SyntheticEventCtor(
      reactName,
      reactEventType,
      null,
      nativeEvent,
      nativeEventTarget
    );

    // 依赖收集完毕后，插入队列中
    dispatchQueue.push({ event, listeners });
  }
  ...
}
```

#### accumulateSinglePhaseListeners

```ts
/**
 * 代码位置 packages/react-dom/src/events/DOMPluginEventSystem.js
 */
export function accumulateSinglePhaseListeners(
  targetFiber: Fiber | null,
  reactName: string | null,
  nativeEventType: string,
  inCapturePhase: boolean,
  accumulateTargetOnly: boolean,
  nativeEvent: AnyNativeEvent
): Array<DispatchListener> {
  ...
  const captureName = reactName !== null ? reactName + 'Capture' : null;
  const reactEventName = inCapturePhase ? captureName : reactName;

  let listeners: Array<DispatchListener> = [];
  let instance = targetFiber;

  while (instance !== null) {
    const { stateNode, tag } = instance;

    // Fiber 类型必须是 HostComponent，例如 div，img
    if (tag === HostComponent && stateNode !== null) {
      if (reactEventName !== null) {
        // 通过 reactEventName 去匹配该 Fiber 的 props，匹配成功后，即可获得事件的执行函数
        const listener = getListener(instance, reactEventName);
        if (listener != null) {
          listeners.push(
            createDispatchListener(instance, listener, lastHostComponent)
          );
        }
      }
    }
    ...

    // 向父节点搜索
    instance = instance.return;
  }
  return listeners;
}
```

- 收集事件的处理函数比较清晰，概括来说就是从 targetFiber 开始向上搜索，拿事件的名称去匹配每个 Fiber 的 Props。拿此次的例子举例，此次的事件名称为 click，则计算出 reactEventName 为 onClick（如果是捕获阶段的话为 onClickCapture），img 的 Fiber 存在如下 props { onClick: () => console.log('click image') }，即此次匹配成功，会将收集到的事件和事件处理函数存储队列中

## 事件处理函数执行

回到 dispatchEventsForPlugins 函数，当收集完毕后，就开始执行收集的任务了

```ts
/**
 * 代码位置 packages/react-dom/src/events/DOMPluginEventSystem.js
 */
function dispatchEventsForPlugins(
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  nativeEvent: AnyNativeEvent,
  targetInst: null | Fiber,
  targetContainer: EventTarget
): void {
  const nativeEventTarget = getEventTarget(nativeEvent);

  // 存储的事件 & 事件处理函数的队列
  const dispatchQueue: DispatchQueue = [];

  // 根据当前Fiber去收集事件 & 事件处理函数
  extractEvents(
    dispatchQueue,
    domEventName,
    targetInst,
    nativeEvent,
    nativeEventTarget,
    eventSystemFlags,
    targetContainer
  );

  // 依赖收集完毕后开始执行
  processDispatchQueue(dispatchQueue, eventSystemFlags);
}
```

#### processDispatchQueue

```ts
/**
 *代码位置 packages/react-dom/src/events/DOMPluginEventSystem.js
 */
export function processDispatchQueue(
  dispatchQueue: DispatchQueue,
  eventSystemFlags: EventSystemFlags
): void {
  ...
  const inCapturePhase = (eventSystemFlags & IS_CAPTURE_PHASE) !== 0;

  for (let i = 0; i < dispatchQueue.length; i++) {
    const { event, listeners } = dispatchQueue[i];

    processDispatchQueueItemsInOrder(event, listeners, inCapturePhase);
    //  event system doesn't use pooling.
  }
  // This would be a good time to rethrow if any of the event handlers threw.
  rethrowCaughtError();
}
```

#### processDispatchQueueItemsInOrder

```ts
/**
 *代码位置 packages/react-dom/src/events/DOMPluginEventSystem.js
 */
function processDispatchQueueItemsInOrder(
  event: ReactSyntheticEvent,
  dispatchListeners: Array<DispatchListener>,
  inCapturePhase: boolean
): void {
  ...
  if (inCapturePhase) {
    // 捕获阶段从后往前
    for (let i = dispatchListeners.length - 1; i >= 0; i--) {
      const { instance, currentTarget, listener } = dispatchListeners[i];
      if (instance !== previousInstance && event.isPropagationStopped()) {
        return;
      }
      executeDispatch(event, listener, currentTarget);
      previousInstance = instance;
    }
  } else {
    // 冒泡阶段从前往后
    for (let i = 0; i < dispatchListeners.length; i++) {
      const { instance, currentTarget, listener } = dispatchListeners[i];

      // 这里调用了 isPropagationStopped，如果事件在冒泡时，执行力event.stopPropagation()，则此函数返回为true，后续的listener也不会执行了
      if (instance !== previousInstance && event.isPropagationStopped()) {
        return;
      }

      // 执行事件的回到函数，此时才是真正执行 JSX 中传递的事件回调函数
      executeDispatch(event, listener, currentTarget);
      previousInstance = instance;
    }
  }
  ...
}
```

- React 进行收集时的顺序是从 子 Fiber -> 根 Fiber，所以存储在队列中的事件执行函数顺序也是如此，即顺序执行就可以模拟冒泡阶段，逆序执行就可以模拟捕获阶段

## 总结

1. React 将事件分为可以委托的事件（可以冒泡的事件）和不可以委托的事件（不可以冒泡的事件）。可以委托的事件在 createRoot 段绑定于 rootContainerElement；不可以委托的事件在 completeWork 段绑定于对应的元素上
2. 当一个元素触发事件后，对于可以冒泡的事件，即冒泡到 rootContainerElement 后触发事件，并执行回调；对于不可冒泡事件，则直接执行回调。捕获事件和冒泡事件执行步骤相似，只是捕获先于冒泡执行，也就是说，步骤 3，4 是会执行两次的，一次是捕获，一次是冒泡
3. 执行回调时，通过 JS 原生事件的 event.target 找到触发的元素，并根据元素获取到对应的 Fiber 节点，找到对应的 Fiber 节点后，向上遍历，收集事件 & 事件处理函数（JSX 中传入的事件回调），插入队列中
4. 收集完毕后，根据不同阶段，捕获阶段从后往前执行，冒泡节点从前往后执行，依次清空队列

## 思考

1. 子元素如此绑定 `<div onClick={(event) => { event.stopPropagation() }}></div>` 后，父元素通过 ref.current.addEventListener 还可以监听到吗
2. 子元素通过 ref.current.addEventListener 内 event.stopPropagation() 后，父元素如此绑定 `<div onClick={(event) => { event.stopPropagation() }}></div>`还可以监听到吗
