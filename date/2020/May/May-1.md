# May-1

## Typescript

### tsconfig.json

#### tsconfig.json

```
{
  "compilerOptions": {
    /* 基本选项 */
    "target": "es5",                       // 指定 ECMAScript 目标版本: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', or 'ESNEXT'
    "module": "commonjs",                  // 指定使用模块: 'commonjs', 'amd', 'system', 'umd' or 'es2015'
    "lib": [],                             // 指定要包含在编译中的库文件，譬如 node，jest
    "allowJs": true,                       // 允许编译 javascript 文件
    "checkJs": true,                       // 报告 javascript 文件中的错误
    "jsx": "preserve",                     // 指定 jsx 代码的生成: 'preserve', 'react-native', or 'react'
    "declaration": true,                   // 生成相应的 '.d.ts' 文件
    "sourceMap": true,                     // 生成相应的 '.map' 文件
    "outFile": "./",                       // 将输出文件合并为一个文件
    "outDir": "./",                        // 指定输出目录
    "rootDir": "./",                       // 用来控制输出目录结构 --outDir.
    "removeComments": true,                // 删除编译后的所有的注释
    "noEmit": true,                        // 不生成输出文件
    "importHelpers": true,                 // 从 tslib 导入辅助工具函数
    "isolatedModules": true,               // 将每个文件做为单独的模块 （与 'ts.transpileModule' 类似）.

    /* 严格的类型检查选项 */
    "strict": true,                        // 启用所有严格类型检查选项
    "noImplicitAny": true,                 // 在表达式和声明上有隐含的 any类型时报错
    "strictNullChecks": true,              // 启用严格的 null 检查
    "noImplicitThis": true,                // 当 this 表达式值为 any 类型的时候，生成一个错误
    "alwaysStrict": true,                  // 以严格模式检查每个模块，并在每个文件里加入 'use strict'

    /* 额外的检查 */
    "noUnusedLocals": true,                // 有未使用的变量时，抛出错误
    "noUnusedParameters": true,            // 有未使用的参数时，抛出错误
    "noImplicitReturns": true,             // 并不是所有函数里的代码都有返回值时，抛出错误
    "noFallthroughCasesInSwitch": true,    // 报告 switch 语句的 fallthrough 错误。（即，不允许 switch 的 case 语句贯穿）

    /* 模块解析选项 */
    "moduleResolution": "node",            // 选择模块解析策略： 'node' (Node.js) or 'classic' (TypeScript pre-1.6)
    "baseUrl": "./",                       // 用于解析非相对模块名称的基目录
    "paths": {},                           // 模块名到基于 baseUrl 的路径映射的列表
    "rootDirs": [],                        // 根文件夹列表，其组合内容表示项目运行时的结构内容
    "typeRoots": [],                       // 包含类型声明的文件列表
    "types": [],                           // 需要包含的类型声明文件名列表
    "allowSyntheticDefaultImports": true,  // 允许从没有设置默认导出的模块中默认导入。

    /* Source Map Options */
    "sourceRoot": "./",                    // 指定调试器应该找到 TypeScript 文件而不是源文件的位置
    "mapRoot": "./",                       // 指定调试器应该找到映射文件而不是生成文件的位置
    "inlineSourceMap": true,               // 生成单个 soucemaps 文件，而不是将 sourcemaps 生成不同的文件
    "inlineSources": true,                 // 将代码与 sourcemaps 生成到一个文件中，要求同时设置了 --inlineSourceMap 或 --sourceMap 属性

    /* 其他选项 */
    "experimentalDecorators": true,        // 启用装饰器
    "emitDecoratorMetadata": true          // 为装饰器提供元数据的支持
  },
  "files": [],   // 指定文件
  "include": [], // 包含的文件、文件夹
  "exclude": [] // 不包含的文件、文件夹
}
```

include 指定文件，exclude 排除文件，例如，_src/\**/*_

##### command

- `tsc -p path` 指定 tsconfig.js 路径
- `tsc -w` watch 模式

### 类型系统

#### never、void 区别

**void**：

1. 用作表达式 `void 1` 表示 undefined
2. 用作类型，void 表示无返回值，由于 js 函数特殊性，无任何显示返回值时，返回 undefined
3. void 可赋值给 null，undefined，在*strictNullChecks*下，只能赋值给 undefined

**never**：表示无法发生，无法出现的类型，例如 `while(true){}; throw Error()`，**never 无法赋值给其他类型，只能赋值给 never**

#### Tuple

##### Tuple 的边界溢出

```typescript
const t: [number, string] = [1, 'A'];
t.push(2); // 可以正常运行
t[2]; // 获取报错
```

#### Enum

1. 数字枚举：`enum ENumber { 'Red', 'Blue };`，数字枚举数字自增，如果未指定则以 0 开始，如果指定则以指定的递增
2. 字符串枚举：`enum EString { 'Red': 'Red' };`，**无法使用常量枚举表达式**

```typescript
enum ES {
  name = 'name',
  // name = 1 + 1 无法使用常量枚举表达式
}
```

3. 异构枚举：`enum EMix { 'Red', 'Blue' = 'Blue' };`，异构枚举要小心数字自增，**自增前提是数字枚举前面也是数组枚举**
4. const 枚举：`const enum EConstant { 'Red' };`，**常量枚举经过编译阶段后，不会存在，其他枚举类型编译为对象**

**常量枚举表达式：表达式的子集，可以在编译阶段完全求值**

1. 字符串、数字
2. 对先前的枚举引用
3. 带括号的常量枚举表达式
4. +、-、\*、/、%、<<、>>、>>>、&、|、^ 、~ 表达式

```typescript
const a = { age: 20 };
const arr = [2];
enum E1 = {
  age = 25,
}
enum E {
  t1 = a.age,
  t2 = arr[0],
  t3 = E1.age,
  t4 = 2 >> 1,
}
```

**const 枚举**

const 枚举可以使用常量枚举表达式，但**只能使用部分**

1. 字符串、数字
2. 对先前的枚举引用
3. +、-、\*、/、%、<<、>>、>>>、&、|、^ 、~ 表达式

**无法使用括号表达式和.表达式**

```typescript

const E1 {
  // t1 = a.age;  要报错
  t1 = 1 + 1, // 可以运行
  t2 = E1.age // 可以运行
}
```

#### Interface

interface：一些方法和属性的接口，属于特征集合，用于描述类，**只存在于编译阶段**

##### 可索引接口

描述数组或对象的接口

```typescript
interface AI {
  [index: number]: any;
}

interface OI {
  [key: string]: any;
}
```

##### 函数接口

描述函数

```typescript
interface IF {
  (x: number, y: number): number;
}
```

#### Function

##### Function 的重载和重写

1. 重载：在其他语言中，表示对不同的参数个数或参数类型调用不同的函数，_typescript 没有正真意义上的重载，只是函数的参数进行不同声明_，**最终还是调用一个函数**

```typescript
declare function fn(x: string, y: string): string;
declare function fn(x: number, y: number): number;

// 最终由js完成计算
function fn(x, y) {
  return x + y;
}
```

2. 重写：子类继承父类时，对继承父类的方法修改

```typescript
class Person {
  show() {}
}

class YellowMan extends Person {
  show() {}
}
```

#### Class

##### 访问权限控制符

1. private
2. public
3. protected

## Flutter

### Flutter Demo

```dart
import 'package:flutter/material.dart';

void main() => runApp(new TestApp()); // 入口函数

// 入口 app
class TestApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Test Demo', // 标题
      theme: new ThemeData.light(), // 主体
      home: new Home(title: 'Demo'), // 首路由
    );
  }
}

// Widget 简单分为 StatelessWidget（无状态组件） StatefulWidget（有状态组件），有状态组件中 StatefulWidget 本身是不变的，而其中 State 类是可变的，
class Home extends StatefulWidget {
  Home({Key key, this.title}) : super(key: key); // 构造函数运行，由于 title 是 final 声明，则只能在构造函数改变一次

  final String title;

  @override
  HomeState createState() => HomeState();
}

class HomeState extends State<Home> {
  int count = 0;

  void addCount() {
    // 调用 setState 时，会调用 HomeState 的 build 方法，实现UI更新
    setState(() {
      count++;
    });
  }

  // build 方法在初始化时会执行，后续在setState调用时执行
  @override
  Widget build(BuildContext context) {
    // Scaffold 是 Material 库中提供的页面脚手架
    return new Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      // Center 将组件对齐到屏幕中心
      body: Center(
          child: new Column(children: <Widget>[
        new Text("你点击了多少次"),
        new Text(
          '$count',
          style: Theme.of(context).textTheme.headline,
        )
      ])),
      floatingActionButton: new FloatingActionButton(
        onPressed: addCount,
        tooltip: 'Add',
        child: new Icon(Icons.add),
      ),
    );
  }
}
```

### Flutter Router

#### Navigator

Navigator：flutter 的路由控制的 Widget

1. `Future push(BuildContext context, Route route)`
2. `bool pop(BuildContext context, [ result ])`
3. `Future pushNamed(BuildContext context, String routeName,{Object arguments})`

**Navigator.push(BuildContext context, Route route) 等价 Navigator.of(context).push(Route route)**

```dart
new RaisedButton(onPressed: () {
  Navigator.push(
    context,
    new MaterialPageRoute(builder: (context) {
      return // 新的Widget实例
    })
  );
}),
```

##### 路由传参

```dart
Navigator.push(
  context,
  new MaterialPageRoute(builder: (context) {
    return // 新的Widget实例 new Home(此时可以传参);
  })
);

Navigator.pop(context, ''); // 路由返回时传参
```

#### MaterialPageRoute

继承于 PageRoute，PageRoute 类是一个抽象类，表示占有整个屏幕空间的一个模态路由页面，它还定义了路由构建及切换时过渡动画的相关接口及属性，MaterialPageRoute 是 Material 组件库提供的组件，它可以针对不同平台，实现与平台页面切换动画风格一致的路由切换动画

- builder 是一个 WidgetBuilder 类型的回调函数，它的作用是构建路由页面的具体内容，返回值是一个 widget。我们通常要实现此回调，返回新路由的实例。
- settings 包含路由的配置信息，如路由名称、是否初始路由（首页）。
- maintainState：默认情况下，当入栈一个新路由时，原来的路由仍然会被保存在内存中，如果想在路由没用的时候释放其所占用的所有资源，可以设置 maintainState 为 false。
- fullscreenDialog 表示新的路由页面是否是一个全屏的模态对话框，在 iOS 中，如果 fullscreenDialog 为 true，新页面将会从屏幕底部滑入（而不是水平方向）

### 命名路由

```dart
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      initialRoute: '/', // 初始路由
      // 命名路由设置
      routes: {
        'tip': (context) => new Tip(),
        'tip1' (context) => new Tip(title: ModalRoute.of(context).settings.arguments.toString()); // 可以还是构造函数传参
      },

      // 路由钩子，该钩子只对命名路由有效
      onGenerateRoute:(RouteSettings settings){
        return MaterialPageRoute(builder: (context){
            String routeName = settings.name;
            // 如果访问的路由页需要登录，但当前未登录，则直接返回登录页路由，
            // 引导用户登录；其它情况则正常打开路由。
      }),
      onUnknownRoute: (){}, // 打开不存在路由
      navigatorObservers: (){}, // 监听所有路由
    );
  }
}

// 跳转
Navigator.pushNamed(context, 'tip', arguments: 'Hi'); // 命名路由传参，由以下方法获取参数

// 获取参数
@override
Widget build(BuildContext context) {
  var args = ModalRoute.of(context).settings.arguments; // ps dart 所有数据类型都输对象，无论是数字，字符串，null...
}
```

### FLutter 包管理

由 _pubspec.yaml_ 文件管理

- name：应用或包名称。
- description: 应用或包的描述、简介。
- version：应用或包的版本号。
- dependencies：应用或包依赖的其它包或插件。
- dev_dependencies：开发环境依赖的工具包（而不是 flutter 应用本身依赖的包）。
- flutter：flutter 相关的配置选项

导入包，以及包路径

```yaml
# pubspec.yaml

dependencies:
  cupertino_icons: ^0.1.2

  pkg1:
    path: xxx #指定路径

  pkg2:
    git:
      url: git://github.com/xxx/pkg1.git #指定url

  pkg3:
    git:
      url: xx
      path: xx
  #同时指定url和本地路径，先寻找url，后寻找path
```

## LeetCode

### 最大正方形

思路：动态规划

dp(i,j) 表示 matrix[i][j]的正方形边长，递推公式：dp(i,j) = matrix[i][j] === 0 ? 0 : min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1;（原理是，只有一个点的左，左上，上全部为 n，时，该正方形的边长为 n+1，任何一个的缺失，都无法组成正方形）

```typescript
function maximalSquare(matrix: string[][]): number {
  if (matrix.length < 1) return 0;
  const store: number[][] = [];
  let max = 0;

  const col = matrix.length;
  const row = matrix[0].length;

  for (let i = 0; i < col; i++) {
    store[i] = [];
    for (let j = 0; j < row; j++) {
      if (matrix[i][j] === '0') {
        store[i][j] = 0;
      } else {
        const min: number =
          i === 0 || j === 0
            ? 1
            : Math.min(store[i - 1][j], store[i - 1][j - 1], store[i][j - 1]) +
              1;

        store[i][j] = min;
        max = Math.max(max, min);
      }
    }
  }

  return max * max;
}
```

[最大正方形](https://leetcode-cn.com/problems/maximal-square/)
