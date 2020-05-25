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

class：类型声明空间、变量声明空间
typeof class：类型声明空间

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

### 调试

### Widget

#### Widget、Element

#### Widget 按照状态分类

### 状态管理

### 文本

### 按钮

### 图片

### Checkbox、Radio

### TextFiled、Form

### 安全

#### HTML 安全

#### a 标签

## LeetCode

### 快速求幂

### 最小栈、最大栈

### 在无限数 1、2、3、4、5...中寻找第 n 个数

### 和为 k 的子数组

### 股票算法

### 全排列
