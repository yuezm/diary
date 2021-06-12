# Javascript this

## this 是什么，为什么需要 this

在 Javascript 中，this 是一个对象，存在于 _构造函数中_ 或 _prototype 对象的方法中_（合并起来说就是存在于方法中），**指向当前调用的对象**。

```javascript
function Person{
  this.name = ''; // 存在于构造函数中
}

// 存在于 prototype 对象的方法中
Person.prototype.talk = function(){
  console.log(typeof this); // object
  return this.name;
}

var person = new Person();
person.talk(); // talk 方法内的 this 指向 person 对象
```

### 为什么需要 this

1. 用于指向当前类的实例（指向当前调用对象）

   - 使用 this 访问对象的属性
   - 使用 this 访问对象的方法

```javascript
Person.prototype.talk = function () {
  return this.name;
};

new Person().talk(); // 访问 name
```

2. 返回当前调用对象，例如可以用于*链式调用*

```javascript
Person.prototype.talk = function () {
  console.log(`I am ${this.name}`);
  return this;
};
Person.prototype.run = function () {
  console.log(`${this.name} is running`);
  return this;
};

new Person().talk().run(); // 链式调用
```

设想一下，如果不存在 this，那么在方法中如何调用当前对象？如何访问当前对象的属性、方法？难道是像如下这样？

```javascript
Person.prototype.talk = function () {
  return Person.name; // ???
};
```

显然这个是错误的方法，因为我们要访问的是 Person 的实例，而非 Person

## this 的几个误解

### this 指向函数自身？

由于 this 本身的单词意思，容易造成误解，this 的英文翻译为 ”此，本身，这个“，但在 Javascript 当中，this 并非指向当前函数

   可以由如下代码证实一下

```javascript
function testThis() {
  return this;
}

console.log(testThis() === testThis); // false
```

### this 指向作用域？
这个问题有点迷惑性，因为确实有的时候 this _表现确实像作用域_，如下代码所示

```javascript
var test = 'Hello Test';

function testThis() {
  console.log(this.test); // 表现得像词法作用域
}

testThis();
```

是不是表现得很像作用域，~~函数作用域内无法寻找 test，则向上级作用域查找，获取到 test~~，但千万要注意，<font color="red">此处仅仅是表现得像作用域而已，但绝对不是作用域</font>，具体可以通过以下两点来分析

1. 全局作用域中 var 声明时，声明属性会赋值到全局对象 window 上，表现为 `window.test = "Hello Test";`
2. testThis 执行时，内部 this 指向 window，则访问到了 window 的 test 属性

### this 是指针？ 

首先 Javascript 不存在指针这一说；其次在 C/C++中，指针是一个特殊的变量，内部存储的数值解释为**内存的地址**。但在 Javascript 中是无法直接操作内存的，所以 this 是指针这种说法是不严谨的

**Javascript 中是无法操作内存**这句话其实不是很严谨，因为到目前为止可以使用 ArrayBuffer 来操作内存。但是需要明确一点，Javascript 必须通过 ArrayBuffer 来操作内存，除此之外无法操作内存，举个例子

```cpp
int main(){
  int n = 1;
  int *nPoint = &n; // 假装nPoint的数值是 0x7ffeed2c252c

  *nPoint // 可以通过 *nPoint 来访问该内存，得到内存中存储的值
}
```

```javascript
// 假设给到 javascript *nPoint，或者给到 0x7ffeed2c252c
*nPoint // 无效语法，无法识别
0x7ffeed2c252c // 也无法识别，Javascript只认为这是个16进制的数，无法根据该数值读取到该内存的值

// 但可以使用 ArrayBuffer，例如使用 TypedArray
var t = new Uint8Array(20);
console.log(t[0]);
```

## this 的绑定规则及优先级

### new 构造调用

在 Javascript 中，new 操作符和其他面向对象语言表现得不一致，具体见 [Javascript 面向对象#实例化](./Javascript面向对象.md##实例化)。当进行构造调用时，构造函数内的 this 指向生成的对象

```javascript
function Person() {
  this.name = 'Person';
}
var p = new Person();
p.name; // 'Person'，this 指向 p
```

### 显示绑定

Javascript 提供`call、apply、bind` 函数，来显示的绑定 this。如果第一个参数传入非对象（null、undefined 除外，下面会单独介绍）时，则会转化为包装对象

```javascript
// MDN 文档
function.call(thisArg, arg1, arg2, ...);
function.bind(thisArg[, arg1[, arg2[, ...]]]);
function.apply(thisArg, [argsArray]);
```

```javascript
function binding() {
  return this.name;
}

binding.call(new Person()); // 'Person'
```

### 隐式绑定

在调用时，需要考虑该函数是否是被某个对象调用，但需要注意隐式丢失

```javascript
var p = {
  name: 'Person',
  binding() {
    return this.name;
  },
};

p.binding(); // 'Person'
```

#### 隐式丢失

对象函数赋值很容易出现一种情况， this 在非严格模式下指向 window/global；严格模式下为 undefined

```javascript
var p = {
  name: 'Person',
  binding() {
    return this.name;
  },
};

var fn = p.binding;
fn(); // undefined
```

### 默认绑定

判定绑定规则不属于以上三种时，则为默认绑定。在非严格模式下 this 指向 window/global；在严格模式下为 undefined

```javascript
function binding() {
  return this.name;
}

binding(); // undefined
```

### 绑定规则优先级

new 构造调用 > 显示绑定 > 隐式绑定 > 默认绑定。下面举个特别的例子。由于默认绑定是在其他规则之后的规则，则优先级必然最低，那如何证明 new 构造调用 > 显示绑定 > 隐式绑定 的优先级呢

#### new 构造调用、显示绑定 对比

```javascript
var p1 = { name: 'Person1' };

function Person() {
  console.log(this);
}

Person.call(p1); // p1 对象

var Binding = Person.bind(p1); // 由于无法直接使用 new Fn.call()，所以得借助 bind
new Binding(); // this为新的Person对象，而非p1对象，由此可以直到 new构造调用 > 显示绑定
```

由此证明： new 构造调用 > 隐式绑定

#### 显示绑定和隐式绑定

```javascript
var p1 = {
  name: 'Person1',
  talk() {
    console.log(this.name);
  },
};

p1.talk.call({ name: 'PersonCall' }); // PersonCall
```

由此证明：显示绑定 > 隐式绑定

### this 的绑定例外

#### 忽略 this

在显示绑定时，如果第一个参数传入 null、undefined 时，则会忽略传入的参数，在非严格模式下 this 指向 window/global；严格模式下为 undefined

```javascript
function binding() {
  console.log(this);
}

binding.call(null); // window/global
```

#### 间接引用

像隐式忽略一样，有时候会创建一些见解引用

```javascript
var p1 = {
  name: 'p1',
  talk() {
    console.log(this.name);
  },
};

var p2 = { name: 'p2' };

p1.talk(); // p1;
(p2.talk = p1.talk)(); // undefined，千万不要以为是 p2
// 变形为

var p3 = p1.talk;
p2.talk = p3;
p3(); // 默认绑定规则
```

## 词法绑定 this（箭头函数）

[ecma-262/6.0 sec-arrow-function-definitions](http://www.ecma-international.org/ecma-262/6.0/#sec-arrow-function-definitions)

ES6 中，出现了一个新的函数 **箭头函数**，箭头函数的 this 和普通函数的绑定规则完全不同，它遵守如下规则

1. 箭头函数是根据词法作用域寻找 this
2. 箭头函数根据作用域确定 this 后，**无法对 this 进行修改**，即便是显示绑定也无法修改（由于箭头函数无[[Constructor]]，则无法使用 new 语句，所以也不存在构造调用规则了）

[ES6 文档中有两段话](http://www.ecma-international.org/ecma-262/6.0/#sec-arrow-function-definitions-runtime-semantics-evaluation)

> Let scope be the LexicalEnvironment of the running execution context  
// 对于箭头函数来说，当前运行的执行上下文的词法环境就是作用域

> An ArrowFunction does not define local bindings for arguments, super, this, or new.target. Any reference to arguments, super, this, or new.target within an ArrowFunction must resolve to a binding in a lexically enclosing environment. Typically this will be the Function Environment of an immediately enclosing function. Even though an ArrowFunction may contain references to super, the function object created in step 4 is not made into a method by performing MakeMethod. An ArrowFunction that references super is always contained within a non-ArrowFunction and the necessary state to implement super is accessible via the scope that is captured by the function object of the ArrowFunction

// 箭头函数没有本地绑定arguments、super、this、new.target

### 箭头函数和普通函数区别

1. 箭头函数无 arguments、super、this、new.target，如果访问这些属性，则按照词法环境解析绑定
2. 箭头函数无法 new 构造调用
3. 箭头函数无 prototype
4. 无法使用 Generator 语法

## Javascript 的 this 到底指向什么

this 指向当前调用对象，这句话是没有问题，但这只是我们（程序员）用来判断 this 值的方式，比如面试的时候被面试官问到了，就用以上步骤去推算。那 this 真正指向的是什么？

### ES5.1

根据 ES5.1 文档，this 为**当前 Execution Contexts 的 ThisBinding 属性**[ECMAScript 第 5 版 - 10.3 执行环境](http://lzw.me/pages/ecmascript/#143)，获取 this 即为获取 Execution Contexts 的属性

> ThisBinding: The value associated with the _this_ keyword within ECMAScript code associated with this execution context.

### ES6

但 ECMAScript 第 6 版文档中，未看到 Execution Contexts 包含 this 属性，看起来有点和 ES5.1 不太一样。Execution Contexts 包含 _code evaluation state_、_Function_、_Realm_、_LexicalEnvironment_、_VariableEnvironment_ 等属性，**请注意 LexicalEnvironment**。

而 this 是调用 _Resolvethisbinding_ 获取 [ecma-262/6.0 sec-resolvethisbinding](http://www.ecma-international.org/ecma-262/6.0/#sec-resolvethisbinding)

> The abstract operation ResolveThisBinding determines the binding of the keyword this using the LexicalEnvironment of the running execution context.

调用方式为 `GetThisEnvironment().GetThisBinding()`，下面对该调用流程进行分析，且以下分析**排除箭头函数影响，未考虑任何异常**

GetThisEnvironment() => 返回*处于运行状态*下的*执行上下文*的*LexicalEnvironment* [ecma-262/6.0 getthisenvironment](http://www.ecma-international.org/ecma-262/6.0/#sec-getthisenvironment)，由于我们这里说的函数执行，则返回的是 envRec = [Function Environment Records](http://www.ecma-international.org/ecma-262/6.0/#sec-function-environment-records)

envRec.GetThisBinding() => 返回[Function Environment Record 的[[thisValue]]](http://www.ecma-international.org/ecma-262/6.0/#sec-function-environment-records-getthisbinding)，至此得到 this 值
