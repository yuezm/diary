# Javascript this

## this 是什么，为什么需要 this

### Javascript 的 this

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

const person = new Person();
person.talk(); // talk 方法内的 this 指向 person 对象
```

### 为什么需要 this

用于指向当前类的实例（指向当前调用对象）

- 使用 this 访问对象的属性
- 使用 this 访问对象的方法

设想一下，如果不存在 this，那么在方法中如何调用当前对象？如何访问当前对象的属性？难道是像如下这样？

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

2. **this 指向作用域？**：这个问题有点迷惑性，因为确实有的时候 this 的表现确定像指向作用域，如下代码所示

   ```javascript
   var test = 'Hello Test';

   function testThis() {
     console.log(this.test); // 确定特别像词法作用域
   }

   testThis();
   ```

3. this 是指针

## this 的绑定规则及优先级

### this 的绑定例外

1. 忽略 this
2. 简介引用

## 词法绑定 this（箭头函数）
