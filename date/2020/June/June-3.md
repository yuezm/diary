# June-3

## Flutter

### 动画

#### 交织动画

1. 要创建交织动画，需要使用多个动画对象（Animation）。
2. 一个 AnimationController 控制所有的动画对象。
3. 给每一个动画对象指定时间间隔（Interval）CurvedAnimation 对象

```dart
class HomeState extends State<Home> with TickerProviderStateMixin {
  Animation<double> _sizeAnimation;
  Animation<Color> _colorAnimation;
  Animation<EdgeInsets> _paddingAnimation;

  AnimationController _crossAnimationController;

  @override
  void initState() {
    super.initState();

    _crossAnimationController = AnimationController(duration: Duration(seconds: 3), vsync: this);

    _sizeAnimation = Tween<double>(begin: 0.0, end: 300.0).animate(CurvedAnimation(
      parent: _crossAnimationController,
      curve: Interval(0.0, 0.6, curve: Curves.ease),
    ));

    _colorAnimation = ColorTween(begin: Colors.green, end: Colors.red).animate(CurvedAnimation(
      parent: _crossAnimationController,
      curve: Interval(0.0, 0.6, curve: Curves.ease),
    ));


    _paddingAnimation = Tween<EdgeInsets>(begin: EdgeInsets.only(left: 0.0), end: EdgeInsets.only(left: 100.0))
    .animate(
      CurvedAnimation(
          parent: _crossAnimationController,
          curve: Interval(0.6, 1.0, curve: Curves.ease),
      ),
    );

    _crossAnimationController.forward();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(title: Text('Animation')),
        body: Center(
          child: AnimatedBuilder(
            animation: _crossAnimationController,
            builder: (ctx, child) {
              return Container(
                padding: _paddingAnimation.value,
                width: _sizeAnimation.value,
                height: _sizeAnimation.value,
                color: _colorAnimation.value,
              );
            },
          ),
        ));
  }
}
```

#### AnimatedSwitcher 动画切换组件

```dart
const AnimatedSwitcher({
  Key key,
  this.child,
  @required this.duration, // 新child显示动画时长
  this.reverseDuration,// 旧child隐藏的动画时长
  this.switchInCurve = Curves.linear, // 新child显示的动画曲线
  this.switchOutCurve = Curves.linear,// 旧child隐藏的动画曲线
  this.transitionBuilder = AnimatedSwitcher.defaultTransitionBuilder, // 动画构建器
  this.layoutBuilder = AnimatedSwitcher.defaultLayoutBuilder, //布局构建器
})
```

```dart
Column(
  children: <Widget>[
    AnimatedSwitcher(
      duration: Duration(milliseconds: 1000),
      transitionBuilder: (Widget child, Animation<double> animation) {
        // return ScaleTransition(child: child, scale: animation); // 旋转
        return FadeTransition(child: child, opacity: animation); // 渐隐
      },
      child: Text('$count', key: ValueKey<int>(count),),
    ),
    RaisedButton(onPressed: () {
      setState(() {
        count++;
      });
    }, child: Text('点击加1'),)
  ],
)
```

**在 didUpdateWidget 回调中判断其新旧 child 是否发生变化，如果发生变化，则对旧 child 执行反向退场（reverse）动画，对新 child 执行正向（forward）入场动画即可**

#### ImplicitlyAnimatedWidget 自定义过度动画

##### Flutter 内置的过度组价

AnimatedPadding： 在 padding 发生变化时会执行过渡动画到新状态
AnimatedPositioned： 配合 Stack 一起使用，当定位状态发生变化时会执行过渡动画到新的状态。
AnimatedOpacity ：在透明度 opacity 发生变化时执行过渡动画到新状态
AnimatedAlign ：当 alignment 发生变化时会执行过渡动画到新的状态。
AnimatedContainer ：当 Container 属性发生变化时会执行过渡动画到新的状态。
AnimatedDefaultTextStyle： 当字体样式发生变化时，子组件中继承了该样式的文本组件会动态过渡到新样式。

### 自定义组件

#### 组合其他组件

// 实现颜色渐变，点击一下有涟漪的按钮

```dart
import 'package:flutter/material.dart';

class GradientButton extends StatelessWidget {
  GradientButton({
    this.width,
    this.height,
    this.borderRadius,
    @required this.onPress,
    @required this.child,
  });

  final double width;
  final double height;
  final BorderRadius borderRadius;
  final Widget child;
  final GestureTapCallback onPress;

  @override
  Widget build(BuildContext context) {
    ThemeData _themeData = Theme.of(context);
    List<Color> _colors = [
      _themeData.primaryColor,
      _themeData.primaryColorDark ?? _themeData.primaryColor
    ];

    return DecoratedBox( // 渐变
      decoration: BoxDecoration(
        gradient: LinearGradient(colors: _colors),
        borderRadius: borderRadius,
      ),
      child: Material(
        type: MaterialType.transparency, // Material + InkWell 实现点击涟漪
        child: InkWell(
          splashColor: _colors.last,
          highlightColor: Colors.transparent,
          borderRadius: borderRadius,
          onTap: onPress,
          child: ConstrainedBox(
            constraints: BoxConstraints.tightFor(height: height, width: width),
            child: Center(
              child: Padding(
                padding: EdgeInsets.all(10),
                child: DefaultTextStyle(
                  style: TextStyle(fontWeight: FontWeight.bold),
                  child: child,
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}

```

组合自旋转

```dart
import 'package:flutter/widgets.dart';

class TurnBox extends StatefulWidget {
  const TurnBox({
    Key key,
    this.turns = .0, //旋转的“圈”数,一圈为360度，如0.25圈即90度
    this.speed = 200, //过渡动画执行的总时长
    this.child
  }) :super(key: key);

  final double turns;
  final int speed;
  final Widget child;

  @override
  _TurnBoxState createState() => new _TurnBoxState();
}

class _TurnBoxState extends State<TurnBox>
    with SingleTickerProviderStateMixin {
  AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = new AnimationController(
        vsync: this,
        lowerBound: -double.infinity,
        upperBound: double.infinity
    );
    _controller.value = widget.turns;
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return RotationTransition(
      turns: _controller,
      child: widget.child,
    );
  }

  @override
  void didUpdateWidget(TurnBox oldWidget) {
    super.didUpdateWidget(oldWidget);
    //旋转角度发生变化时执行过渡动画
    if (oldWidget.turns != widget.turns) {
      _controller.animateTo(
        widget.turns,
        duration: Duration(milliseconds: widget.speed??200),
        curve: Curves.easeOut,
      );
    }
  }
}
```

#### 自绘

CustomPaint

```dart
CustomPaint({
  Key key,
  this.painter, // 背景画笔，会显示在子节点后面;
  this.foregroundPainter, // 前景画笔，会显示在子节点前面
  this.size = Size.zero, // 当child为null时，代表默认绘制区域大小，如果有child则忽略此参数，画布尺寸则为child尺寸。如果有child但是想指定画布为特定大小，可以使用SizeBox包裹CustomPaint实现
  this.isComplex = false, // 是否复杂的绘制，如果是，Flutter会应用一些缓存策略来减少重复渲染的开销
  this.willChange = false, // isComplex配合使用，当启用缓存时，该属性代表在下一帧中绘制是否会改变
  Widget child, //子节点，可以为空
})
```

如果 CustomPaint 有子节点，为了避免子节点不必要的重绘并提高性能，通常情况下都会将子节点包裹在 RepaintBoundary 组件中，这样会在绘制时就会创建一个新的绘制层（Layer），其子组件将在新的 Layer 上绘制，而父组件将在原来 Layer 上绘制，也就是说 RepaintBoundary 子组件的绘制将独立于父组件的绘制，RepaintBoundary 会隔离其子节点和 CustomPaint 本身的绘制边界

```dart
CustomPaint(
  size: Size(300, 300), //指定画布大小
  painter: MyPainter(),
  child: RepaintBoundary(child:...)),
)
```

##### CustomPainter

CustomPainter 中提定义了一个虚函数 paint

```dart
void paint(Canvas canvas, Size size);
// Canvas：一个画布，包括各种绘制方法，我们列出一下常用的方法：

Size：当前绘制区域大小
```

| API 名称   | 功能   |
| ---------- | ------ |
| drawLine   | 画线   |
| drawPoint  | 画点   |
| drawPath   | 画路径 |
| drawImage  | 画图像 |
| drawRect   | 画矩形 |
| drawCircle | 画圆   |
| drawOval   | 画椭圆 |
| drawArc    | 画圆弧 |

##### 画笔 Paint

```dart
var paint = Paint() //创建一个画笔并配置其属性
  ..isAntiAlias = true //是否抗锯齿
  ..style = PaintingStyle.fill //画笔样式：填充
  ..color=Color(0x77cdb175);//画笔颜色

```

**Flutter 提供的所有组件最终都是通过调用 Canvas 绘制出来的，只不过绘制的逻辑被封装起来了**！！！

**性能**，绘制是比较昂贵的操作，所以我们在实现自绘控件时应该考虑到性能开销，下面是两条关于性能优化的建议：

1. 尽可能的利用好 shouldRepaint 返回值；在 UI 树重新 build 时，控件在绘制前都会先调用该方法以确定是否有必要重绘；假如我们绘制的 UI 不依赖外部状态，那么就应该始终返回 false，因为外部状态改变导致重新 build 时不会影响我们的 UI 外观；如果绘制依赖外部状态，那么我们就应该在 shouldRepaint 中判断依赖的状态是否改变，如果已改变则应返回 true 来重绘，反之则应返回 false 不需要重绘。

2. 绘制尽可能多的分层；在上面五子棋的示例中，我们将棋盘和棋子的绘制放在了一起，这样会有一个问题：由于棋盘始终是不变的，用户每次落子时变的只是棋子，但是如果按照上面的代码来实现，每次绘制棋子时都要重新绘制一次棋盘，这是没必要的。优化的方法就是将棋盘单独抽为一个组件，并设置其 shouldRepaint 回调值为 false，然后将棋盘组件作为背景。然后将棋子的绘制放到另一个组件中，这样每次落子时只需要绘制棋子

### File I/O

Flutter I/O 依赖于 Dart I/O，但 Flutter 的目录和 Dart 目录不同，Dart VM 运行于 PC，Flutter 运行于手机

#### APP 目录问题

PathProvider 提供兼容

**临时目录**: 可以使用 getTemporaryDirectory() 来获取临时目录； 系统可随时清除的临时目录（缓存）。在 iOS 上，这对应于 NSTemporaryDirectory() 返回的值。在 Android 上，这是 getCacheDir())返回的值。
**文档目录**: 可以使用 getApplicationDocumentsDirectory()来获取应用程序的文档目录，该目录用于存储只有自己可以访问的文件。只有当应用程序被卸载时，系统才会清除该目录。在 iOS 上，这对应于 NSDocumentDirectory。在 Android 上，这是 AppData 目录。
**外部存储目录**：可以使用 getExternalStorageDirectory()来获取外部存储目录，如 SD 卡；由于 iOS 不支持外部目录，所以在 iOS 下调用该方法会抛出 UnsupportedError 异常，而在 Android 下结果是 android SDK 中 getExternalStorageDirectory 的返回值

```dart
// 引入 path_provider 包：path_provider: ^0.4.1
import 'package:path_provider/path_provider.dart';

getApplicationDocumentsDirectory().then((value) => {print(value)});
getTemporaryDirectory().then((value) => {print(value)});
getExternalStorageDirectory().then((value) => {print(value)}); // ios报错
```

### Network I/O

#### HTTP

```dart
HttpClient httpClient = new HttpClient();
HttpClientRequest request = await httpClient.getUrl(uri); // httpClient.post()、httpClient.get();

// 设置query参数
Uri uri = Uri(scheme: "https", host: "flutterchina.club", queryParameters: {
    "xx":"xx",
    "yy":"dd"
  });

// 设置请求头
request.headers.add("user-agent", "test");

// 设置请求主体
String payload="...";
request.add(utf8.encode(payload));
//request.addStream(_inputStream); //可以直接添加输入流

// 等待连接服务器
HttpClientResponse response = await request.close();

// 读取响应内容
String responseBody = await response.transform(utf8.decoder).join();

// 请求结束，关闭HttpClient
httpClient.close();
```

```dart
void httpRequestDemo() async {
  HttpClient _client = new HttpClient();
  HttpClientRequest _request =
      await _client.getUrl(Uri.parse('http://www.baidu.com'));
  HttpClientResponse _response = await _request.close();

  String res = await _response.transform(utf8.decoder).join();

  print(_response.headers);
  print(res);
}
```

dleTimeout：对应请求头中的 keep-alive 字段值，为了避免频繁建立连接，httpClient 在请求结束后会保持连接一段时间，超过这个阈值后才会关闭连接。
connectionTimeout： 和服务器建立连接的超时，如果超过这个值则会抛出 SocketException 异常。
maxConnectionsPerHost： 同一个 host，同时允许建立连接的最大数量。
autoUncompress： 对应请求头中的 Content-Encoding，如果设置为 true，则请求头中 Content-Encoding 的值为当前 HttpClient 支持的压缩算法列表，目前只有"gzip"
userAgent： 对应请求头中的 User-Agent 字段。

可以这在 HttpClient 上，也可以设置在 HttpClientRequest 上

#### 代理

```dart
client.findProxy = (uri) {
  // 如果需要过滤uri，可以手动判断
  return "PROXY 192.168.1.2:8888";
};
```

## LeetCode

### 最长公共前缀

思路 1： 从 0 开始计算，每次遍历一次数组

```javascript
function longestCommonPrefix(strs: string[]): string {
  if (strs.length < 1) return '';

  let i = 0;
  let result = '';

  while (i < strs[0].length) {
    const s = strs[0][i];

    for (let j = 1; j < strs.length; j++) {
      if (s !== strs[j][i]) {
        return result;
      }
    }

    result += s;
    i++;
  }

  return result;
}
```

思路 2：二分法

```javascript
function longestCommonPrefix(strs: string[]): string {
  if (strs.length < 1) return '';

  while (strs.length > 1) {
    const res: string[] = [];
    for (let i = 0; i < strs.length; i += 2) {
      res.push(diff(strs[i], strs[i + 1]));
    }
    strs = res;
  }
  return strs[0];
}

function diff(s1: string, s2: string | undefined) {
  if (s2 === undefined) return s1;
  let i = 0;
  while (i < s1.length && s1[i] === s2[i]) {
    i++;
  }
  return s1.slice(0, i);
}
```

### 二叉树的序列化

思路 1：BFS

```typescript
class TreeNode {
  val: number;
  left: TreeNode | null;
  right: TreeNode | null;

  constructor(val: number) {
    this.val = val;
    this.left = this.right = null;
  }
}

function serialize(root: TreeNode | null): string {
  if (root === null) return '';

  const queue: (TreeNode | null)[] = [root];
  let result = '';

  while (queue.length > 0) {
    const node = queue.pop();
    if (node === null) {
      result += ',null';
    } else {
      result += `,${node!.val}`;
      queue.unshift(node!.left);
      queue.unshift(node!.right);
    }
  }

  return result.slice(1);
}

function deserialize(data: string) {
  if (data === '') return null;
  const treeNodeList = data.split(',');

  const root = new TreeNode(Number(data[0]));
  const queue = [root];

  let i = 1;
  while (i < treeNodeList.length) {
    const v = treeNodeList[i];
    const parent = queue.pop()!;

    const leftVal = treeNodeList[i];
    const rightVal = treeNodeList[i + 1];

    if (leftVal !== 'null') {
      queue.unshift((parent.left = new TreeNode(Number(leftVal))));
    }

    if (rightVal !== 'null') {
      queue.unshift((parent.right = new TreeNode(Number(rightVal))));
    }

    i += 2;
  }

  return root;
}
```

思路 2：DFS

```typescript
```

### 最佳观光组合

思路 1：暴力法
思路 2：动态规划

设 max = nums[i] + nums[j] + i - j (i < j) ,由于在循环时 nums[i] + i 可以已知，则求出最大的 nums[j] - i 即可；所以在循环时，需要做两件事情

1. 找到 MAX_I = nums[i] + i 最大值，并记录起来
2. MAX_J = nums[j] - j 最大值，并每次与 MAX_I 做计算

```typescript
function maxScoreSightseeingPair(nums: number[]): number {
  let max = 0;
  let max_i = nums[0];

  for (let i = 1; i < nums.length; i++) {
    max = Math.max(max, nums[i] - i + max_i);
    max_i = Math.max(max_i, nums[i] + i);
  }
  return max;
}
```

### 先序遍历还原二叉树

```typescript
type DeepAndVal = [number, number, number]; // 值，深度，遍历的index

class TreeNode {
  val: number;
  left: TreeNode | null;
  right: TreeNode | null;
  constructor(val?: number, left?: TreeNode | null, right?: TreeNode | null) {
    this.val = val === undefined ? 0 : val;
    this.left = left === undefined ? null : left;
    this.right = right === undefined ? null : right;
  }
}

type DeepAndVal = [number, number, number]; // 值，深度，遍历的index

function recoverFromPreorder(str: string): TreeNode | null {
  if (str === '') return null;

  const [val, , ind] = findDeepAndVal(str, 0);
  const root = new TreeNode(val);
  const queue: TreeNode[] = [root];
  let i = ind;

  while (i < str.length) {
    let [val, deep, ind] = findDeepAndVal(str, i);
    const node = new TreeNode(val);

    if (deep === queue.length) {
      queue[queue.length - 1].left = node;
    } else {
      while (queue.length > deep) {
        queue.pop();
      }
      queue[queue.length - 1].right = node;
    }

    queue.push(node);
    i = ind;
  }
  return root;
}

function findDeepAndVal(str: string, i: number): DeepAndVal {
  let deep = 0;
  let val = '';

  while (str[i] === '-') {
    deep++;
    i++;
  }

  while (str[i] !== '-' && i < str.length) {
    val += str[i];
    i++;
  }

  return [Number(val), deep, i];
}
```
