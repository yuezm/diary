# May-3

## Javascript

### bind 怪异函数

```javascript
bind(thisArg[, arg1[, arg2[, ...]]]);
```

bind 调用后函数为 _Bound Function（BF）_，该函数为怪异函数，函数包含如下对象

- [[ TargetFunction ]]：包装函数、理解为原函数
- [[ BoundThis ]]：thisArgs 传值
- [[ BoundArguments ]]：args 传参
- [[ Call ]]：Bound Function 调用

**普通调用时**：

调用 [[ TargetFunction ]] 的 [[ Call ]]，并传入[[ BoundThis ]] 和 [[ BoundArguments ]]，类似于 `Call(BoundThis,BoundArguments)`

**构造调用时**：

## Typescript

### typescript 全局声明报错

```typescript
const name: string = 1; // 此时是会报错的
```

### let name: string 报错

typescript 对文件进行区分

1. 文件被声明为模块，存在自身的作用域
2. 文件被声明为脚本，与其他脚本公用全局作用域

而以上代码报错的原因

1. typescript 认为该文件未使用 export，则认为声明为**脚本文件**
2. window 对象本身存在 name 属性，则与之冲突，_其他 window 属性也是一样的道理_

```typescript
let name: string;

export default {}; // 让typescript明白，这个是一个模块
```

### interface 扩展、合并

```typescript
interface Test {}

interface Test {} // 扩展 Test
```

例如用来扩展 String 方法

```typescript
interface String {
  toBig(): void;
}
String.prototype.toBig = function () {
  //
};

const s: string = '';
s.toBig();
```

### 函数的参数（结构参数类型）

函数的参数类型声明

```typescript
function fn(x: number, y: number): number {
  return x + y;
}

// 那结构赋值如何声明参数

function fn({ x, y }): number {
  return x + y;
}

// 使用如下声明

function fn({ x = 1, y = 2 }: { x?: number; y?: number } = {}): number {
  return x + y;
}
```

### exclude file 失效

exclude file 不是万能的，exclude file 无法排除如下文件

1. 被未排除的文件 `import` 引入的
2. 被未排除的文件 `///` 引入的

## Flutter

### 表单

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

#### 表单状态管理（保存、重置、校验）

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

**如果想正确的使用 Form.of(context)获取**：

```dart
// 使用build
Expanded(
  child: new Builder(builder: (context) =>
  new RaisedButton(onPressed: () {
    print(Form.of(context)); // 此时可以获取正确的FormState
  })),
),
```

#### 输入框

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

##### 输入框文本 onchange 监听

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

##### 输入框文本 Controller 监听

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

**优势**：

1. 监听文本变化
2. 设置默认值
3. 设置选择文本
4. 控制焦点：FocusNode 和 FocusScopeNode 控制焦点
   - 监听焦点改变事件

### 进度指示器

#### LinearProgressIndicator 条形进度执指示器

```dart
LinearProgressIndicator({
  double value, // 如果 value 为null，则播放循环动画，而非准确的进度展示
  Color backgroundColor,
  Animation<Color> valueColor,
  ...
})
```

#### CircularProgressIndicator 圆形进度指示器

```dart
 CircularProgressIndicator({
  double value,
  Color backgroundColor,
  Animation<Color> valueColor,
  this.strokeWidth = 4.0, // 圆圈粗细
  ...
})
```

#### SizedBox 尺寸自定义

```dart
// 线高度2
new SizedBox(
  height: 2,
  child: new LinearProgressIndicator(
      value: .2,
  ),
)

// 直径100
new SizedBox(
  width: 100,
  height: 100,
  child: new CircularProgressIndicator(
  ),
)
```

### Widget 按照子元素个数分类

1. LeafRenderObjectWidget => LeafRenderObjectElement：无子 Widget ，例如 Image
2. SingleChildRenderObjectWidget => SingleChildRenderObjectElement，存在一个子 Widget，例如 Button
3. MultiChildRenderObjectWidget => MultiChildRenderObjectElement：存在多个子 Widget，例如 Row、Column

Widget <= RenderObjectWidget <= (Leaf/SingleChild/MultiChild)RenderObjectWidget

RenderObjectWidget 类中定义了创建、更新 RenderObject 的方法，子类必须实现他们，关于 **RenderObject 我们现在只需要知道它是最终布局、渲染 UI 界面的对象**即可，也就是说，对于布局类组件来说，其布局算法都是通过对应的 RenderObject 对象来实现的

### 布局

#### 线性布局

##### Row

水平排列

```dart
Row({
  ...
  TextDirection textDirection, // 文本对齐
  MainAxisSize mainAxisSize = MainAxisSize.max, // 主轴（水平）占用空间
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start, // 子组件在主轴（水平）对齐方式
  VerticalDirection verticalDirection = VerticalDirection.down, // 表示Row在交叉轴（竖直）的对齐方式
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center, // 子组件在交叉轴（竖直）对齐方式
  List<Widget> children = const <Widget>[],
})
```

##### Column

竖直排列，属性和 Row 一致

#### 弹性布局

##### Flex

```dart
Flex({
  ...
  @required this.direction, //弹性布局的方向, Row默认为水平方向，Column默认为垂直方向
  List<Widget> children = const <Widget>[],
})
```

##### Expanded

表示子 Widget 所占空间

```dart
// 划分一半
Flex(
  direction: Axis.horizontal,
  children: <Widget>[
    Expanded(flex: 1, child: Text('hello')),
    Expanded(flex: 1, child: Text('word'))
  ],
))
```

#### 流式布局

使用 Row 时，如果超过一行，则会报错，此时可以使用 Wrap 和 Flow

##### Wrap

```dart
Wrap({
  ...
  this.direction = Axis.horizontal,
  this.alignment = WrapAlignment.start,
  this.spacing = 0.0, // 主轴（水平）间距
  this.runSpacing = 0.0,  //  交叉轴（竖直）间距
  this.runAlignment = WrapAlignment.start, // 交叉轴（对齐方式）
  this.crossAxisAlignment = WrapCrossAlignment.start,
  this.textDirection,
  this.verticalDirection = VerticalDirection.down,
  List<Widget> children = const <Widget>[],
})
```

##### Flow

较少使用，在对自定义布局要求较高才会使用

**优势**：

1. 性能好；Flow 是一个对子组件尺寸以及位置调整非常高效的控件，Flow 用转换矩阵在对子组件进行位置调整的时候进行了优化：在 Flow 定位过后，如果子组件的尺寸或者位置发生了变化，在 FlowDelegate 中的 paintChildren()方法中调用 context.paintChild 进行重绘，而 context.paintChild 在重绘时使用了转换矩阵，并没有实际调整组件位置。
2. 灵活；由于我们需要自己实现 FlowDelegate 的 paintChildren()方法，所以我们需要自己计算每一个组件的位置，因此，可以自定义布局策略

**劣势**：

1. 使用复杂
2. 无法自适应子组件，必须指定父容器大小或实现 TestFlowDelegate 的 getSize 返回固定大小

#### 层叠布局（绝对定位）

##### Stack

实现组件层叠

```dart
Stack({
  this.alignment = AlignmentDirectional.topStart, // 设置没有定位的元素如何对齐
  this.textDirection,
  this.fit = StackFit.loose, // 没有定位的子组件如何去适应Stack的大小
  this.overflow = Overflow.clip,
  List<Widget> children = const <Widget>[],
})
```

##### Positioned

```dart
const Positioned({
  Key key,
  this.left,
  this.top,
  this.right,
  this.bottom,
  this.width,
  this.height,
  @required Widget child,
})
```

```dart
Stack(
  alignment: AlignmentDirectional.center,
  fit: StackFit.expand, // 如果未定位，则撑满Stack
  children: <Widget>[
    Text('hello word'),
    Positioned(
      child: Text('I am pos1'),
      left: 0,
    ),
    Positioned(
      child: Text('I am pos2'),
      top: 0,
    ),
  ],
)
```

实现子组件按照父组件四个角定位

#### 对齐和相对定位

##### Align

```dart
Align({
  Key key,
  this.alignment = Alignment.center,
  this.widthFactor, // 缩放因子，如果为 null，则尽可能占多的宽度
  this.heightFactor, // 缩放因子，如果为 null，则尽可能占多的高度
  Widget child,
})
```

```dart
Container(
  color: Colors.blue,
  child: Align(
    alignment: Alignment.center,
    widthFactor: 3,
    heightFactor: 3,
    child: FlutterLogo(
      size: 60,
    ),
  ),
)

// 缩放因子为3，则Align宽高都为60*3
```

##### Alignment

Alignment 以矩形中心为坐标原点，x、y 轴同为[-1,1]，则左上角顶点为[-1,-1]，右下角顶点为[1,1]，除了 Alignment 枚举外，也可以自定义

```dart
Alignment(-1, 0); // 等于Alignment.centerLeft
```

实际偏移为：`(Alignment.x*childWidth/2+childWidth/2, Alignment.y*childHeight/2+childHeight/2)`

##### FractionalOffset

以左侧顶点为原点，则 x,y 同为[0,1]，实际偏移为 `FractionalOffset.x * childWidth, FractionalOffset.y * childHeight`

```dart
FractionalOffset(0.1, 0.1);
```

**Align 和 Stack 区别**：

1. Align 的偏移计算是根据 Alignment，存在两种不同的原点计算方式；Stack 是以父元素四个角计算
2. Align 只有一个子元素；Stack 多个子元素

### 容器

#### Padding 填充

```dart
Padding({
  ...
  EdgeInsetsGeometry padding, // EdgeInsetsGeometry 抽象类，一般使用 EdgeInsets
  Widget child,
})
```

##### EdgeInsets

1. fromLTRB(double left, double top, double right, double bottom)：分别指定四个方向的填充。
2. all(double value) : 所有方向均使用相同数值的填充。
3. only({left, top, right ,bottom })：可以设置具体某个方向的填充(可以同时指定多个方向)。
4. symmetric({ vertical, horizontal })：用于设置对称方向的填充，vertical 指 top 和 bottom，horizontal 指 left 和 right

```dart
Padding(
 padding: EdgeInsets.all(),
 child: Text('Hello word'),
)
```

#### 尺寸限制容器

##### ConstrainedBox

对子 Widget 额外约束

```dart
ConstrainedBox({
  ...
  @required this.constraints, // BoxConstraints: constraints
  Widget child,
})
```

```dart
const BoxConstraints({
  this.minWidth = 0.0,
  this.maxWidth = double.infinity,
  this.minHeight = 0.0,
  this.maxHeight = double.infinity,
});
```

BoxConstraints 也提供静态方法，例如 BoxConstraints.expand();

```dart
ConstrainedBox(
  constraints: BoxConstraints(maxHeight: 50, minWidth: double.infinity),
  child: Container(
    color: Colors.red,
  ),
))
```

**多重限制**：多个 ConstrainedBox 限制

1. min\*：取父子值中较大的
2. max\*：取父子值中较小的

##### SizedBox

对子 Widget 固定宽高

```dart
const SizedBox({ Key key, this.width, this.height, Widget child });
```

```dart
SizedBox(
  width: 60,
  height: 60,
  child: Container(
    color: Colors.red
  )
)
```

##### UnconstrainedBox

对子 Widget 无限制，一般不使用，但是在对去除 ConstrainedBox 影响时，比较有用

**UnconstrainedBox 对父组件限制的“去除”并非是真正的去除**，而仅仅是去除 UnconstrainedBox 包裹的 Widget。对于其他 Widget，影响任然存在。

_定义一个通用的组件时，如果要对子组件指定限制，那么一定要注意，因为一旦指定限制条件，子组件如果要进行相关自定义大小时将可能非常困难，**因为子组件在不更改父组件的代码的情况下无法彻底去除其限制条件**_。例如在实际开发中，当我们发现已经使用 SizedBox 或 ConstrainedBox 给子元素指定了宽高，但是仍然没有效果时，几乎可以断定：已经有父元素已经设置了限制

##### AspectRatio

指出长宽比例

##### LimitedBox

指定最大宽高

##### FractionallySizedBox

根据父容器宽高的百分比来设置子组件宽高

## Algorithm

### 乘积最大子数组

思路 1：暴力法
思路 2：根据正负理论，进行动态规划

dp[i] 表示第 i 位乘积最大值

```text
dp[i][max] = max(
  nums[i],
  nums[i] < 0 ? dp[i-1].min * nums[i] : dp[i-1].max * nums[i]
);

dp[i][min] = min(
  nums[i],
  nums[i] < 0 ? dp[i-1].max * nums[i] : dp[i-1].min * nums[i]
);
```

```typescript
function maxProduct(nums: number[]): number {
  let prevMax = 1;
  let prevMin = 1;
  let _prevMax = 1;
  let max = -Infinity;

  for (const num of nums) {
    _prevMax = prevMax; // 防止prevMax值被覆盖
    prevMax = Math.max(num, num < 0 ? prevMin * num : prevMax * num);
    prevMin = Math.min(num, num < 0 ? _prevMax * num : prevMin * num);

    max = Math.max(max, prevMax);
  }

  return max;
}
```

[乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray/)

### 最大子序和

思路 1：暴力法
思路 2：动态规划 dp[i] = max(0, dp[i-1]) + nums[i]

```typescript
function maxSubArray(nums: number[]): number {
  let prevMax = 0;
  let max = -Infinity;

  for (const num of nums) {
    prevMax = Math.max(0, prevMax) + num;
    max = Math.max(max, prevMax);
  }

  return max;
}
```

[最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)

### 验证回文字符串 Ⅱ

思路 1：暴力法，循环字符串，删除后判断是否为回文
思路 2：双指针，i 指向前，j 指向后，由于只能删除一个字符，则当出现 str[i] !== str[j] 时，可能删除 i（i++），可能删除 j（j--），第二次出现则不是回文了

```typescript
function validPalindrome(s: string): boolean {
  return isValid(0, s.length - 1, false);

  function isValid(
    start: number,
    end: number,
    onOff: boolean = false
  ): boolean {
    while (start < end) {
      if (s[start] !== s[end]) {
        return (
          onOff ||
          isValid(start + 1, end, true) ||
          isValid(start, end - 1, true)
        );
      }
      start++;
      end--;
    }
    return true;
  }
}
```

[验证回文字符串 Ⅱ](https://leetcode-cn.com/problems/valid-palindrome-ii/)

### 每个元音包含偶数次的最长子字符串

思路 1：暴力法，对 str[i] ~ str[j] 字符串判断
思路 2：动态规划，以 dp[i] = []; 来记录第 i 位前，各个元音的数量。则当 dp[i] ~ dp[j] 的各个原因差值为偶数，则满足条件

1. 由于只是偶数，则只能出现 0、1 的情况，且如果要出现偶数，则必须对应 0=>0，1=>1。由于只有 5 个元音
2. 只是存在 1、0 的话，那么 1=>1、0=>0 正确；1=>0 或 0=>1 错误，所以可以使用 `^`来计算
3. 此时可以使用 `{{a:0,e:0,i:0,o:0,u:0}: 0, {a:0,e:0,i:0,o:0,u:0}: 2}` 来维护，但维护成本太大，可以使用 **5 个二进制来表示**： `00000`， 如果第 dp[i] ^ dp[j] === 0，则表示符合条件

```typescript
// 动态规划
function findTheLongestSubstring(s: string): number {
  const store: number[] = new Array(32); // 使用数组存储，也可以使用对象存储
  const wordMap: Map<string, number> = new Map([
    ['a', 0],
    ['e', 1],
    ['i', 2],
    ['o', 3],
    ['u', 4],
  ]);

  store[0] = 0;

  let prevRadixCount = 0;
  let max = 0;

  for (let i = 1; i <= s.length; i++) {
    if (wordMap.has(s[i - 1])) {
      let nowRadix = 1 << wordMap.get(s[i - 1])!;
      prevRadixCount ^= nowRadix;
    }

    if (store[prevRadixCount] !== undefined) {
      max = Math.max(max, i - store[prevRadixCount]);
    } else {
      store[prevRadixCount] = i;
    }
  }
```

[每个元音包含偶数次的最长子字符串](https://leetcode-cn.com/problems/find-the-longest-substring-containing-vowels-in-even-counts/)
