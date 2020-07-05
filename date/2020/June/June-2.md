# June-2

## Typescript

### interface 扩展

1. 对 node_modules 的扩展

   例如，对 express 的 Request 扩展

   ```typescript
   declare module express {
     export interface Request {
       ...
     }
   }
   ```

2. 对本地模块的扩展

   例如，对同级目录的 type.d.ts 的 Request 进行扩展

   ```typescript
   declare module './type' {
     export interface Request{
       ...
     }
   }
   ```

**特别注意**：declare module 的模块名，为 import 的文件名

## Flutter

### 动画

#### 动画结构

多重动画，区分不同的时间点

```dart
class HomeState extends State<Home> with SingleTickerProviderStateMixin {
  Animation<double> _animation;
  AnimationController _animationController;

  @override
  void initState() {
    super.initState();
    _animationController = new AnimationController(duration: Duration(seconds: 3), vsync: this);
    _animation = CurvedAnimation(parent: _animationController, curve: Curves.bounceIn); // 弹性曲线
    _animation = Tween(begin: 0.0, end: 300.0).animate(_animationController)
      ..addListener(() {
        setState(() => {});
      });
    _animationController.forward();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Center(
        child: Image(
          width: _animation.value,
          height: _animation.value,
          image: AssetImage('images/test.png'),
        ),
      ),
    );
  }
}
```

```dart
// 省略addListener，使用其他方式更新UI
class AnimateImage extends AnimatedWidget {
  AnimateImage({Key key, this.animation}) : super(key: key, listenable: animation);

  final Animation<double> animation;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Image(
        image: AssetImage('images/test.png'),
        width: animation.value,
        height: animation.value,
      ),
    );
  }
}

// 使用时
AnimateImage(animation: _animation);
```

##### AnimatedBuilder

```dart
AnimatedBuilder(
  animation: _animation,
  child: null, // 这里的child作为builder的child的参数传递
  builder: (BuildContext context, Widget child) {
    return Center(
      child: Image(
        width: _animation.value,
        height: _animation.value,
        image: AssetImage('images/test.png'),
      ),
    );
  },
);
```

##### 动画状态监听

```dart
addStatusListener(AnimationStatus status){
}
```

AnimationStatus:

- ismissed 动画在起始点停止
- forward 动画正在正向执行
- reverse 动画正在反向执行
- completed 动画在终点停止

#### 自定义路由切换动画（PageRouteBuilder）

```dart
Navigator.push(
    context,
    PageRouteBuilder(
        transitionDuration: Duration(seconds: 1),
        pageBuilder: (BuildContext context, Animation<double> animation,
            Animation<double> secondaryAnimation) {
          return FadeTransition(
            opacity: animation,
            child: List(),
          );
        }
    )
);
```

#### Hero

Hero 动画就是在路由切换时，有一个共享的 widget 可以在新旧路由间切换

**tag**必须对应

```dart
GestureDetector(
  child: Container(
    alignment: Alignment.center,
    child: Hero(
      tag: 'PERSON',
      child: Image.asset('images/test.png', width: 100, height: 100),
    ),
  ),
  onTap: () {
    Navigator.push(context, PageRouteBuilder(
        pageBuilder: (BuildContext context, Animation<double> animation, Animation<double> secondaryAnimation) {
          return FadeTransition(
            opacity: animation,
            child: List(),
          );
        }
    ));
  },
);

// List
Scaffold(
  appBar: AppBar(title: Text('List')),
  body: Center(
    child: Hero(
      tag: "PERSON",
      child: Image.asset('images/test.png'),
    ),
  ),
)
```

## 设计模式

### 观察者模式和发布订阅模式

#### 观察者模式

subject => observer

subject（主体）内维护了 observer（观察者），在触发某个事件时，通知这些 observer

```typescript
class Subject {
  private deps: Observer[] = [];

  add(ob: Observer): void {
    this.deps.push(ob);
  }

  notify() {
    this.deps.forEach((ob) => ob.update());
  }
}

class Observer {
  update() {}
}
```

1. subject 和 observer 强耦合，subject 对 observer 是知晓存在的，且内部维护了 observer 的依赖
2. 观察者模式大多数为同步
3. 需要在单个应用程序中实现

#### 发布订阅模式

publisher = event channel => subscriber

publisher 发布到的消息，不会直接到 subscriber，而是由 event channel

```typescript
const EVENT_TYPE = 'DATE_NOTICE';

class Publisher {
  private eventChannel: EventChannel;

  constructor(ec: EventChannel) {
    this.eventChannel = ec;
  }

  notify() {
    this.eventChannel.publish(EVENT_TYPE);
  }
}

class EventChannel {
  private deps: Map<string, Function[]> = new Map();

  publish(eventType: string, ...args) {
    if (this.deps.has(eventType)) {
      for (const handler of this.deps.get(eventType)) {
        handler(...args);
      }
    }
  }

  subscribe(eventType: string, handler: Function) {
    if (!this.deps.has(eventType)) {
      this.deps.set(eventType, []);
    }

    this.deps.get(eventType).push(handler);
  }
}

class Subscriber {
  update() {
    console.log('我订阅的消息发布咯');
  }
}

const eventChannel = new EventChannel(); // 此event channel 可以随意更换
const publisher = new Publisher(eventChannel);
const subscriber = new Subscriber();

// 订阅消息中新
eventChannel.subscribe(EVENT_TYPE, subscriber.update);

// 发布消息咯
publisher.notify();
```

1. publisher 和 subscriber 不知道对方的存在，只和 event channel 有关，**event channel 可以随意替换**
2. 发布订阅大多为异步，event channel 使用消息队列实现
3. 发布订阅可以交叉应用，譬如利用消息队列实现跨应用

## LeetcCode

### 等式方程的可满足性

### 每日温度

思路 1：暴力法之 [i,j] 循环
思路 2：暴力法之 next 数组，由于温度只是 30-100，所以用长度为 101 的数组存储，**从后往前**循环，以 next[arr[i]]=i 来记录，当遇到新温度时，则在 next 数组中去寻找大于 arr[i]+1 的值

```typescript
function dailyTemperatures(T: number[]): number[] {
  const next: number[] = new Array(101);

  for (let i = T.length - 1; i >= 0; i--) {
    const temp: number = T[i];
    let min = Number.MAX_SAFE_INTEGER;

    for (let j = temp + 1; j < 101; j++) {
      if (next[j] !== undefined && next[j] > i) {
        min = Math.min(min, next[j] - i);
      }
    }

    next[temp] = i;
    T[i] = min === Number.MAX_SAFE_INTEGER ? 0 : min;
  }

  return T;
}
```

思路 3：单调栈，以栈存储温度，当遇到比栈顶的温度大时，则开始出栈**直到栈顶温度大于当前温度或栈空了**

```typescript
class Stack {
  private store: number[] = [];

  head(): number | undefined {
    return this.store[this.store.length - 1];
  }

  push(v: number): void {
    this.store.push(v);
  }

  pop(): number | undefined {
    return this.store.pop();
  }

  isEmpty(): boolean {
    return this.store.length < 1;
  }
}

function dailyTemperatures(T: number[]): number[] {
  const stack: Stack = new Stack();
  const result: number[] = new Array(T.length).fill(0);

  for (let i = 0; i < T.length; i++) {
    let v: number;
    while (!stack.isEmpty() && T[stack.head()!] < T[i]) {
      v = stack.pop()!;
      result[v] = i - v;
    }

    stack.push(i);
  }

  return result;
}
```

### 三数之和

思路 1：暴力法 i,j,k 三重循环
思路 2：借鉴两数之和，排序后，固定一个数，然后寻找其余两数（剩下就可以用 双指针 或 hash 了）

```typescript
function threeSum(nums: number[]): number[][] {
  nums.sort((a, b) => a - b);
  const result: number[][] = [];

  let i = 0;

  // 如何确定 i 的 循环终止条件 1. 由于是三个数，i 肯定不能到末尾的，最大限度到 nums.length - 3 的位置； 2. 由于是三数之和等于0 ，排序后，如果 nums[i] > 0 三数之和绝不可能等于0
  while (i <= nums.length - 3 && nums[i] <= 0) {
    const t = 0 - nums[i];

    let left = i + 1;
    let right = nums.length - 1;

    while (left < right) {
      const _t = nums[left] + nums[right];
      if (_t === t) {
        result.push([nums[i], nums[left], nums[right]]);
        left = leftBiggerMove(left);
        right = rightSmallerMove(right);
      } else if (_t > t) {
        right = rightSmallerMove(right);
      } else {
        left = leftBiggerMove(left);
      }
    }

    i = leftBiggerMove(i);
  }

  return result;

  // 移动并去掉重复数
  function leftBiggerMove(v: number) {
    while (nums[v] === nums[++v]) {}
    return v;
  }
  function rightSmallerMove(v: number) {
    while (nums[v] === nums[--v]) {}
    return v;
  }
}
```
