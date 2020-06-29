# June-1.md

## Flutter

### 事件

#### Listener 基本事件

在移动端，各个平台或 UI 系统的原始指针事件模型基本都是一致，即：一次完整的事件分为三个阶段：**手指按下、手指移动、和手指抬起**，而更高级别的手势（如点击、双击、拖动等）都是基于这些原始事件的

Flutter 存在冒泡机制，和 Web 开发中浏览器的事件冒泡机制相似， 但是 Flutter 中**没有机制取消或停止“冒泡”过程**，而浏览器的冒泡是可以停止的。注意，**只有通过命中测试的组件才能触发事件**

```dart
Listener({
  Key key,
  this.onPointerDown, //手指按下回调
  this.onPointerMove, //手指移动回调
  this.onPointerUp,//手指抬起回调
  this.onPointerCancel,//触摸事件取消回调
  this.behavior = HitTestBehavior.deferToChild, //在命中测试期间如何表现
  Widget child
})
```

##### PointerEvent

PointerDownEvent、PointerMoveEvent、PointerUpEvent 都是 PointerEvent 的一个子类，PointerEvent 类中包括当前指针的一些信息，如：

- position：它是鼠标相对于当对于全局坐标的偏移。
- delta：两次指针移动事件（PointerMoveEvent）的距离。
- pressure：按压力度，如果手机屏幕支持压力传感器(如 iPhone 的 3D Touch)，此属性会更有意义，如果手机不支持，则始终为 1。
- orientation：指针移动方向，是一个角度值

##### HitTestBehavior

- deferToChild：子组件会一个接一个的进行命中测试，如果子组件中有测试通过的，则当前组件通过，这就意味着，如果指针事件作用于子组件上时，其父级组件也肯定可以收到该事件
- opaque：在命中测试时，将当前组件当成不透明处理(即使本身是透明的)，最终的效果相当于当前 Widget 的整个区域都是点击区域
- translucent：当点击组件透明区域时，可以对自身边界内及底部可视区域都进行命中测试，这意味着点击顶部组件透明区域时，顶部组件和底部组件都可以接收到事件

##### 忽略 PointerEvent

- IgnorePointer：本身会参与命中测试
- AbsorbPointer：本身不会参与命中测试

AbsorbPointer 本身是可以接收指针事件的(但其子树不行)，而 IgnorePointer 不可以

#### GestureDetector 手势事件

```dart
GestureDetector({
// 点击事件
this.onTapDown,
this.onTapUp,
this.onTap,
this.onTapCancel，
this.onDoubleTap, // 双击
this.onLongPress, // 长按

// 滑动
this.onPanDown, // 按下
this.onPanStart,
this.onPanUpdate, // 滑动
this.onPanEnd, // 结束
this.onPanCancel, // 取消

// 单一方向拖动
this.onVerticalDragUpdate, // 垂直拖动
...
this.onHorizontalDragUpdate,
...

// 缩放
this.onScaleUpdate,
})
```

```dart
GestureDetector(
  child: Container(
    color: _color,
    constraints: BoxConstraints.expand(height: 50),
  ),
  onTap: () {
    print('tap');
    setState(() {
      _color = Colors.lightBlue;
    });
  },
  onDoubleTap: () {
    setState(() {
      _color = Colors.black45;
    });
  },
  onLongPress: () {
    setState(() {
      _color = Colors.green;
    });
  },
)
```

##### GestureRecognizer(abstract)

GestureDetector 内部是使用一个或多个 GestureRecognizer 来识别各种手势的，而 GestureRecognizer 的作用就是通过 Listener 来将原始指针事件转换为语义手势，GestureDetector 直接可以接收一个子 widget

常用的有子类有：

- TapGestureRecognizer：点击

```dart
TextSpan(
  text: 'Home',
  style: TextStyle(
      fontSize: 60, decoration: TextDecoration.underline, color: toggle ? Colors.red : Colors.blue),
  recognizer: _tap
    ..onTap = () {
      setState(() {
        toggle = !toggle;
      });
    }
)
```

##### 手势竞争与冲突

Flutter 中的手势识别引入了一个 Arena 的概念，Arena 直译为“竞技场”的意思，每一个手势识别器（GestureRecognizer）都是一个“竞争者”（GestureArenaMember），当发生滑动事件时，他们都要在“竞技场”去竞争本次事件的处理权，而最终只有一个“竞争者”会胜出(win)

我们以拖动手势为例，同时识别水平和垂直方向的拖动手势，当用户按下手指时就会触发竞争（水平方向和垂直方向），一旦某个方向“获胜”，则直到当次拖动手势结束都会沿着该方向移动

**由于手势竞争最终只有一个胜出者，所以，当有多个手势识别器时，可能会产生冲突**，如果我们的代码逻辑中，对于手指按下和抬起是强依赖的，比如在一个轮播图组件中，我们希望手指按下时，暂停轮播，而抬起时恢复轮播，但是由于轮播图组件中本身可能已经处理了拖动手势（支持手动滑动切换），甚至可能也支持了缩放手势，这时我们如果在外部再用 onTapDown、onTapUp 来监听的话是不行的，此时使用**Listener**；

#### 全局事件总线

发布订阅 + 单例模式

#### 通知

通知（Notification）是 Flutter 中一个重要的机制，在 widget 树中，每一个节点都可以分发通知，通知会沿着当前节点向上传递，所有父节点都可以通过 NotificationListener 来监听通知。Flutter 中将这种由子向父的传递通知的机制称为通知冒泡（Notification Bubbling）。通知冒泡和用户触摸事件冒泡是相似的，但有一点不同：通知冒泡可以中止，但用户触摸事件不行

```dart
NotificationListener(
  onNotification: (notification) {
    print(notification.runtimeType);
    return false; // 阻止冒泡
  },
  child: ListView(
    children: List.filled(100, 1).map((e) => Center(child: Text(e.toString()))).toList(),
  ),
);
```

##### 自定义通知

```dart
class SelfNotice extends Notification {

  @override
  void dispatch(BuildContext target) { // 分发通知
    // implement dispatch
    super.dispatch(target);
  }
}
```

### 动画

#### Animation

Animation 是一个抽象类，它本身和 UI 渲染没有任何关系，而它主要的功能是**保存动画的插值和状态**；其中一个比较常用的 Animation 类是 `Animation<double>`

- addListener()；它可以用于给 Animation 添加帧监听器，在每一帧都会被调用。帧监听器中最常见的行为是改变状态后调用 setState()来触发 UI 重建
- addStatusListener()；它可以给 Animation 添加 a“动画状态改变”监听器；动画开始、结束、正向或反向（见 AnimationStatus 定义）时会调用状态改变的监听器

#### Curve

描述动画过程

| Curves 曲线        |             动画过程 |
| :----------------- | -------------------: |
| linear             |               匀速的 |
| decelerate 匀      |                 减速 |
| ease 开始加速，    |             后面减速 |
| easeOut 开始快，   |               后面慢 |
| easeInOut 开始慢， | 然后加速，最后再减速 |

#### AnimationController

控制动画，例如开始，暂停，反向

##### Ticker: TickerProvider

Flutter 应用在启动时都会绑定一个 SchedulerBinding，通过 SchedulerBinding 可以给每一次屏幕刷新添加回调，而 Ticker 就是通过 SchedulerBinding 来添加屏幕刷新回调，这样一来，每次屏幕刷新都会调用 TickerCallback。使用 Ticker(而不是 Timer)来驱动动画会防止屏幕外动画（动画的 UI 不在当前屏幕时，如锁屏时）消耗不必要的资源，因为 Flutter 中屏幕刷新时会通知到绑定的 SchedulerBinding，而 Ticker 是受 SchedulerBinding 驱动的，由于锁屏后屏幕会停止刷新，所以 Ticker 就不会再触发

#### Tween

AnimationController 对象值的范围是[0.0，1.0]。如果我们需要构建 UI 的动画值在不同的范围或不同的数据类型，则可以使用 Tween 来添加映射以生成不同的范围或数据类型的值

比如可以在一个动画内，不同阶段触发不同的动画

##### Tween.animate

## Vue

### Vue3.0 重写原因及优势

**重写原因**：

1. Vue2.0 和 Vue3.0 的响应式数据核心改变，由 `Object.defineProperty=>Proxy`
2. Vue2.0 的类型体系为 flow，vue3.0 改为 typescript
3. 解耦内部包
4. 解决内部架构问题（解决技术债）

**重写优势**：

1. 更小：体积更小，vue3.0 压缩后（开启 gzip）只有不到 10kb
   - 大多数全局 API 和内部 helper 移到了 ES 模块导出中，从而实现了这个目标，实现**树摇优化**
2. 更快
   - 在树级别：根据结构化指令（例如 v-for，v-if）将模板拆分为**嵌套块**，当块中的节点更新时，无序遍历整个树，避免遍历的开销
   - 在编译器：编译器会检查模板，将**静态数据拆分到渲染函数之外**（render）,减少每次渲染的开销，减少内存占用，减少 GC 次数
   - 在元素级别：根据元素更新类型生成优化标志，减少 diff 时间
3. Composition API
4. Proxy API
   - 直接监听对象，而不需要遍历监听属性
   - 可以监听数组
   - 对于新增属性也可以监听
   - 可以监听任意属性（_虽然想要监听到变化属性的准确 key，还是需要递归的_）

## LeetCode

### 不使用乘除，for，while，if，else，switch 等计算 1+2+3+...n

思路：可以使用加减，即要一个个计算，既然无法使用循环，则可以使用递归；无法使用 if，可以考虑使用 && 或 ||

```typescript
function sumNums(n: number): number {
  n > 0 && (n += sumNums(n - 1));
  return n;
}
```

[计算 1+2+3+...n](https://leetcode-cn.com/problems/qiu-12n-lcof/submissions/)

### 除自身以外数组的乘积

思路 1：遍历一遍，计算出所有；再遍历一遍出去自己
思路 2：由于题目要求不能使用除法，则使用两次遍历法，1. 从左到右；2. 从右到左

```typescript
function productExceptSelf(nums: number[]): number[] {
  const store: number[] = new Array(nums.length);

  store[0] = 1;

  let i = 1;
  let prevCount = 1;
  let afterCount = 1;

  // 这样就将第i的左边积求出来了
  for (; i < nums.length; i++) {
    prevCount *= nums[i - 1];
    store[i] = prevCount;
  }

  i = nums.length - 2;
  for (; i >= 0; i--) {
    afterCount *= nums[i + 1];
    store[i] *= afterCount;
  }

  return store;
}
```

[除自身以外数组的乘积](https://leetcode-cn.com/problems/product-of-array-except-self/)
