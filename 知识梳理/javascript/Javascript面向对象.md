# Javascript 面向对象

一直在说面向对象，也在说 Javascript 是面向对象、面向过程、函数式编程的语言。那么到底什么是面向对象？

**面向对象程序设计（Object Oriented Programming，OOP）** 是一种计算机编程架构，OOP 的一条基本原则是计算机程序由单个能够起到子程序作用的单元或对象组合而成。OOP 达到了软件工程的三个主要目标：重用性、灵活性和扩展性，其中核心概念是类和对象。[面向对象程序设计-百度百科](https://baike.baidu.com/item/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1/24792?fromtitle=OOP&fromid=1152915)

了解了什么是面向对象，也知道了面向的核心概念是类和对象，那么此时遇到第二个问题，什么是类？什么是对象？类和对象有关系吗？

**对象** 人们研究所研究的事物本身，就是对象，例如一个具体的人、一棵树、一只狗、一条规则等等。对象包含自身属性，方法，实现数据和操作的结合。这里顺带提一嘴，在学数据结构的时候，也看到过这句话，数据结构就是 _数据_ + _操作_

**类**对相同的特性和行为的对象的抽象，例如 Person、Tree、Dog、Rule 等等，其中包含数据的形式和操作

**对象和类的关系**类的实例就是对象，可以理解为*类是模具*，对象是*根据模具创造的产品*

说这么多，下面直接上代码

```typescript
// 这是一个Person类
// 对人的抽象，定义了人的特性，例如名字，年龄，身高，体重等等的数据形式，也定义了人的行为，例如说话，跑步等等
class Person {
  name: string;
  age: number;
  height: number;
  weight: number;
  ...

  talk(){
    console.log('speak english');
  }
  run(){}
  ...
}

// keven 是一个对象，他是一个具体的人，他包含名字，年龄，身高...，也可以说话，跑步...
const keven:Person = new Person();
```

## 面向对象三大特性

1. **继承**子类继承父类，子类与父类表现得很像（子类继承了父类的特性和行为），当然子类也可以包含自己的特性和行为；

```typescript
// YellowPerson 继承了 Person，则继承了 Person 的特性和行为
// 虽然 YellowPerson 内部未声明任何属性和方法，但它已经具有 name，age，height...
class YellowPerson extends Person {
  playTableTennis() {}
}
```

2. **多态**子类重写父类，继承同一个父类的子类对同一个特性或行为会表现得不同

```typescript
// 子类继承了父类，但子类对同一个特性 talk 表现的不同，例如可以说中文，可以说非洲语
class YellowPerson extends Person {
  talk() {
    console.log('说中文');
  }
}

class BlackPerson extends Person {
  talk() {
    console.log('说非洲语');
  }
}
```

3. **封装**内部实现细节对外部隐藏，使用属性描述符来控制成员的访问，属性描述符一般有`private、protected、public`

```typescript
class Person {
  private assets: number; // 他有很多资产，除了他自己，并不想让任何人（包含自己的家人）知道
  protected houseKey: string; // 这个人不想让外人知道自己家的钥匙，除非是自己的家人，例如他的儿子
  public name: string; // 他的名字任何人都可以知道
}
```

## Javascript"面向对象"

上面说到 Javascript 是可以面向对象的，且面向对象的核心是类和对象，那么类和对象在 Javascript 是如何表现的？

### Javascript 的对象

Javascript 的数据类型分为：number，string，boolean，null，undefined，symbol 和 object，其中 object 就是我们说的 Javascript 对象。由于存在其他的数据结构，所以 Javascript 并不是全是对象，即在 Javascript 中，并*不是万物皆是对象*

Javascript 对象有很多，例如有以下内置对象 `Object，Array，Function，RegExp...`，当然你还可以自己创建对象，常用的有以下方式

```javascript
const obj1 = {}; // 字面量，常用
const obj2 = new Object(); // 构造调用，用的很少
const obj3 = new Person(); // 构造调用
const obj4 = Object.create(null); // 用的也不多
```

#### Javascript 对象属性、方法

##### 属性和方法

你可以使用 "." 或 "[]" 来访问属性、方法。我们一般会对对象的成员加以区分：成员值为函数的称为方法，值为非函数称为属性，这是按照其他面向对象的语言来称呼的。

但是在 Javascript 中，一个函数其实不会属于某个对象（其实仅仅是一个引用），即该函数不会是某个对象的方法，所以对*方法*这个称呼不是十分严谨，它仅仅是在进行对象*属性访问*的时候，*返回值是函数*罢了《你不知道的 Javascript》。

**有几个注意点**

1. "." 和 "[]" 区别：
   - "." 一般称为属性访问，且属性名必须满足命名规范；"[]" 一般称为键访问，键名可接受任意的 utf-8/unicode 字符串
   - "." 属性名只能为常量；"[]" 可以为变量
   - "[]" 在 ES6 中可用于计算属性
   - "." 和 "[]" 在 AST 是不一样的，`. => PropertyAccessExpression; [] => ElementAccessExpression`

##### [[GET]]、[[PUT]]

对象的*获取属性*、*设置属性*操作。这里注意一点，和 Getter、Setter 不一样的是 [[GET]]、[[PUT]] 是针对对象的操作，Getter、Setter 是针对对象的某个属性的。这里提一嘴，[[GET]]和[[PUT]]是 ES5.1 的文档，ES6 文档内是[[GET]]和[[SET]]

由下面代码假装模拟一个[[GET]] 和 [[PUT]]，需要注意的是，[[GET]] 和 [[PUT]]并非只关注本对象，还要*按照原型链往上查找*

```typescript
const obj = {};

const proxy = new Proxy(obj, {
  get() {}, // 假装当成[[GET]]
  set() {}, // 假装当成[[PUT]]
});
```

tips：这里只列举了[[GET]]和[[PUT]]，如果你想知道全部的方法，例如[[Delete]]，请点这里 --> [Algorithms for Object Internal Methods](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-8.12)

##### 属性描述符

需要注意的是，属性描述符 `configurable、enumerable、writable` _默认都是 false_

1. 公共属性描述符： `configurable、enumerable`

```typescript
{
  configurable: false, // 该属性无法被改变，无法删除，无法重新设置属性描述，且对无法进行的操作静默失败。ps 严格模式下就会报错哦
  enumerable: false, // 该属性无法被遍历。ps 有些方法还是可以获取到的，例如 Reflect.ownKeys()，Object.getOwnPropertyNames/getOwnPropertySymbols()
}
```

2. 互斥属性描述符，以下两组属性描述符互斥

   - `get、set`：对获取、设置属性拦截

```typescript
{
  get(){},
  set(){},
}
```

- `writable、value`

```typescript
  {
    writable: false, // 该属性无法被修改，如果进行修改则静默错误。ps 严格模式下就会报错哦
    value: xx,
  }
```

##### 存在性检测

以下方法对对象的属性、方法进行存在检测：

|                  | 是否检测原型 | 是否包含 enumerable=false | 是否包含 symbol |
| :--------------- | :----------: | :-----------------------: | :-------------: |
| `in`             |      是      |            是             |       是        |
| `Reflect.has`    |      是      |            是             |       是        |
| `hasOwnProperty` |      否      |            是             |       是        |

##### 迭代

以下方法对对象的迭代

|                         | 是否遍历原型 | 是否遍历 enumerable=false | 是否遍历 symbol |
| :---------------------- | :----------: | :-----------------------: | :-------------: |
| `for...in`              |      是      |            否             |       否        |
| `Object.keys()`         |      否      |            否             |       否        |
| `getOwnPropertyNames`   |      否      |            是             |       否        |
| `getOwnPropertySymbols` |      否      |            是             |       是        |
| `Reflect.ownKeys()`     |      否      |            是             |       是        |

### Javascript 的原型

每个 Javascript 对象，都有一个 _[[Prototype]]_ 的特殊属性, 这就是对象的原型。**[[Prototype]] 本质上是对其他对象的引用**。

_在有的浏览器，可以使用 `__proto__` 访问该属性，比如 Google，但是按标准，需使用 `Object.getPrototypeOf()` 获取_

**原型链**每个对象都存在特殊属性 [[Prototype]]，[[Prototype]] 指向另一个对象，另一个对象也存在 [[Prototype]] 属性...直到为 null 结束，由此对象组成的原型链接就是原型链

**原型链查找**在 [[GET]] 时，如果当前对象不存在该属性，且该对象的 [[Prototype]] 不为空，则会继续沿着 [[Prototype]] 引用的对象继续查找，直到找到该属性或对象为空为止

#### prototype 和 [[Prototype]] 区别

函数也存在 `prototype` 属性，且在 `new` 构造调用时，生成的对象可以访问 `prototype` ，由于都叫*原型*，所以这里对他们进行区分：

**prototype** 用于构造函数中，用于模拟的*类*  
**[[Prototype]]** 用于对象中，用于*实例*。ps 函数也是对象，所以函数有 prototype，也有 [[Prototype]]

```javascript
function Person {}
Person.prototype.xx = xx;

// person 通过 [[Prototype]] 指向 Person.prototype ，从而访问 prototype 对象
var person = new Person();
```

![](https://user-gold-cdn.xitu.io/2020/7/3/173134eb08dca6de?w=397&h=670&f=png&s=23176)

### Javascript 的"类"

在 ES6 之前，Javascript 没有类的概念，对象都是由 **`new` 构造调用 构造函数** 生成对象。在 ES6 之后，可以使用 `class` 了，那么是不是意味着 Javascript 在 ES6 后就有类了呢？

#### Javascript "类" 和其他语言类的区别

这里首选明确一个概念：**<font color="red">Javascript 没有类</font>**，Javascript 中的 class 也只是模拟类的而已（可以使用 typescript 或 babel 编译一下，查看编译后的代码）

```typescript
// typescript3.9
class Person {
  private name: string = '';
  talk(): void {}
}
```

编译后

```javascript
// javascript
var Person = /** @class */ (function () {
  function Person() {
    this.name = '';
  }
  Person.prototype.talk = function () {};
  return Person;
})();
```

也可以这样理解，Javascript 一直使用语法糖来装成有类的样子，其实并没有。下面会从几个方面来说明这个

##### 实例化

**类** 在面向对象中，类是一个模具，通过模具生成事物的步骤叫做实例化，例如 `new Person()`。生成对象后*对象和类互不影响*，且*对象之间互不影响*  
**Javascript"类"** Javascript 生成对象时，依靠的是构造调用，而非类的实例化，例如 `new Person()`。Javascript 生成对象不需要依靠类，而是直接生成，生成后，通过 [[Prototype]] 来模拟类，由于 [[Prototype]] 是 prototype 的引用，所以*对象和类、对象之间可以互相影响*

```cpp
// cpp类

class Person {
 public:
  int age;

  // 构造函数
  Person(int _age) { this->age = _age; }
};

Person *p = new Person(22);
```

```javascript
// js"类"

// 构造函数
function Person(age) {
  this.age = age;
}
Person.prototype.getAge = function () {
  return this.age;
};
Person.prototype.ind = [1, 2, 3];

var p = new Person(22); // new 为构造调用
p.ind.push(4);

var p2 = new Person(23);
p2.ind; // [1, 2, 3, 4]，对象之间互相影响了
```

**Javascript new 简单实现**

1. 创建一个新的对象 newObj
2. newObj 赋值[[Prototype]]
3. 运行构造函数，并使用 call、apply 来指定 this 为 newObj
4. 判构造函数运行后返回值，如果返回值为对象，则返回该对象，否则返回 newObj

```javascript
function newSelf(construct, ...args) {
  var newObj = Object.create(construct.prototype); // [[Prototype]]赋值
  // 也可以使用 var newObj = Object.setPrototypeOf({}, construct.prototype);
  var result = construct.call(newObj, ...args); // 构造函数运行

  return typeof result === 'object' && result !== null ? result : newObj;
}
```

##### 继承

**类** 继承后，子类继承父类的属性和方法  
**Javascript"类"** 继承后，子类并不是继承父类的属性和方法，而是依靠 [[Prototype]] 去访问父类的 prototype

```cpp
// cpp 类继承
class YellowPerson : public Person {
 public:
  YellowPerson(int _age) : Person(_age) { this->age = _age; }
};

YellowPerson *yp = new YellowPerson(22);
```

```javascript
function YellowPerson(age) {
  Person.call(this, age); // 构造函数继承
}

// 原型继承
YellowPerson.prototype = new Person();
YellowPerson.prototype.constructor = YellowPerson;

var yp = new YellowPerson(22);
```

##### 多态

**类** 通过函数重载实现多态  
**Javascript"类"** 通过 [[GET]] 操作时，如果已找到该属性，则不会沿着 [[Prototype]] 继续查询的特性，来实现多态

### Javascript"面向委托"

**面向委托**：某些对象在自身无法寻找属性和方法时，把该请求委托给另一个对象《你不知道的 Javascript》。

```javascript
var person = {
  showAge() {
    return this.age;
  },
};

var yellowMan = Object.create(Person);
yellowMan.age = 22; // yellowMan 自己不包含 age 属性，依靠 [[Prototype]] 访问 person 的 age 属性
```

个人感觉面向委托在 Javascript 中和面向对象表现差不多，只是在以下有点小区别：

- 在面向对象中：利用父类（Person）保存属性和方法，再利用*多态*来实现不同的操作
- 在面向委托中：最好将状态保存在委托者上（YellowPerson），而不是委托对象（Person）

### Javascript 的继承

```javascript
// 父类
function Person(name) {
  this.name = name;
  this.ind = [1, 2, 3];
}
Person.prototype.getName = function () {
  return this.name;
};
```

#### 构造函数继承

```javascript
function YellowPerson(name) {
  Person.call(this, name);
}
```

**优点**：

1. 子类之间属性不共用，**即使为引用数据类型也是不共用的**
2. 构造函数可以传参数

**缺点**：

1. 无法获取父类的 prototype 上定义的方法和属性

#### 原型链继承

```javascript
function YellowPerson() {}

YellowPerson.prototype = new Person();
YellowPerson.prototype.constructor = YellowPerson;
```

切记切记不要这样搞 `YellowPerson.prototype = Person.prototype`，这样会造成子类和父类的 prototype 指向同一个对象，如果子类修改 prototype，则其他类（父类和其他继承过父类的子类）都会发生改变

**优点**

1. 可获取父类的 prototype 上定义的方法和属性

**缺点**

1. 构造函数无法传参
2. 子类虽然能访问到父类的属性，但子类共用父类的属性，如果是基本类型还好说，如果引用类型，则会互相影响
3. 需要修复 constructor 属性

#### 组合继承

```javascript
function YellowPerson(name) {
  Person.call(this, name);
}
YellowPerson.prototype = new Person();
YellowPerson.prototype.constructor = YellowPerson;
```

**优点**

1. 可获取父类的 prototype 上定义的方法和属性
2. 子类属性不共用，**即使为引用数据类型也是不共用的**
3. 构造函数可以传参数

**缺点**

1. 父类运行了两次，多余父类实例
2. 需要修复 constructor 属性

#### 寄生组合继承

```javascript
function YellowPerson(name) {
  Person.call(this, name);
}
YellowPerson.prototype = Object.create(Person.prototype); // Object.setPrototypeOf 也可以
YellowPerson.prototype.constructor = YellowPerson;
```

**优点**

1. 可获取父类的 prototype 上定义的方法和属性
2. 子类属性不共用，**即使为引用数据类型也是不共用的**
3. 构造函数可以传参数
4. 父类只运行一次，无多余实例

**缺点**

1. 需要修复 constructor 属性
