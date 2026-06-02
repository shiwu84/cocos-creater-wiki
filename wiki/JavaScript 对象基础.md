---
title: JavaScript 对象基础
date: 2026-06-02
tags:
  - javascript
  - concept
aliases:
  - JS 对象基础
  - Object 基础
---

# JavaScript 对象基础

> [!abstract] 摘要
> 对象是 JavaScript 中除七种原始类型之外的第八种类型，用于存储键值对和更复杂的实体。本章讲解对象的创建、属性访问（点符号与方括号）、对象引用与复制、`this` 关键字、构造函数与 `new` 运算符、可选链、Symbol 类型以及对象到原始值的转换。

## 核心概念

### 对象的创建与属性
- 字面量语法：`let obj = { key: value, ... }`（首选）
- 构造函数语法：`let obj = new Object()`
- 属性访问：`obj.prop`（点符号，要求 key 是有效标识符）或 `obj["prop"]`（方括号，支持任意字符串和动态表达式）
- 尾随逗号：最后一个属性后可加逗号，便于增删和排序
- 计算属性：`[expression]` 可在字面量中动态计算键名

### 属性存在性与遍历
- `"key" in obj` 检查属性是否存在（区别于 `obj.key === undefined`）
- `for...in` 遍历对象可枚举属性
- `Object.keys(obj)` / `Object.values(obj)` / `Object.entries(obj)` 获取键/值/键值对数组
- 整数属性按升序排列，其他属性按创建顺序

### 对象引用与复制
- 对象变量存储的是 **引用**（内存地址），而非值本身
- 赋值 `obj2 = obj1` 创建引用副本，两者指向同一对象
- **浅拷贝**：`Object.assign({}, obj)` 或展开运算符 `{ ...obj }`
- **深拷贝**：`structuredClone(obj)` 或 `_.cloneDeep(obj)`（lodash）
- `const` 声明对象：不能重新赋值，但可以修改对象属性

### 方法中的 this
- 对象属性值为函数的称为方法
- 方法简写：`sayHi() { ... }` 替代 `sayHi: function() { ... }`
- `this` 指向调用方法的对象（点符号前的对象）
- `this` 的值在运行时确定，取决于调用上下文（无 `this` 绑定约束）
- 箭头函数没有自己的 `this`，从外层作用域继承

### 构造函数与 new 运算符
- 构造函数：命名以大写开头，通过 `new` 调用
- `new` 的执行步骤：创建空对象 `{}` → 将 `this` 指向该对象 → 执行函数体添加属性 → 隐式返回 `this`
- 支持 `new.target` 检测是否通过 `new` 调用
- 若构造器显式 `return` 对象，则返回该对象（原始类型返回值被忽略）

### Symbol 类型
- 创建唯一标识符：`Symbol("description")`，描述仅用于调试
- 每个 Symbol 值都是唯一的
- 可作为对象的隐藏属性键，避免意外访问或覆盖
- 全局 Symbol 注册表：`Symbol.for(key)` / `Symbol.keyFor(sym)`

### 对象到原始值转换
调用顺序：
1. `obj[Symbol.toPrimitive](hint)` — 自定义转换
2. hint 为 `"string"` 时尝试 `toString()`，再 `valueOf()`
3. hint 为 `"number"` 或 `"default"` 时尝试 `valueOf()`，再 `toString()`

## 关键细节

### 可选链 `?.`
```js
obj?.prop        // obj 存在时访问 prop
obj?.[prop]      // obj 存在时用方括号访问
obj.method?.()   // obj.method 存在时才调用
```
安全访问深度嵌套属性，遇 `null`/`undefined` 时短路返回 `undefined`。

### 垃圾回收
JavaScript 引擎自动管理内存，主要算法：
- **引用计数**：被引用则保留（有循环引用问题）
- **标记清除（Mark-and-Sweep）**：从根对象（全局对象、当前函数局部变量）出发，标记所有可达对象，清除不可达对象

## 与其他系统的关系

- [[JavaScript 基础知识]] — 数据类型中介绍了对象的分类位置
- [[JavaScript 数据类型]] — 数组、Map/Set 等特殊对象结构
- [[JavaScript 对象属性]] — 属性标志、描述符和 getter/setter
- [[JavaScript 函数进阶]] — 函数的 `this` 绑定和调用控制

## 注意事项

- 比较 `==` 和 `===` 对对象都按引用比较（相同引用才为 true）
- `for...in` 会遍历继承的属性，用 `hasOwnProperty` 过滤
- `this` 在非严格模式下指向全局对象，严格模式下为 `undefined`
- 可选链 `?.` 只能在可安全使用的地方使用，不宜滥用
- Symbol 属性在 `for...in` 中被跳过，在 `Object.keys` 中也不可见

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 基础知识]]
- [[JavaScript 数据类型]]
- [[JavaScript 对象属性]]
- [[JavaScript 函数进阶]]

## 原始来源

- [Object（对象）：基础知识](raw/zh.javascript.info/1-js/04-object-basics/index.md)
- [对象](raw/zh.javascript.info/1-js/04-object-basics/01-object/article.md)
- [对象引用和复制](raw/zh.javascript.info/1-js/04-object-basics/02-object-copy/article.md)
- [垃圾回收](raw/zh.javascript.info/1-js/04-object-basics/03-garbage-collection/article.md)
- [对象方法，"this"](raw/zh.javascript.info/1-js/04-object-basics/04-object-methods/article.md)
- [构造器和操作符 "new"](raw/zh.javascript.info/1-js/04-object-basics/06-constructor-new/article.md)
- [可选链 "?."](raw/zh.javascript.info/1-js/04-object-basics/07-optional-chaining/article.md)
- [Symbol 类型](raw/zh.javascript.info/1-js/04-object-basics/08-symbol/article.md)
- [对象 — 原始值转换](raw/zh.javascript.info/1-js/04-object-basics/09-object-toprimitive/article.md)
