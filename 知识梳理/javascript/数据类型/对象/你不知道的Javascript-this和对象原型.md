# this 和 对象

## this

在 JavaScript 中，this 是在运行时绑定的，而非在编译时。所以 this 与函数在何处声明没有关系，取决于函数调用方式。简单来说就是**谁调用了函数，this 指向谁**

### this 的绑定及优先级顺序

#### new 绑定

```
function Person(name, age) {
  this.name = name;
  this.age = age;
}
new Person('Tom', 12);
```

在 JavaScript 中，new 和传统的面向对象语言是有所区别的  
例如 C++中

- 构造函数是一个特殊函数，且属于该类
- 构造函数与类同名
- 在实例化类时，会调用该特殊函数

而在 JavaScript 中

- 构造函数并不属于某个类，它是一个单独的函数，可以说任意一个函数都可以作为构造函数
- new 的作用也不是实例化类，事实上，JavaScript 中没有类这个概念，new 仅仅是**构造调用**该函数，如以下步骤实现：

**构造调用**

1. 创建一个新的对象
2. 将该对象绑定到"构造函数"中的 this 上
3. 将该对象连接"构造函数"的原型
4. 如果"构造函数"返回值为对象，则返回值为该对象，否则为 this

#### 显示绑定 call、apply、bind

显示绑定 this，使优先级低于显示绑定的都无效。

```
function.call(thisArg, arg1, arg2, ...)
function.apply(thisArg, [arg1, arg2, ...])
function.bind(thisArg, arg1, arg2, ...)
```

tips：

- **请注意，如果 thisArg 为 undefined、null 时，将应用其他规则**
- bind 函数也适用 new 运算符，但是 thisArg 将无效，而参数是有效的，例如

```
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.show = function name(params) {
  console.log(this.name, this.age);
}

const Tom = Person.bind({}, "Tom", 12);
new Tom().show(); // Tom 12
```

#### 隐式绑定

函数被对象调用，或调用处存在上下文对象

```
const Tom = {
  name: 'Tom',
  show() {
    console.log(this.name);
  }
};
Tom.show();
```

#### 默认绑定

无法应用其他规则时，则为默认绑定

```
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person();
```

#### 箭头函数

箭头函数是一种特殊的函数

- 箭头函数没有自身的 this，是按照词法作用域来决定
- 箭头函数没有原型
- 箭头函数不能 使用 new 调用

## 对象

Javascript 存在 7 中数据类型 string、number、boolean、null、undefined、symbol、object，除去 object 外都为基本类型，object 为复杂类型。

### 属性和方法

通常来说 对象的 Key 值为函数称为方法，非函数称为属性

### 属性描述符

```
Object.defineProperty(target, propertyKey, attributes)
```

描述符分为两组

**共享可选键**

- configurable：是否可修改该属性的描述符和是否可删除该属性，默认 false
- enumerable：是否可遍历该属性，默认 false

tips：**即便属性是 configurable:false， 我们还是可以，把 writable 的状态由 true 改为 false， 但是无法由 false 改为 true**

**互斥可选键**

- writable：是否可通过赋值运算修改该属性值，默认 false
- value：属性值，默认 undefined

---

- get：返回值作为该属性的值，默认 undefined
- set：赋值属性时，将调用该函数，默认为 undefined
