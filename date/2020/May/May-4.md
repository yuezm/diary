# May-4

## Flutter

### DecoratedBox（装饰容器）

装饰，前景，背景，渐变等

```dart
const DecoratedBox({
  Decoration decoration,
  DecorationPosition position = DecorationPosition.background,
  Widget child
})

DecoratedBox(
  decoration: BoxDecoration(
    color: Colors.red,
  ),
  child: Padding(
    padding: EdgeInsets.all(16),
    child: Text('点我一下试试'),
  ),
)
```

#### BoxDecoration

Decoration 的子类

```dart
BoxDecoration({
  Color color, //颜色
  DecorationImage image,//图片
  BoxBorder border, //边框
  BorderRadiusGeometry borderRadius, //圆角
  List<BoxShadow> boxShadow, //阴影,可以指定多个
  Gradient gradient, //渐变
  BlendMode backgroundBlendMode, //背景混合模式
  BoxShape shape = BoxShape.rectangle, //形状
});
```

### Transform（变化）

```dart
const Transform({
    Key key,
    @required this.transform,
    this.origin,
    this.alignment, // Alignment.topRight
    this.transformHitTests = true,
    Widget child,
})
```

**Transform 的变换是应用在绘制阶段，而并不是应用在布局(layout)阶段，所以无论对子组件应用何种变化，其占用空间的大小和在屏幕上的位置都是固定不变的，因为这些是在布局阶段就确定的**，
由于矩阵变化只会作用在绘制阶段，所以在某些场景下，在 UI 需要变化时，可以直接通过矩阵变化来达到视觉上的 UI 改变，而不需要去重新触发 build 流程，这样会节省 layout 的开销，所以性能会比较好

#### translate

Transform.translate 平移

#### rotate

Transform.rotate 旋转

#### scale

Transform.scale 缩放

#### RotatedBox

和 Transform.rotate 功能相似，但 RotatedBox 变化是在 Layout 阶段，会影响后续子组件的计算

### Container

### 容器 Container

```dart
Container({
  this.alignment,
  this.padding, //容器内补白，属于decoration的装饰范围
  Color color, // 背景色
  Decoration decoration, // 背景装饰
  Decoration foregroundDecoration, //前景装饰
  double width,//容器的宽度
  double height, //容器的高度
  BoxConstraints constraints, //容器大小的限制条件
  this.margin,//容器外补白，不属于decoration的装饰范围
  this.transform, //变换
  this.child,
})
```

Container 是 DecorateBox、ConstrainedBox、Transform、Padding、Align 的综合体

1. 容器的大小可以通过 width、height 属性来指定，也可以通过 constraints 来指定；如果它们同时存在时，width、height 优先。
   实际上 Container 内部会根据 width、height 来生成一个 constraints。
2. color 和 decoration 是互斥的，如果同时设置它们则会报错！实际上，当指定 color 时，Container 内会自动创建一个 decoration

### Scaffold（路由骨架）、TabBar、底部导航

#### Scaffold

脚手架， Flutter Material 组件库提供的路由页面骨架，用户可以拼装界面

```dart
const Scaffold({
    Key key,
    this.appBar, // 导航栏
    this.body, // 主体
    this.floatingActionButton, // 浮动动作按钮
    this.floatingActionButtonLocation, // 悬浮位置，例如 FloatingActionButtonLocation.centerDocked, 悬浮于底部中间
    this.floatingActionButtonAnimator,
    this.persistentFooterButtons,
    this.drawer, // 左抽屉菜单
    this.endDrawer, // 右抽屉
    this.bottomNavigationBar, // 底部导航
    this.bottomSheet,
    this.backgroundColor,
    this.resizeToAvoidBottomPadding,
    this.resizeToAvoidBottomInset,
    this.primary = true,
    this.drawerDragStartBehavior = DragStartBehavior.start,
    this.extendBody = false,
    this.extendBodyBehindAppBar = false,
    this.drawerScrimColor,
    this.drawerEdgeDragWidth,
})
```

##### AppBar 顶部导航

##### AppBar

```dart
AppBar({
  Key key,
  this.leading, //导航栏最左侧Widget，常见为抽屉菜单按钮或返回按钮。
  this.automaticallyImplyLeading = true, //如果leading为null，是否自动实现默认的leading按钮
  this.title,// 页面标题
  this.actions, // 导航栏右侧菜单
  this.bottom, // 导航栏底部菜单，通常为Tab按钮组
  this.elevation = 4.0, // 导航栏阴影
  this.centerTitle, //标题是否居中
  this.backgroundColor,
})
```

顶部导航完整版

```dart
return Scaffold(
    appBar: AppBar(
      title: Text('Home'),
      actions: <Widget>[
        IconButton(icon: Icon(Icons.share), onPressed: null), // 右侧操作按钮
      ],
      leading: Builder(builder: (context) {
        return IconButton(
            icon: Icon(Icons.dashboard),
            onPressed: () {
              Scaffold.of(context).openDrawer(); // 打开抽屉
            });
      }),
      // 顶部导航按钮
      bottom: TabBar(
        tabs: tabs.map((e) => Tab(text: e)).toList(),
        controller: _tabController,
      ),
    ),
    // 顶部导航按钮状态改变时，body自动渲染
    body: TabBarView( // PageView 也可以实现相同功能
        controller: _tabController,
        children: tabs.map((e) {
          return Container(
            child: Text(e.toString()),
          );
        }).toList()));
}
```

##### body

其他的 Widget

##### bottomNavigationBar 底部导航

##### drawer 左右抽屉

### Clip（剪裁）

#### ClipOval 子组件为正方形时剪裁为内贴圆形，为矩形时，剪裁为内贴椭圆

#### ClipRRect 将子组件剪裁为圆角矩形

#### ClipRect 剪裁子组件到实际占用的矩形大小（溢出部分剪裁）

#### CustomClipper 剪裁特定区域

### Scrollable

```dart
Scrollable({
  ...
  this.axisDirection = AxisDirection.down,
  this.controller,
  this.physics,
  @required this.viewportBuilder, //后面介绍
})
```

1. axisDirection 滚动方向。
2. physics：此属性接受一个 ScrollPhysics 类型的对象，它决定可滚动组件如何响应用户操作，比如用户滑动完抬起手指后，继续执行动画；或者滑动到边界时，如何显示。默认情况下，Flutter 会根据具体平台分别使用不同的 ScrollPhysics 对象，应用不同的显示效果，如当滑动到边界时，继续拖动的话，在 iOS 上会出现弹性效果，而在 Android 上会出现微光效果。如果你想在所有平台下使用同一种效果，可以显式指定一个固定的 ScrollPhysics，Flutter SDK 中包含了两个 ScrollPhysics 的子类，他们可以直接使用：
   ClampingScrollPhysics：Android 下微光效果。
   BouncingScrollPhysics：iOS 下弹性效果。
3. controller：此属性接受一个 ScrollController 对象。ScrollController 的主要作用是控制滚动位置和监听滚动事件。默认情况下，Widget 树中会有一个默认的 PrimaryScrollController，如果子树中的可滚动组件没有显式的指定 controller，并且 primary 属性值为 true 时（默认就为 true），可滚动组件会使用这个默认的 PrimaryScrollController。这种机制带来的好处是父组件可以控制子树中可滚动组件的滚动行为，例如，Scaffold 正是使用这种机制在 iOS 中实现了点击导航栏回到顶部的功能。我们将在本章后面“滚动控制”一节详细介绍 ScrollController

#### Scrollbar

Material 风格的滚动指示器（滚动条）

#### CupertinoScrollbar

iOS 风格的滚动条,果你使用的是 Scrollbar，那么在 iOS 平台它会自动切换为 CupertinoScrollbar

#### Sliver

如果一个可滚动组件支持 Sliver 模型，那么该滚动可以将子组件分成好多个“薄片”（Sliver），只有当 Sliver 出现在视口中时才会去构建它，这种模型也称为“基于 Sliver 的延迟构建模型”。可滚动组件中有很多都支持基于 Sliver 的延迟构建模型，如 ListView、GridView，但是也有不支持该模型的，如 SingleChildScrollView

#### SingleChildScrollView

```dart
SingleChildScrollView({
  this.scrollDirection = Axis.vertical, //滚动方向，默认是垂直方向
  this.reverse = false,
  this.padding,
  bool primary,
  this.physics,
  this.controller,
  this.child,
})
```

1. reverse：该属性 API 文档解释是：是否按照阅读方向相反的方向滑动，如：scrollDirection 值为 Axis.horizontal，如果阅读方向是从左到右(取决于语言环境，阿拉伯语就是从右到左)。reverse 为 true 时，那么滑动方向就是从右往左。其实此属性本质上是决定可滚动组件的初始滚动位置是在“头”还是“尾”，取 false 时，初始滚动位置在“头”，反之则在“尾”，读者可以自己试验
2. primary：指是否使用 widget 树中默认的 PrimaryScrollController；当滑动方向为垂直方向（scrollDirection 值为 Axis.vertical）并且没有指定 controller 时，primary 默认为 true

_需要注意的是，通常 SingleChildScrollView 只应在期望的内容不会超过屏幕太多时使用，**这是因为 SingleChildScrollView 不支持基于 Sliver 的延迟实例化模型**，所以如果预计视口可能包含超出屏幕尺寸太多的内容时，那么使用 SingleChildScrollView 将会非常昂贵（性能差），此时应该使用一些支持 Sliver 延迟加载的可滚动组件，如 ListView_

```dart
Scrollbar(
  child: SingleChildScrollView(
    child: Container(
      width: 200,
      height: 3000,
      color: Colors.red,
    )
  )
)
```

#### ListView

```dart
ListView({
  ...
  //可滚动widget公共参数
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController controller,
  bool primary,
  ScrollPhysics physics,
  EdgeInsetsGeometry padding,

  //ListView各个构造函数的共同参数
  double itemExtent, // 该参数如果不为null，则会强制children的“长度”为itemExtent的值；这里的“长度”是指滚动方向上子组件的长度，也就是说如果滚动方向是垂直方向，则itemExtent代表子组件的高度；如果滚动方向为水平方向，则itemExtent就代表子组件的宽度。在ListView中，指定itemExtent比让子组件自己决定自身长度会更高效，这是因为指定itemExtent后，滚动系统可以提前知道列表的长度，而无需每次构建子组件时都去再计算一下，尤其是在滚动位置频繁变化时（滚动系统需要频繁去计算列表高度
  bool shrinkWrap = false, // shrinkWrap：该属性表示是否根据子组件的总长度来设置ListView的长度，默认值为false 。默认情况下，ListView的会在滚动方向尽可能多的占用空间。当ListView在一个无边界(滚动方向上)的容器中时，shrinkWrap必须为true
  bool addAutomaticKeepAlives = true, // 该属性表示是否将列表项（子组件）包裹在AutomaticKeepAlive 组件中；典型地，在一个懒加载列表中，如果将列表项包裹在AutomaticKeepAlive中，在该列表项滑出视口时它也不会被GC（垃圾回收），它会使用KeepAliveNotification来保存其状态。如果列表项自己维护其KeepAlive状态，那么此参数必须置为false
  bool addRepaintBoundaries = true, // 该属性表示是否将列表项（子组件）包裹在RepaintBoundary组件中。当可滚动组件滚动时，将列表项包裹在RepaintBoundary中可以避免列表项重绘，但是当列表项重绘的开销非常小（如一个颜色块，或者一个较短的文本）时，不添加RepaintBoundary反而会更高效。和addAutomaticKeepAlive一样，如果列表项自己维护其KeepAlive状态，那么此参数必须置为false
  double cacheExtent,

  //子widget列表
  List<Widget> children = const <Widget>[],
})
```

##### children 属性

适用于子集较少的情况

```dart
ListView(
  children: str.split('')
      .map((c) => Padding(padding: EdgeInsets.all(20), child: Center(child: Text('$c'),),))
      .toList(),
)
```

##### ListView.builder

适合列表项比较多（或者无限）的情况，因为只有当子组件真正显示的时候才会被创建，也就说通过该构造函数创建的 ListView 是支持基于 Sliver 的懒加载模型的

```
ListView.builder({
  // ListView公共参数已省略
  ...
  @required IndexedWidgetBuilder itemBuilder,
  int itemCount,
  ...
})
```

```
ListView.builder(
    itemCount: 24,
    itemExtent: 50,
    itemBuilder: (BuildContext context, int index) {
      return ListTile(title: ListTile(title: Text('${str[index]}')), subtitle: Text('$index'),);
    }
)
```

##### ListView.separated

可以在生成的列表项之间添加一个分割组件

```dart
ListView.separated(
  itemCount: 24,
  itemBuilder: (BuildContext context, int index) {
    return ListTile(title: ListTile(title: Text('${str[index]}')));
  },
  separatorBuilder: (BuildContext context, int index) {
    return divider;
  },
)
```

#### GridView

```dart
GridView({
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController controller,
  bool primary,
  ScrollPhysics physics,
  bool shrinkWrap = false,
  EdgeInsetsGeometry padding,
  @required SliverGridDelegate gridDelegate, //控制子widget layout的委托
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  double cacheExtent,
  List<Widget> children = const <Widget>[],
});
```

##### SliverGridDelegate

抽象类，由 SliverGridDelegateWithFixedCrossAxisCount（横轴最大子元素个数），SliverGridDelegateWithMaxCrossAxisExtent（横轴元素最大长度） 实现

```dart
SliverGridDelegateWithFixedCrossAxisCount({
  @required double crossAxisCount,  // 横轴子元素的数量。此属性值确定后子元素在横轴的长度就确定了，即ViewPort横轴长度除以crossAxisCount的商
  double mainAxisSpacing = 0.0, // 主轴方向的间距
  double crossAxisSpacing = 0.0, // 横轴方向子元素的间距
  double childAspectRatio = 1.0, // 子元素在横轴长度和主轴长度的比例。由于crossAxisCount指定后，子元素横轴长度就确定了，然后通过此参数值就可以确定子元素在主轴的长度
});

SliverGridDelegateWithMaxCrossAxisExtent({
  double maxCrossAxisExtent, // maxCrossAxisExtent为子元素在横轴上的最大长度，之所以是“最大”长度，是因为横轴方向每个子元素的长度仍然是等分的
  double mainAxisSpacing = 0.0,
  double crossAxisSpacing = 0.0,
  double childAspectRatio = 1.0,
})
```

```dart
GridView(
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 4,
  ),
  children: <Widget>[
    Text('home1'),
    Text('home2'),
    Text('home3'),
    Text('home4'),
    Text('home5'),
  ],
),
```

##### GridView.count

构造函数，快速实现 SliverGridDelegateWithFixedCrossAxisCount

##### GridView.extent

快速实现 SliverGridDelegateWithMaxCrossAxisExtent

```dart
GridView.extent(
  maxCrossAxisExtent: 200,
  children: <Widget>[
    Text('home1'),
    Text('home2'),
    Text('home3'),
    Text('home4'),
    Text('home5'),
  ],
);
```

```dart
GridView.count(
crossAxisCount: 4,
  children: <Widget>[
    Text('home1'),
    Text('home2'),
    Text('home3'),
    Text('home4'),
    Text('home5'),
  ],
)
```

##### GridView.builder

当子元素较多时，使用 GridView.builder 创建子元素

```dart
GridView.builder(
  ...
  @required SliverGridDelegate gridDelegate,
  @required IndexedWidgetBuilder itemBuilder
)
```

"flutter_staggered_grid_view"

#### CustomScrollView

做为胶水层，粘贴多个可滚动 Widget，每个 Widget 必须是 Sliver，例如 SliverList、SliverGrid、SliverPadding、SliverAppBar 等等

**Sliver 通常指可滚动组件子元素（就像一个个薄片一样）**！！！

上面之所以说“大多数”Sliver 都和可滚动组件对应，是由于还有一些如 SliverPadding、SliverAppBar 等是和可滚动组件无关的，它们主要是为了结合 CustomScrollView 一起使用，这是因为 CustomScrollView 的子组件必须都是 Sliver

```dart
CustomScrollView(
  slivers: <Widget>[
    SliverAppBar(
      title: Text('Home'),
    ),
    SliverPadding(
      padding: EdgeInsets.all(10),
      sliver: SliverGrid(
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(crossAxisCount: 4),
        delegate: SliverChildBuilderDelegate((BuildContext context, int index) {
          return Text('Grid');
        }, childCount: 100),
      ),
    ),
    SliverList(
      delegate: SliverChildBuilderDelegate((BuildContext context, int index) {
        return Center(child: Text('List'));
      }, childCount: 100),
    )
  ],
);
```

#### 滚动监听及控制

##### ScrollController

```dart
ScrollController({
  double initialScrollOffset = 0.0, //初始滚动位置
  this.keepScrollOffset = true,//是否保存滚动位置
  jumpTo(double offset), // 跳转不执行动画
  animateTo(double offset,...), // 跳转时执行动画
  ...
})
```

### 功能组件

#### 导航拦截

```dart
const WillPopScope({
  ...
  @required WillPopCallback onWillPop,
  @required Widget child
})
```

```dart
WillPopScope(
child: Scaffold(appBar: AppBar(title: Text('Back'))),
onWillPop: () async {
  if (_lastClickTime == null) {
    _lastClickTime = DateTime.now();
    return false;
  } else {
    if (DateTime.now().difference(_lastClickTime) >
        Duration(seconds: 1)) {
      return true;
    }
  }
})
```

#### 数据共享

InheritedWidget：它提供了一种数据在 widget 树中从上到下传递、共享的方式

##### didChangeDependencies

在“依赖”发生变化时被 Flutter Framework 调用。而这个“依赖”指的就是子 widget 是否使用了父 widget 中 InheritedWidget 的数据

```dart

```

##### 跨组件状态共享

Provider: InheritedWidget 实现
Redux
MobX
BLoc

#### Color

```dart
Color(0xff000000);
Color(int.parse('ff000000', radix: 16));

// computeLuminance 计算颜色深浅[0,1]，越深数字越大
Colors.red.computeLuminance();
```

##### MaterialColor

MaterialColor 是实现 Material Design 中的颜色的类，它包含一种颜色的 10 个级别的渐变色。
MaterialColor 通过"[]"运算符的索引值来代表颜色的深度，有效的索引有：50，100，200，…，900，数字越大，颜色越深

```dart
Colors.red[900];
```

#### Theme

##### ThemeData

子组件，使用 Theme.of 获取 ThemeData

```dart
ThemeData({
  Brightness brightness, //深色还是浅色
  MaterialColor primarySwatch, //主题颜色样本，见下面介绍
  Color primaryColor, //主色，决定导航栏颜色
  Color accentColor, //次级色，决定大多数Widget的颜色，如进度条、开关等。
  Color cardColor, //卡片颜色
  Color dividerColor, //分割线颜色
  ButtonThemeData buttonTheme, //按钮主题
  Color cursorColor, //输入框光标颜色
  Color dialogBackgroundColor,//对话框背景颜色
  String fontFamily, //文字字体
  TextTheme textTheme,// 字体主题，包括标题、body等文字样式
  IconThemeData iconTheme, // Icon的默认样式
  TargetPlatform platform, //指定平台，应用特定平台控件风格
  ...
})
```

1. 可以通过局部主题覆盖全局主题
2. context.inheritFromWidgetOfExactType 会在 widget 树中从当前位置向上查找第一个类型为\_InheritedTheme 的 widget。
   所以当局部指定 Theme 后，其子树中通过 Theme.of()向上查找到的第一个\_InheritedTheme 便是我们指定的 Theme
3. 对整个应用换肤，则可以去修改 MaterialApp 的 theme 属性

#### 异步 UI 更新

##### FutureBuilder

```dart
FutureBuilder({
  this.future, // FutureBuilder依赖的Future，通常是一个异步耗时任务
  this.initialData, // 初始数据
  @required this.builder, // Function (BuildContext context, AsyncSnapshot snapshot)
  // snapshot会包含当前异步任务的状态信息及结果信息 ，比如我们可以通过snapshot.connectionState获取异步任务的状态信息、
  // 通过snapshot.hasError判断异步任务是否有错误等等，完整的定义读者可以查看AsyncSnapshot类定义
})
```

```dart
Future<String> mockNetwork() {
    return Future.delayed(Duration(seconds: 2), () => '数据');
}

FutureBuilder(
  future: mockNetwork(),
  builder: (BuildContext context, AsyncSnapshot snapshot) {
    if(snapshot.connectionState == ConnectionState.done){
      return Text('has get：${snapshot.data}');
    }else{
      return Text('正在获取');
    }
  },
)

// ConnectionState.none
// ConnectionState.waiting // 异步任务处于等待状态
// ConnectionState.active // Stream处于激活状态（流上已经有数据传递了），对于FutureBuilder没有该状态。
// ConnectionState.done
```

##### StreamBuilder

```dart
StreamBuilder({
  Key key,
  this.initialData,
  Stream<T> stream,
  @required this.builder,
})
```

```dart
Stream<int> mockStreamData() {
  return Stream.periodic(Duration(seconds: 2), (int i) {
    return ++i;
  });
}

StreamBuilder(
  stream: mockStreamData(),
  builder: (BuildContext context, AsyncSnapshot snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return Text('正在获取');
    } else if (snapshot.connectionState == ConnectionState.active) {
      return Text('has get：${snapshot.data}');
    } else if (snapshot.connectionState == ConnectionState.done) {
      return Text('stream结束');
    }
  },
)
```

#### 对话框

##### Dialog

实际上 AlertDialog 和 SimpleDialog 都使用了 Dialog 类。由于 AlertDialog 和 SimpleDialog 中使用了 IntrinsicWidth 来尝试通过子组件的实际尺寸来调整自身尺寸，
这就导致他们的子组件不能是延迟加载模型的组件（如 ListView、GridView 、 CustomScrollView 等）

**如果使用延迟加载，则可以如下使用**：

```dart
Dialog(
  child: ListView(
    children: <Widget>[],
  ),
);
```

##### SimpleDialog

```dart
Future<bool> showBooleanDialog() {
  return showDialog(
      context: context,
      builder: (context) {
        return SimpleDialog(
          title: Text('请选择语言'),
          children: <Widget>[
            SimpleDialogOption(
              child: Text('中文简体'),
              onPressed: null,
            ),
            SimpleDialogOption(
              child: Text('美式英语'),
              onPressed: null,
            )
          ],
        );
      });
}
```

##### AlertDialog

```dart
const AlertDialog({
  Key key,
  this.title, //对话框标题组件
  this.titlePadding, // 标题填充
  this.titleTextStyle, //标题文本样式
  this.content, // 对话框内容组件
  this.contentPadding = const EdgeInsets.fromLTRB(24.0, 20.0, 24.0, 24.0), //内容的填充
  this.contentTextStyle,// 内容文本样式
  this.actions, // 对话框操作按钮组
  this.backgroundColor, // 对话框背景色
  this.elevation,// 对话框的阴影
  this.semanticLabel, //对话框语义化标签(用于读屏软件)
  this.shape, // 对话框外形
})
```

```dart
Future<bool> showBooleanDialog() {
  return showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: Text('提示'),
          content: Text('确定要删除该项吗？'),
          actions: <Widget>[
            FlatButton(
                onPressed: () {
                  print('I am cancel');
                  Navigator.of(context).pop();
                },
                child: Text('取消', style: TextStyle(color: Colors.red))),
            FlatButton(
                onPressed: () {
                  print('I am confirm');
                  Navigator.of(context).pop(true);
                },
                child: Text('确定', style: TextStyle(color: Colors.blue)))
          ],
        );
      });
}
```

##### showDialog

```dart
Future<T> showDialog<T>({
  @required BuildContext context,
  bool barrierDismissible = true, //点击对话框barrier(遮罩)时是否关闭它
  WidgetBuilder builder, // 对话框UI的builder
})
```

##### showGeneralDialog

动画打开对话框

```dart
Future<T> showGeneralDialog<T>({
  @required BuildContext context,
  @required RoutePageBuilder pageBuilder, //构建对话框内部UI
  bool barrierDismissible, //点击遮罩是否关闭对话框
  String barrierLabel, // 语义化标签(用于读屏软件)
  Color barrierColor, // 遮罩颜色
  Duration transitionDuration, // 对话框打开/关闭的动画时长
  RouteTransitionsBuilder transitionBuilder, // 对话框打开/关闭的动画
})
```

##### 对话框状态管理

## Leetcode

### LRU 缓存机制

思路 1：Map + Array，以 Map 存储 Key=>Node，以 Array 存储 Key
思路 2：Map + 双向链表，双向链表插入时移动效率比 Array 高

```typescript
class LinkNode {
  key: number;
  val: number;
  prev: LinkNode | null = null;
  next: LinkNode | null = null;

  constructor(key: number, val: number) {
    this.val = val;
    this.key = key;
  }
}

class LRUCache {
  private cacheNodeMap: Map<number, LinkNode> = new Map();
  // private cacheList: number[] = []; // 以数组存储节点
  private header: LinkNode = new LinkNode(-1, -1);
  private footer: LinkNode = new LinkNode(-1, -1);
  private linkLen: number = 0;
  private readonly capacity: number;

  constructor(capacity: number) {
    this.capacity = capacity;
    this.header.next = this.footer;
  }

  updateLinkNode(node: LinkNode, header: LinkNode) {
    // 如果node存在，则删除掉node
    if (node.prev) {
      node.prev!.next = node.next;
      node.next!.prev = node.prev;
    }

    // 并插入头节点
    node.next = header.next;
    node.next!.prev = node;

    header.next = node;
    node.prev = header;
  }

  get(key: number): number {
    // get时候，需要移动key到链表头指针
    if (this.cacheNodeMap.has(key)) {
      const node = this.cacheNodeMap.get(key)!;
      this.updateLinkNode(node, this.header);
      return node.val;
    }
    return -1;
  }

  put(key: number, value: number): void {
    if (this.cacheNodeMap.has(key)) {
      const node = this.cacheNodeMap.get(key)!;
      this.updateLinkNode(node, this.header);
      node.val = value;
    } else {
      this.linkLen++;
      const node = new LinkNode(key, value);
      this.cacheNodeMap.set(key, node);
      this.updateLinkNode(node, this.header);

      if (this.linkLen > this.capacity) {
        // 删除缓存key
        this.cacheNodeMap.delete(this.footer.prev!.key);

        // 删除link中的末尾节点
        this.footer.prev!.prev!.next = this.footer;
        this.footer.prev = this.footer.prev!.prev;
        this.linkLen--;
      }
    }
  }
}
```

### 寻找重复数

1. 二分法：例如在[x,y] 中出现了一个重复数（k），设 n = (x + y) / 2，则 [x,y]之间数**小于等于**n 的个数 如果 > n，则 k 必然在 n 的左边，否则在 n 的右边

   举个栗子：[1,2,3,4,5] 正常下 n = 3；那么正常情况下，数组**小于等于 3**的个数应该就是 3，因为（3-1+1 === 3），但此时如果出现了重复数 2，那么比 3 小的数的个数就会增多，以此来进行二分查找

   ```typescript
   function findDuplicate(nums: number[]): number {
     let left = 1;
     let right = nums.length;

     while (left < right) {
       // 开始二分
       const mid = (left + right) >> 1;
       let count = 0;

       for (const num of nums) {
         if (num <= mid) count++;
       }

       if (count > mid) {
         right = mid;
       } else {
         left = mid + 1;
       }
     }
     return left;
   }
   ```

2. 链表环算法：把数组看做链表，数组存储的值为**指向下一个值的下标**

   ```typescript
   function findDuplicate(nums: number[]): number {
     let slowPoint = nums[nums[0]]; // 移动一次
     let fastPoint = nums[slowPoint]; // 移动两次

     // 计算是否第一次相遇
     while (slowPoint !== undefined && fastPoint !== undefined) {
       if (slowPoint === fastPoint) {
         break;
       }

       slowPoint = nums[slowPoint];
       fastPoint = nums[nums[fastPoint]];
     }

     // 计算第二次相遇
     slowPoint = nums[0];

     while (slowPoint !== fastPoint) {
       slowPoint = nums[slowPoint];
       fastPoint = nums[fastPoint];
     }

     return slowPoint;
   }
   ```

[寻找重复数](https://leetcode-cn.com/problems/find-the-duplicate-number/submissions/)

### 链表环算法

确定单链表是否有环，环的入口在哪，环的长度时多少

1. hashMap 存储法：将遍历过的单链表存储起来，如果出现重复了，则存在环

   ```typescript
   class SingleLinkNode {
     val: any;
     next: SingleLinkNode | null;
   }

   type IsCircleResponse = { isCircle: boolean; circleLength: number };

   function findIsCircle(root: SingleLinkNode): IsCircleResponse {
     // hash法
     const map: Map<SingleLinkNode, any> = new Map();
     let node = root;
     let circleLength = 0;

     while (node !== null) {
       if (map.has(node)) {
         return {
           isCircle: true,
           circleLength: circleLength - map.get(node),
         };
       }
       map.set(node, count);
       count++;
       node = node.next;
     }

     return {
       isCircle: false,
       circleLength,
     };
   }
   ```

2. 双指针法（快慢指针）：快指针一步走两个，慢指针一步走一个，如果快慢指针相遇了，_当快慢指针**首次相遇了**，以慢指针指向首部，快指针指向相遇处，**第二次相遇即为环的入口**_

   ```typescript
   import { on } from 'cluster';

   class SingleLinkNode {
     val: any;
     next: SingleLinkNode | null;
   }

   type IsCircleResponse = { isCircle: boolean; circleLength: number };

   function findIsCircle(root: SingleLinkNode): IsCircleResponse {
     // 首先移动一个、2个位置，免得在链表头指针直接相等了
     let slowPoint = root.next;
     let fastPoint = root.next?.next;

     let isCircle = false;
     let circleLength = 0;

     // 第一次循环，得去发现
     while (slowPoint !== null && fastPoint !== null) {
       if (slowPoint === fastPoint) {
         // 此时存在环
         isCircle = true;
         break;
       }

       slowPoint = slowPoint.next;
       fastPoint = fastPoint.next?.next;
     }

     // 如果第一次循环强制中断，则存在环，否则不存在环
     if (isCircle) {
       slowPoint = root;

       while (slowPoint !== fastPoint) {
         slowPoint = slowPoint.next;
         fastPoint = fastPoint.next;
       }

       // 再次相遇时，此时发现环入口，开始计算环长度
       slowPoint = slowPoint.next;
       circleLength++;

       while (slowPoint !== fastPoint) {
         circleLength++;
         slowPoint = slowPoint.next;
       }
     }

     return {
       isCircle,
       circleLength,
     };
   }
   ```

### 可被 k 整除的子数组

思路 1：暴力法，遍历第 [i,j]之间的数，判断是否可以被 k 整除
思路 2：动态规划

1. 以 dp[i]表示第 i 个前所有元素和
2. 那么当存在(dp[j] - dp[i]) % k === 0 时，则表示存在子数组可以被 k 整除
3. 对余除进行优化，使用 nums 数组，且 nums 长度为 k 来表示除以 k 的余数（例如 nums[1]表示除以 k 余 1 的数），如果此时设 m = (dp[i] % k)，那么**判断 nums[k]是否存在则可以判断 dp[i]是否可以祖晨整除 k 的子数组**

```typescript
function subArraysDivByK(nums: number[], k: number): number {
  const lastNumArray: number[] = new Array<number>(k).fill(0); // 用于存储dp[i] % k 的余数
  lastNumArray[0] = 1; // 注意第0位应该是1，因为第0位的前和就是0

  let preSum = 0;
  let count = 0;
  for (let i = 1; i <= nums.length; i++) {
    preSum += nums[i - 1];
    const last = preSum % k;
    count += lastNumArray[last];
    lastNumArray[last]++;
  }

  return count;
}
```

[可被 k 整除的子数组](https://leetcode-cn.com/problems/subarray-sums-divisible-by-k/)

### 字符串编码

思路：该题主要难度在于**确定括号的闭合位置**，例如 `a[2c[2]]`，对于这种括号的有序结构，直接采用栈

```typescript
// 3[a]2[bc]

function decodeString(s: string): string {
  const numberStack: number[] = [];
  const strStack: string[] = [];

  let result = '';

  let i = 0; // 下标计数

  let m = 0; // 暂存s[i]转换为数字
  let str = ''; // 暂存所有s[i]的联合字符串
  let n = 0; // 暂存所有s[i]的联合数字

  while (i < s.length) {
    // 在进行循环时，如果 numberStack 长度为0时，则表示为单字符串，否则进行乘法运算
    if (s[i] === '[') {
      if (numberStack.length < 1) {
        result += str;
      } else {
        strStack.push(str);
      }
      numberStack.push(n);

      n = 0;
      str = '';
    } else if (s[i] === ']') {
      // 开始出栈时，需要注意，如果当前栈长度大于1，则表示肯定是多层嵌套，需要注意嵌套
      str = str.repeat(numberStack.pop()!);

      if (numberStack.length > 0) {
        str = strStack.pop() + str;
      } else {
        result += str;
        str = '';
      }
    } else {
      if (isNaN((m = Number(s[i])))) {
        // 为字符串
        str += s[i];
      } else {
        // 为数字
        n = n * 10 + m;
      }
    }
    i++;
  }

  return result + str;
}
```

[字符串编码](https://leetcode-cn.com/problems/decode-string/)
