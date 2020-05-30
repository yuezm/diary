# May-2

## Typescript

### 面向对象特性

1. 继承：子类继承父类，继承父类的属性和方法
2. 多态：子类继承父类，且重写父类方法，对于不同子类方法名相同而调用方法不同
3. 封装：类访问控制符，private、protected、public 控制类访问权限

### 泛型

在函数、interface、class 时无法确定数据类型，可以使用泛型。_泛型和函数传参类似，函数传参传值_，**泛型传参传类型**

### 类型断言

手动指出当前数据类型，存在两种方法

1. `<type>`：由于在 tsx 中，与标签容易混淆，所以不建议使用
2. `as type`

**typescript 的类型断言只能欺骗 typescript 编译器**，在代码运行时该报错还是报错

### 类型防护

在联合类型时，需要确定当前类型，以免发生错误，例如

```typescript
type T = string | number;

function (t: T): void{
  t.toFix(2); // string 是没有toFix方法的
}
```

1. `typeof t` 判断
2. `t instanceeof T` 判断
3. `hasOwnProperty` 判断
4. `in`判断

#### 类型谓词

对于某个类型进行判断 parameterName is Type

```typescript
interface T1 {
  age: string;
}
interface T2 {
  name: string;
}

type T = T1 | T2;

function isT1(t: T): t is T1 {
  return t.age !== undefined;
}
```

### typescript 3.7 新增操作符

1. ?. 可选链接：例如 `obj?.name` 表示当 obj 部位 null 或 undefined 时，取 name
2. ! 非空断言：不为 null
3. !.非空断言：`obj!.name` 表示 obj 不可能为 null，可以在 `document.getElementById、Object.getOwnPropertyDescriptor` 使用
4. ?? ：和 "||" 类似，但 ?? 只判断 null 和 undefined，而||判断所有为 false 的

### class 和 typeof class

class：类型声明空间、变量声明空间 typeof class：类型声明空间

对于类型来说是一致的，但 class 可以用作变量赋值

### typescript 3.9

1. 修复 Promise 问题，对于 Promise.all、Promise.race 推断问题
2. 速度改进，编译速度增加
3. @ts-expect-error 注释
4. 编译器提示改进

**重大变化：**

1. **foo?.bar!.baz ==> foo?.bar.baz**，_原来解释为 (foo?.bar)!.baz_
2. },>不允许在 tsx 使用，必须转义
3. 更严格的交叉类型推断
4. 更严格的联合类型检查
5. Getter 和 Setter 不在属于可枚举属性
6. 扩展 any 不在作为 any，例如`T extends any`，但 T 不再是 any 了
7. 导出结果提升与初始赋值

```typescript
export * from 'mod';
export const nameFromMod = 0;
```

解释为

```javascript
exports.nameFromMod = void 0;
exports.nameFromMod = 0;
```

## Flutter

### 静态资源管理

_由文件 pubspec.yaml 管理_

```yaml
// pubspec.yaml
flutter:
  assets:
    - images/a_dot_burr.jpeg # 以 pubspec.yaml 目录为路径
    - images/a_dot_ham.jpeg
```

#### 使用

```dart
// rootBundle 加载
import 'dart:async' show Future;
import 'package:flutter/services.dart' show rootBundle;

rootBundle.loadString('assets/config.json'); // string

DefaultAssetBundle.of(context)..loadString('images/test.png'); // string

// AssetImage 加载图片，且可以加载不同分辨率，但需要以特殊目录来保存asset
/*
asset/image.png
asset/2.0x/image.ong
asset/3.0x/image.png
*/

class BackgroundWidget extends StatelessWidget {
  @override
  Widget build(context) {
    return new DecoratedBox(
        decoration: new BoxDecoration(
            image: new DecorationImage(image: new AssetImage("images/test.png"))
        )
    );
  }
}

// new AssetImage("images/test.png") 为一个 ImageProvider，而非Widget，如果想用Widget，则可以使用
Image.asset('assets/config.json');


// 加载指定包的图片
new AssetImage('icons/heart.png', package: 'my_icons')
```

#### 平台资源

在如下路径下，修改各个平台的资源，例如 app 图片

**android**：_/android/app/src/main/res_

**iso**：_ios/Runner/Assets.xcassets/AppIcon.appiconset_

### 调试

#### flutter 分析工具

```bash
$ flutter analyze 静态分析代码

$ flutter run --profile 网页调试

$ flutter run --release 发布模式

```

```dart
debugger(); // 断点
print(); // 使用flutter logs查看，在android中，如果一次性输入过多，可能会丢失，此时可以使用 debugPrint();
```

#### 调用应用程序层

```dart
debugDumpApp(); // 转存储Widget
debugDumpRenderTree(); // 渲染树
debugDumpLayerTree(); // 渲染树是分层的，最终绘制是将各个层合并，layer输出的为需要合成的层
```

### 错误

```dart
// 全局错误处理
void main() {
  FlutterError.onError = (FlutterErrorDetails details) {
    reportError(details);
  };
 ...
}


// 同步方法错误捕获
try{
  //
}catch(){

}finally{

}

// 异步方法错误捕获，runZoned
runZoned(() => runApp(MyApp()), zoneSpecification: new ZoneSpecification(
      print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
        parent.print(zone, "Intercepted: $line");
      }),
  );
```

#### 完整错误处理

```dart
void collectLog(String line){
    ... //收集日志
}
void reportErrorAndLog(FlutterErrorDetails details){
    ... //上报错误和日志逻辑
}

FlutterErrorDetails makeDetails(Object obj, StackTrace stack){
    ...// 构建错误信息
}

void main() {
  FlutterError.onError = (FlutterErrorDetails details) {
    reportErrorAndLog(details);
  };
  runZoned(
    () => runApp(MyApp()),
    zoneSpecification: ZoneSpecification(
      print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
        collectLog(line); // 收集日志
      },
    ),
    onError: (Object obj, StackTrace stack) {
      var details = makeDetails(obj, stack);
      reportErrorAndLog(details);
    },
  );
}
```

### Widget

Widget：表示 UI 元素，功能性组件等。**Widget 只是描述一个元素（Element）的配置，而非正真在的元素**，一个 Widget 可以对应多个 Element，因为同一个 Widget 可以添加到 UI 树不同部分，而真正渲染时，每一个 Element 都会对应一个 Widget 对象

1. Widget 实际上就是 Element 的配置数据，Widget 树实际上是一个配置树，而真正的 UI 渲染树是由 Element 构成；不过，由于 Element 是通过 Widget 生成的，所以它们之间有对应关系，在大多数场景，我们可以宽泛地认为 Widget 树就是指 UI 控件树或 UI 渲染树。
2. 一个 Widget 对象可以对应多个 Element 对象。这很好理解，根据同一份配置（Widget），可以创建多个实例（Element）

```dart
@immutable
// DiagnosticableTree 为 诊断树，提供调试信息
abstract class Widget extends DiagnosticableTree {
  const Widget({ this.key });

  final Key key; // 是否复用，和 vue key 一致

  @protected
  Element createElement(); // 调用该方法，生成Element，该方法由Framework完成，开发人员一般不调用

  @override
  String toStringShort() {
    return key == null ? '$runtimeType' : '$runtimeType-$key';
  }

  @override
  // 诊断树信息
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  // 新老树更新，runtimeType一致，且key一致则可以更新组件，否则需要重新创建组件
  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
}
```

#### StatelessWidget

```dart
@override
StatelessElement createElement() => new StatelessElement(this); // 重写了createElement方法
```

1. widget 参数 应该标注出是否必传 @required，更好的静态检查
2. widget 第一个参数始终为 Key
3. StatelessWidget 属性 应该声明为 final，防止被修改

```dart
class APP extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'APP',
      home: new Home(),
    );
  }
}
```

#### StatefulWidget

```dart
  @override
  StatefulElement createElement() => new StatefulElement(this); // 重写了createElement方法

  @protected
  State createState(); //
```

##### State

1. State 为 StatefulWidget 维护的状态
2. State 状态 在构建时被同步读取，在生命周期内可以修改
3. State 发生改变，应该使用 setState() 通知 Flutter Framework 进行更新，Framework 会调用 build 方法重新构建 Widget，从而实现更新

4. widget，它表示与该 State 实例关联的 widget 实例，由 Flutter framework 动态设置。注意，这种关联并非永久的，因为在应用生命周期中，UI 树上的某一个节点的 widget 实例在重新构建时可能会变化，但 State 实例只会在第一次插入到树中时被创建，当在重新构建时，如果 widget 被修改了，Flutter framework 会动态设置 State.widget 为新的 widget 实例。
5. context。StatefulWidget 对应的 BuildContext，作用同 StatelessWidget 的 BuildContext

###### initState

初始化状态方法：当 Widget 第一次插入到 Widget 树时会被调用，对于每一个 State 对象，Flutter framework 只会调用一次该回调，所以，通常在该回调中做一些一次性的操作，如状态初始化、订阅子树的事件通知等

###### setState

修改状态

###### didChangeDependencies

当 State 对象的依赖发生变化时会被调用

###### build

在 initState()、didUpdateWidget()、setState()、didChangeDependencies() 后调用

###### reassemble

此回调是专门为了开发调试而提供的，在热重载(hot reload)时会被调用，此回调在 Release 模式下永远不会被调用

###### didUpdateWidget

在 widget 重新构建时，Flutter framework 会调用 Widget.canUpdate 来检测 Widget 树中同一位置的新旧节点，然后决定是否需要更新，如果 Widget.canUpdate 返回 true 则会调用此回调。典型的场景是当系统语言 Locale 或应用主题改变时，Flutter framework 会通知 widget 调用此回调

###### deactivate

当 State 对象从树中被移除时，会调用此回调。在一些场景下，Flutter framework 会将 State 对象重新插到树中，如包含此 State 对象的子树在树的一个位置移动到另一个位置时（可以通过 GlobalKey 来实现）。如果移除后没有重新插入到树中则紧接着会调用 dispose()方法

###### dispose

当 State 对象从树中被永久移除时调用；通常在此回调中释放资源。

#### context

context：BuildContext 实例，描述了当前 widget 在 widget 树中的上下文，是 widget 在 widget 树中，执行相关操作的句柄

#### Widget 获取 State

1. 通过 Context 获取

```dart
ScaffoldState _state = context.findAncestorStateOfType<ScaffoldState>(); // 查找父级最近的Scaffold对应的ScaffoldState对象
```

2. of 我们并不能在语法层面指定 StatefulWidget 的状态是否私有，所以在 Flutter 开发中便有了一个默认的约定：如果 StatefulWidget 的状态是希望暴露出的，应当在 StatefulWidget 中提供一个 of 静态方法来获取其 State 对象，开发者便可直接通过该方法来获取；如果 State 不希望暴露，则不提供 of 方法。这个约定在 Flutter SDK 里随处可见。所以，上面示例中的 Scaffold 也提供了一个 of 方法，我们其实是可以直接调用它的

```dart
 ScaffoldState ss = Scaffold.of(context);
```

3. GlobalKey

```
//定义一个globalKey, 由于GlobalKey要保持全局唯一性，我们使用静态变量存储
static GlobalKey<ScaffoldState> _globalKey= GlobalKey();
...
Scaffold(
    key: _globalKey , //设置key
    ...
)

// 使用 globalKey获取
_globalKey.currentState.openDrawer()
```

注意：使用 GlobalKey 开销较大，如果有其他可选方案，应尽量避免使用它。另外同一个 GlobalKey 在整个 widget 树中必须是唯一的，不能重复

#### 内置组件

flutter 基础组件，使用 Widget 时，必须先引入该包

```
import 'package:flutter/widgets.dart';
```

##### Material 组件

安卓组件

```
import 'package:flutter/material.dart';
```

##### Cupertino 组件

ios 组件

```
import 'package:flutter/cupertino.dart';
```

### 状态管理

#### Widget 管理自身状态

在 State 中，自己管理自己的状态

#### 父 Widget 管理子 Widget 状态

```dart

// 父级使用时

class HomeState extends State<Home> {
  int count = 0;
  bool active = false;

  void handleTap(bool act) {
    setState(() {
      active = !act;
    });
  }

  @override
  Widget build(BuildContext context) {
    return new TapBox(
      active: active,
      handleActiveChange: handleTap,
    );
  }
}

class TapBox extends StatelessWidget {
  TapBox({Key key, this.active, this.handleActiveChange}) : super(key: key);

  final bool active;
  final ValueChanged<bool> handleActiveChange; // ValueChanged函数类型 typedef ValueChanged<T> = void Function(T value);

  @override
  Widget build(BuildContext context) {
    return new GestureDetector( // GestureDetector 收拾Widget
      onTap: () {
        handleActiveChange(active);
      },
      child: new Container(
        child: new Text(
          'TapBox',
        ),
        width: 300,
        height: 300,
        decoration: new BoxDecoration(color: active ? Colors.red : Colors.black),
      ),
    );
  }
}
```

#### 混合管理

组织管理自身的状态，父组件管理其他外部状态

#### 全局状态

1. 全局事件总线
2. 专门的状态管理包

### 文本 Text

```dart
new Text(
  'Text' * 4,
  maxLines: 3, // 最大行数
  overflow: TextOverflow.ellipsis,
  textAlign: TextAlign.left, // 水平对齐
  textScaleFactor:
      0.5, // 缩放，代表文本相对于当前字体大小的缩放因子，相对于去设置文本的样式style属性的fontSize，它是调整字体大小的一个快捷方式。该属性的默认值可以通过MediaQueryData.textScaleFactor获得，如果没有MediaQuery，那么会默认值将为1.0
  style: new TextStyle(
      color: Colors.black,
      fontSize: 12,
      fontWeight: FontWeight.bold,
      fontStyle: FontStyle.italic,
  ),
),
```

#### TestSpan

```dart
const TextSpan({
  TextStyle style,
  Sting text,
  List<TextSpan> children,
  GestureRecognizer recognizer, // 手势处理
});

new Text.rich(
  new TextSpan(
    text: 'TestSpan',
  ),
)
```

#### DefaultTextStyle

如果再文本节点中使用该属性，则后续包含的文本节点都会继承该属性

```dart
new DefaultTextStyle(
  style: new TextStyle(
    color: Colors.red,
  ),
  child: new Column(
    children: <Widget>[
      new Text('Home1'),
      new Text('Home2'),
      new Text('Home3'),
    ],
  )
)
```

#### 外部字体

```yaml
// pubspec.yaml
flutter:
  fonts:
    - family: Raleway
      fonts:
        - asset: assets/fonts/Raleway-Regular.ttf
        - asset: assets/fonts/Raleway-Medium.ttf
          weight: 500
        - asset: assets/fonts/Raleway-SemiBold.ttf
          weight: 600
    - family: AbrilFatface
      fonts:
        - asset: assets/fonts/abrilfatface/AbrilFatface-Regular.ttf
        - asset: packages/my_package/fonts/Raleway-Medium.ttf
```

```dart
// main.dart
new TextStyle(
  fontFamily: 'AbrilFatface',
)
```

### 按钮

#### RaisedButton

漂浮按钮，它默认带有阴影和灰色背景。按下后，阴影会变大

#### FlatButton

即扁平按钮，默认背景透明并不带阴影。按下后，会有背景色

#### OutlineButton

默认有一个边框，不带阴影且背景透明。按下后，边框颜色会变亮、同时出现背景和阴影(较弱)

#### IconButton

是一个可点击的 Icon，不包括文字，默认没有背景，点击后会出现背景

**RaisedButton、FlatButton、OutlineButton 都有一个 icon 构造函数，通过它可以轻松创建带图标的按钮**

```dart
// 按钮通用属性
@required this.onPressed, //按钮点击回调
this.textColor, //按钮文字颜色
this.disabledTextColor, //按钮禁用时的文字颜色
this.color, //按钮背景颜色
this.disabledColor,//按钮禁用时的背景颜色
this.highlightColor, //按钮按下时的背景颜色
this.splashColor, //点击时，水波动画中水波的颜色
this.colorBrightness,//按钮主题，默认是浅色主题
this.padding, //按钮的填充
this.shape, //外形
@required this.child, //按钮的内容


new RaisedButton(child: new Text('Button'), onPressed: handleTap),
new FlatButton(child: new Text('Button'), onPressed: handleTap),
new OutlineButton(child: new Text('Button'), onPressed: handleTap),
new IconButton(icon: new Icon(Icons.add), onPressed: handleTap),

//Icon构造函数
new RaisedButton.icon(
    onPressed: handleTap,
    icon: new Icon(Icons.add),
    label: new Text('add')),
```

### 图片

ImageProvider：抽象类，调用 load() 从不同的源获取图片，需要实现不同的 ImageProvider，譬如 AssetImage 实现从 asset 加载的 ImageProvider，而 NetworkImage 实现从网络加载的 ImageProvider

#### Image

用于显示图片，Image 数据源可以为静态文件（AssetImage）、网络（NetworkImage）、内存等

```dart
new Image(
  image: new AssetImage('images/test.png'),
  width: 100, // 宽
  height: 100, // 高
  color: Colors.red, // 在图片绘制时可以对每一个像素进行颜色混合处理，color指定混合色，而colorBlendMode指定混合模式
  colorBlendMode: BlendMode.color,
  fit: BoxFit.fill, // 该属性用于在图片的显示空间和图片本身大小不同时指定图片的适应模式
    - fill: 拉伸，充满空间
    - cover：按比例放大填充空间，图片不会变形，超出被剪裁
    - contain：这是图片的默认适应规则，图片会在保证图片本身长宽比不变的情况下缩放以适应当前显示空间，图片不会变形
    - fitWidth：图片的宽度会缩放到显示空间的宽度，高度会按比例缩放，然后居中显示，图片不会变形，超出显示空间部分会被剪裁
    - fitHeight：图片的高度会缩放到显示空间的高度，宽度会按比例缩放，然后居中显示，图片不会变形，超出显示空间部分会被剪裁
    - none：：图片没有适应策略，会在显示空间内显示图片，如果图片比显示空间大，则显示空间只会显示图片中间部分
  alignment: Alignment.center, // 对齐规则
  repeat: ImageRepeat.noRepeat, // 重复规则
),
//Image.asset(asset);
//Image.network(src);
new Image(image: new NetworkImage('https://pcdn.flutterchina.club/imgs/3-17.png'));
```

框架对加载过的图片存在缓存，最大为 1000 张，最大缓存空间为 100MB

#### Icon

默认包含 [Material Design 的字体图标](https://material.io/tools/icons/)

```
// pubspec.yaml
flutter:
  uses-material-design: true
```

#### 自定义 icon

```yaml
// pubspec.yaml
flutter:
  fonts:
    - family: Schyler
      fonts:
        - asset: fonts/Schyler-Regular.ttf
        - asset: fonts/Schyler-Italic.ttf
          style: italic
```

```dart
// main.dart
class SelfIcon {
  static const IconData book = new IconData(
    'icon-name',
    fontFamily: 'Schyler',
    matchTextDirection: true,
  );

  static const wechat = new IconData(
    'icon-name',
    fontFamily: 'Schyler',
    matchTextDirection: true,
  );
}

// 使用
new Icon(SelfIcon.book, );
```

### Checkbox、Radio

Material 组件库中提供了 Material 风格的单选开关 Switch 和复选框 Checkbox，虽然它们都是继承自 StatefulWidget，但它们本身不会保存当前选中状态，选中状态都是由父组件来管理的。当 Switch 或 Checkbox 被点击时，会触发它们的 onChanged 回调，我们可以在此回调中处理选中状态改变逻辑

```dart
new Checkbox(value: active, onChanged: handleTap);
new Radio(value: count, groupValue: count, onChanged: handleTap);
```

### TextField、Form

#### TextField

```dart
const TextField({
  ...
  TextEditingController controller,  // 编辑框的控制器，通过它可以设置/获取编辑框的内容、选择编辑内容、监听编辑文本改变事件。大多数情况下我们都需要显式提供一个controller来与文本框交互。
  // 如果没有提供controller，则TextField内部会自动创建一个
  FocusNode focusNode, // 是否占有当前键盘输入焦点
  InputDecoration decoration = const InputDecoration(), // 外观显示
  TextInputType keyboardType, // 输入类型 text、multiline、number、phone、datetime、emailAddress、url
  TextInputAction textInputAction, // 键盘回车位图标
  TextStyle style, // 编辑文本格式
  TextAlign textAlign = TextAlign.start, // 水平对齐
  bool autofocus = false,
  bool obscureText = false, // 是否隐藏正在编辑的文本，如用于输入密码的场景等，文本内容会用“•”替换
  int maxLines = 1,
  int maxLength,
  bool maxLengthEnforced = true,
  ValueChanged<String> onChanged, // 输入框内容改变时的回调函数；注：内容改变事件也可以通过controller来监听
  VoidCallback onEditingComplete, // 这两个回调都是在输入框输入完成时触发，比如按了键盘的完成键（对号图标）或搜索键（🔍图标）。不同的是两个回调签名不同 onEditingComplete不接收参数
  ValueChanged<String> onSubmitted, // onSubmitted回调是ValueChanged<String>类型，它接收当前输入内容做为参数
  List<TextInputFormatter> inputFormatters, // 用于指定输入格式；当用户输入内容改变时，会根据指定的格式来校验
  bool enabled, // 可用、禁用
  this.cursorWidth = 2.0, // 这三个属性是用于自定义输入框光标宽度
  this.cursorRadius, // 光标圆角
  this.cursorColor, // 光标颜色
  ...
})
```

**onChanged 事件获取数据**

```dart
new TextField(
  decoration: new InputDecoration(
    labelText: '用户名',
    hintText: '请输入用户名、email',
    prefixIcon: new Icon(Icons.person),
  ),
  autofocus: true,
  onChanged: handleUsernameChange,
);
new TextField(
  decoration: new InputDecoration(
    labelText: '密码',
    hintText: '请输入密码',
    prefixIcon: new Icon(Icons.lock),
  ),
  obscureText: true,
  onChanged: handlePasswordChange,
);
```

**Controller 获取数据**

```dart
TextEditingController usernameController = new TextEditingController();

@override
void initState() {
  usernameController.addListener(() {
    print(usernameController.text);
  }); // 监测文本变化

  usernameController.text = username;// 默认值

  usernameController.selection = new TextSelection(baseOffset: 2, extentOffset: 3); // 选择文本

  super.initState();
}
```

1. 监听文本变化
2. 设置默认值
3. 选择文本
4. 控制焦点：FocusNode 和 FocusScopeNode 控制焦点
   - 监听焦点改变事件
5.

#### Form

```dart
Form({
  @required Widget child,
  bool autovalidate = false, // 是否自动校验输入内容
  WillPopCallback onWillPop, // 决定Form所在的路由是否可以直接返回（如点击返回按钮），该回调返回一个Future对象，如果Future的最终结果是false，则当前路由不会返回；如果为true，则会返回到上一个路由。此属性通常用于拦截返回按钮
  VoidCallback onChanged, // Form的任意一个子FormField内容发生变化时会触发此回调
})
```

Form 内必须是 FormField 类型（FormField 是一个抽象类），Flutter 提供一个 TextFormField Widget

```dart
const FormField({
  ...
  FormFieldSetter<T> onSaved, //保存回调
  FormFieldValidator<T>  validator, //验证回调
  T initialValue, //初始值
  bool autovalidate = false, //是否自动校验。
})
```

##### FormState

使用 Form.of() 或 GlobalKey 返回

1. FormState.validate()：调用此方法后，会调用 Form 子孙 FormField 的 validate 回调，如果有一个校验失败，则返回 false，所有校验失败项都会返回用户返回的错误提示。
2. FormState.save()：调用此方法后，会调用 Form 子孙 FormField 的 save 回调，用于保存表单内容
3. FormState.reset()：调用此方法后，会将子孙 FormField 的内容清空

```dart

GlobalKey _formGlobalKey = new GlobalKey <FormState>();

// 由于Form.of(context)是往父级寻找state，有时候Form存在于子集，所以有时无法考Form.of无法寻找正确的FormState，可以使用GlobalKey
void handleUserInfoComplete() {
  (_formGlobalKey.currentState as FormState).validate();
}

new Form(
  key: _formGlobalKey,
  child: new Column(
    children: <Widget>[
      new TextFormField(
        autofocus: true,
        decoration: new InputDecoration(
          labelText: '用户名',
          hintText: '请输入用户名',
          prefixIcon: new Icon(Icons.person),
        ),
        validator: (String v) {
          return v.length > 0 ? null : '用户名不为空';
        },
      ),
      new TextFormField(
        decoration: new InputDecoration(
          labelText: '密码',
          hintText: '请输入密码',
          prefixIcon: new Icon(Icons.lock),
        ),
        validator: (String v) {
          return v.length > 0 ? null : '密码不能为空';
        },
      ),
      Padding(
        padding: const EdgeInsets.only(top: 28.0),
        child: Row(
          children: <Widget>[
            Expanded(
              child: RaisedButton(
                padding: EdgeInsets.all(15.0),
                child: Text("登录"),
                color: Theme
                    .of(context)
                    .primaryColor,
                textColor: Colors.white,
                onPressed: handleUserInfoComplete,
              ),
            ),
          ],
        ),
      )
    ],
))
```

**如果想正确的使用 Form.of(context)获取**

```dart
// 使用build
Expanded(
  child: new Builder(builder: (context) =>
  new RaisedButton(onPressed: () {
    print(Form.of(context)); // 此时可以获取正确的FormState
  })),
),
```

### 安全

#### HTML 安全

HTML5 新的 API 、新标签，也存在许多安全问题

1. HTML 新标签的 XSS
2. iframe sandbox
3. 跨域资源共享的安全
4. PostMessage API
5. storage

#### a 标签

当 a 标签使用`target="_blank"`时，存在安全隐患

```html
<a href="http://www.baidu.com" target="_blank" />
```

1. 安全问题：在目标界面中，可以使用 window.opener 来获取原 window 对象。对齐进行操作，例如 `window.location.href = "xxx"` 来使原窗口跳转
2. 性能问题：使用\_blank 打开目标地址时，浏览器将使用同个进程控制（可以使用 Google Task Manger）查看

**解决**

```html
<a rel="noopener | noreferrer" href="http://www.baidu.com" target="_blank" />
```

## LeetCode

### 快速求幂

实现 pow(x,y) 的快速求幂

例如需要求：x^y

1. 以二进制理解：例如 x^9，

   ```text
     9 的二进制表示为 1001
     x^9 =>
     x^(1001) =>
     x^( 2^3 + 2^0) =>
     x^2^3 * x^2^0 由于 x^2可以求出，
   ```

2. 以二分法理解：例如 x^9

   ```text
   9 => 4 * 2 + 1
   x^9 =>
   x(4 * 2 + 1) =>
   (x^2)^4 * x
   ```

```typescript
// x ^ 1 ^ 2 ^ 2 ^ 2 ==> x ^ 8
// x ^ 1 ^ 2 ^ 2 ^ 2 * x ===> x ^ 9
// ((x ^ 1 ^ 2 ^ 2 * x) ^ 2 * x) ^ 2 ===> x ^ 11
function pow(x: number, k: number): number {
  let n = 1;
  const isNeg = k < 0;
  k = Math.abs(k);

  while (k > 0) {
    // 如果 k & 1 > 0时，则 x 为奇数
    if (k & 1) {
      n *= x; // 由于规律，当为k奇数时，底数乘积即为值
    }
    x *= x;

    k >>= 1;
  }

  return isNeg ? 1 / n : n;
}
```

[快速求幂](https://leetcode-cn.com/problems/powx-n/)

### 最小栈、最大栈

最小栈、最大栈设计

思路：双栈

1. 使用数组保存栈
2. 使用辅助栈保存最大、最小值

```typescript
class MinStack {
  // 普通栈使用数组、链表来存储；最大栈、最小栈使用双栈来处理
  stack: number[] = [];
  helpStack: number[] = [Infinity];

  push(v: number): void {
    this.stack.push(v);
    this.helpStack.push(Math.min(this.helpStack[this.helpStack.length - 1], v));
    // 最大栈，将此次比较换成Math.max，且将helpStack哨兵换成-Infinity
  }
  pop(): number {
    if (this.stack.length > 0) {
      this.helpStack.pop();
      return this.stack.pop();
    }
    return null;
  }
  getMin(): number {
    if (this.stack.length < 1) return null;
    return this.helpStack[this.helpStack.length - 1];
  }

  top() {
    return this.stack.length > 0 ? this.stack[this.stack.length - 1] : null;
  }
}
```

[最小栈](https://leetcode-cn.com/problems/min-stack/)

### 在无限数 1、2、3、4、5...中寻找第 n 个数

思路：确定区间

[1-9]：9  
[10-99]：90*2  
[100-999]：900*3  
...

确定区间后，可以使用 （n - 区间起始位置），定位到具体数字，在根据多余的遍历

```typescript
function findNthDigit(n: number): number {
  // 确定区间
  let r = 1; // 第一区间，取件费范围为1-9 => 9；第二区间10-99 => 90 * 2；第三区间 100-999 => 900*3
  let p = 9;
  let rangeSum = 0;

  while (true) {
    const s = r * p;
    if (n - s <= 0) break;
    n -= s;

    r++;
    p *= 10;
  }

  // 确定区间后，则确定区间起始位置
  const rangeStart = 10 ** (r - 1);
  const step = Math.ceil((n - r) / r); // 计算距离该数的长度，千万别忘了计算首位，所以需要减去 r
  let number = rangeStart + step;
  n -= step * r;

  return Number(number.toString()[n - 1]);
}
```

[第 n 个数字](https://leetcode-cn.com/problems/nth-digit/)

### 和为 k 的子数组

思路：动态规划

推导公式：以 dp[i]记录从第 i 位前的和，那么寻找存在 dp[i] - dp[j] == k ，则表示可以寻找到该子数组

```typescript
var subarraySum = function (nums: number[], k: number): number {
  if (nums.length < 1) return 0;

  const store = new Map<number, number>();
  store.set(0, 1);

  let count = 0;
  let prev = 0; // 记录第i位前的数字和

  for (let i = 1; i <= nums.length; i++) {
    prev += nums[i - 1];
    const last = prev - k;

    if (store.has(last)) {
      count += store.get(last);
    }

    store.has(prev) ? store.set(prev, store.get(prev) + 1) : store.set(prev, 1);
  }

  return count;
};
```

**注意点**：为什么表示 i 位以前的和，而不是包含 i，因为如果包含 i，则无法表示 0 位以前的数（无法用-1 来表示）

```typescript
```

### 股票算法

思路：动态规划

设股票数组为 nums，总共可以交易 k 次（限制条件 1），冷冻时间为 t（限制条件 2）

**状态转移**：设 2 个状态，持有（1），卖出（0）

```text
0 => 不购买 => 0
0 => 购买 => 1

1 => 不卖 => 1
1 => 卖出 => 0
```

根据状态扭转，推导出公式

```text
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + nums[i]);
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1-t][k-1][0] - nums[i]); // 由于t限制，所以需要减去t，由于k限制，买入只能k次
```

#### 题 1 k=1，t=0

k=1(只能交易一次),t=0(无限制)

```typescript
function gp(nums: number[]): number {
  /**
   * 推导公式
   * dp[i][1][0] = Math.max(dp[i-1][1][0], dp[i-1][1][1] + nums[i]);
   * dp[i][1][1] = Math.max(dp[i-1][1][1], dp[i-1][0][0] - nums[i]); 由于k=1，则 k-1 必然为 0 收益，则将公式简化为
   *
   * dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + nums[i]);
   * dp[i][1] = Math.max(dp[i-1][1], -nums[i]);
   *
   */
  const store: number[][] = [[0, -nums[0]]];

  for (let i = 1; i < nums.length; i++) {
    store[i] = [];
    store[i][0] = Math.max(store[i - 1][0], store[i - 1][1] + nums[i]);
    store[i][1] = Math.max(store[i - 1][1], -nums[i]);
  }
  return store[store.length - 1][0];
}

// 优化空间
function gp(nums: number[]): number {
  let prev0 = 0;
  let prev1 = -nums[0];
  for (let i = 1; i < nums.length; i++) {
    prev0 = Math.max(prev0, prev1 + nums[i]);
    prev1 = Math.max(prev1, -nums[i]);
  }
  return prev0;
}
```

#### 题 2 k=Infinity，t=1

k=Infinity ,t=1(卖出隔一天才能购买)

```typescript
function gp(nums: number[]): number {
  if (nums.length < 2) return 0;

  const store: number[][] = [
    [0, -nums[0]],
    [Math.max(0, nums[1] - nums[0]), Math.max(-nums[0], -nums[1])],
  ];

  for (let i = 2; i < nums.length; i++) {
    store[i] = [
      Math.max(store[i - 1][0], store[i - 1][1] + nums[i]),
      Math.max(store[i - 1][1], store[i - 2][0] - nums[i]),
    ];
  }
  return store[store.length - 1][0];
}

// 优化存储空间
function maxProfit(nums: number[]): number {
  let prev0 = 0; // 保存前一天卖出
  let prev1 = -nums[0]; // 保存前一天买进
  let prevPrev = 0; // 保存前两天卖出

  for (var i = 1; i < nums.length; i++) {
    let _p = prev0;
    prev0 = Math.max(prev0, prev1 + nums[i]);
    prev1 = Math.max(prev1, prevPrev - nums[i]);
    prevPrev = _p;
  }
  return prev0;
}
```

#### 题 3 k = n,t = m 通用解

```typescript
function maxProfit(prices: number[], k: number, t: number): number {
  const store: number[][][] = [];

  for (let i = 0; i < prices.length; i++) {
    store[i] = [];

    for (let j = 0; j < k; j++) {
      store[i][j] = [
        Math.max(getIndex(i - 1, j, 0), getIndex(i - 1, j, 1) + prices[i]),
        Math.max(
          getIndex(i - 1, j, 1),
          getIndex(i - 1 - t, j - 1, 0) - prices[i]
        ),
      ];
    }
  }

  return store[store.length - 1][k - 1][0];

  function getIndex(i: number, k: number, s: number): number {
    return store[i] && store[i][k] ? store[i][k][s] : s === 0 ? 0 : -Infinity;
  }
}
```
