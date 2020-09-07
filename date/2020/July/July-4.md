# July-4

## Typescript

### 数组递归声明

#### 多重数组

有个 N 层的数组，内部元素可以为数组或元素，例如 `[[ 1 ], 2, [3, [ 4, 5 ]]]`，这样的 typescript 如何定义

```typescript
type ValueType<T> = T | ValueType<T>[];
```

#### 多重 JSON

有 N 个 JSON，例如 { name: string, children: [ { name: string, children: [ { name: string .... } ] } ] }

```typescript
type Json = { name: string; children: Json[] };
```

#### 类型 Iterator

```typescript
type Json = [string, { [key: string]: string }, ...Json[]]; // ...Json 其实相等于 Json，但是如果按照如下所写

type Json = [string, { [key: string]: any }, Json]; // 表示为一个元祖，必须具有三个元素，和上面的类型定义完全不同
```

[Recursive type references](https://github.com/microsoft/TypeScript/pull/33050)

### 声明合并

#### Interface 合并

```typescript
interface A {
  name: string;
}
interface A {
  age: number;
}

// 会合并为
interface A {
  name: string;
  age: number;
}
```

#### namespace 合并

```typescript
namespace A {
  export class B {}
}

namespace A {
  export class C {}
}

// 合并为

namespace A {
  export class B {}
  export class C {}
}
```

namespace 还可以和其他类型合并，例如 `class、function、enum`

```typescript
class A {}

namespace A {
  export const id = 2; // 起始就相当于给A对象，添加了个属性
}

// 函数也是一样的

function B() {}

namespace B {
  export const id = 3;
}
```

#### Enum 合并

```typescript
enum A {
  NAME = 1,
}

enum A {
  AGE = 2,
}

// 合并为
enum A {
  NAME = 1,
  AGE = 2,
}
```

#### 扩展

```typescript
// 使用 declare 进行扩展，不仅是扩展 interface，namespace，enum，也可以扩展 class

// code.ts
export class T {}

export interface G {}

export namespace K {
  export interface J {}
}

export enum O {
  AGE = 1,
}

// main.ts
import { T, K, G, O } from './code';

declare module './code' {
  interface T {
    age: number;
  }

  interface G {
    age: number;
  }

  namespace K {
    interface J {
      age: number;
    }
  }

  enum O {
    NAME = 2,
  }
}

const t: K.J = { age: O.NAME };
```

```typescript
// 全局扩展
export {}; // 让ts识别该文件为模块，而不是脚本
declare global {
  interface Array<T> {}
}
```

### interface 和 type 异同

**同**：

1. 都可以声明类型，包含对象类型
2. 都可以进行扩展，尽管扩展的方式不同
3. 都支持泛型

**异**：

1. 语义：interface 为接口；type 为类型别名
2. 类实现：interface 可以被类实现；type 不行
3. 继承：interface 可以继承其他 interface，也可以继承 type；但 type 无法继承，但可以使用联合类型对 type 进行扩展
4. 声明合并：interface 声明可以合并；type 同声明会报错
5. type 运算：type 可以进行一些列的 type 运算
