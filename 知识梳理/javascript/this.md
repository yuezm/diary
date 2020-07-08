# Javascript this

## this 是什么，为什么需要 this

在 Javascript 中，this 是一个对象，存在于 _构造函数中_ 或 _prototype 对象的方法中_（合并来说就是存在于方法中），**指向当前调用的对象**。

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

2. 返回当前调用对象，例如用于 _链式调用_

设想一下，如果不存在 this，那么在方法中如何调用当前对象？如何访问当前对象的属性、方法？难道是像如下这样？

```javascript
Person.prototype.talk = function () {
  return Person.name; // ???
};
```

显然这个是错误的方法，因为我们要访问的是 Person 的实例，而非 Person

## this 的几个误解

1. **this 指向自身？**：由于 this 本身的单词意思，容易造成误解，this 的英文翻译为 ”此，本身，这个“，但在 Javascript 当中，this **并非指向当前函数**

   可以由如下代码证实一下

   ```javascript
   function testThis() {
     console.log(this.test);
   }
   testThis.test = 'Hello Test';

   testThis(); // undefined
   ```

2. **this 指向作用域？**：这个问题有点迷惑性，因为确实有的时候 this 的*表现确定像指向作用域*，如下代码所示

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

3. **this 是指针？**：在 C/C++中，指针是一个特殊的变量，内部存储的数值解释为**内存的地址**。但在 Javascript 中是无法直接操作内存的，所以 this 是指针这种说法是不严谨的

**Javascript 中是无法操作内存**这句话其实不是十分严谨，因为目前可以使用 TypedArray 在内存中开辟一个数组缓冲区，可以通过该缓冲区来操作内存。但是需要明确一点，Javascript 必须通过 TypedArray 来操作内存，除此之外本身无法操作内存，举个例子

```cpp
int main(){
  int n = 1;
  int *nPoint = &n; // 假装nPoint的数值是 0x7ffeed2c252c

  // 可以通过 *nPoint 来访问该内存，得到内存中存储的值
}
```

```javascript
// 假设给到 javascript *nPoint，或者给到 0x7ffeed2c252c
*nPoint // 无法识别
0x7ffeed2c252c // 也无法识别，无法根据该数值读取到该内存的值

// 但可以使用TypedArray
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

Javascript 提供`call、apply、bind` 函数，来显示的绑定 this。如果第一个参数传入非对象（null、undefined 下面会单独介绍）时，则会转化为包装对象

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

new 构造调用 > 显示绑定 > 隐式绑定 > 默认绑定。下面举个特别的例子，比较 new 构造调用和显示绑定

由于默认绑定是在其他规则之后的规则，则优先级必然最低，那如何证明 new 构造调用 > 显示绑定 > 隐式绑定 的优先级呢

#### new 构造调用、显示绑定 对比

```javascript
var p1 = { name: 'Person1' };

function Person() {
  console.log(this);
}

Person.call(p1); // p1 对象

var Binding = Person.bind(p1); // 由于无法直接使用 new Fn.call()，所以得借助 bind
new Binding(); // 新的Person对象，而非p1对象，由此可以直到 new构造调用 > 显示绑定
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

**忽略 this**

在显示绑定时，如果第一个参数传入 null、undefined 时，则会忽略传入的参数，在非严格模式下 this 指向 window/global；严格模式下为 undefined

```javascript
function binding() {
  console.log(this);
}

binding.call(null); // window/global
```

**间接引用**

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

ES6 中，出现了一个新的函数 **箭头函数**，箭头函数的 this 和普通函数的绑定规则完全不同，它遵守如下规则

1. 箭头函数是根据词法作用域寻找 this
2. 箭头函数根据作用域确定 this 后，**无法对 this 进行修改**，即便是显示绑定也无法修改（由于箭头函数无[[Constructor]]，则无法使用 new 语句，所以也不存在构造调用规则了）

### 箭头函数 和普通函数区别

除去上述规则外，还有其他不同

1. 箭头函数无 arguments，如果访问 arguments，也是按照作用域查找
2. 箭头函数无 prototype
3. 无法使用 yield，及无法成为 Generator 函数

## Javascript 的 this 到底指向什么
