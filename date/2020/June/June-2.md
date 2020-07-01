# June-2

## Typescript

### interface 扩展

1. 对 node_modules 的扩展

   例如，对 express 的 Request 扩展

   ```typescript
   declare module express {
     export interface Request {
       ...
     }
   }
   ```

2. 对本地模块的扩展

   例如，对同级目录的 type.d.ts 的 Request 进行扩展

   ```typescript
   declare module './type' {
     export interface Request{
       ...
     }
   }
   ```

**特别注意**：declare module 的模块名，为 import 的文件名

## Flutter

### 动画

#### 动画结构

#### 自定义路由切换动画

#### Hero 动画

## 设计模式

### 发布订阅和观察者模式

## LeetcCode

### 等式方程的可满足性

### 每日温度

### 三数之和
