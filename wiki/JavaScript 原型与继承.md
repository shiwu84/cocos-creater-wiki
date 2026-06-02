---
title: JavaScript 原型与继承
date: 2026-06-02
tags:
  - javascript
  - concept
aliases: []
---

# JavaScript 原型与继承

> [!abstract] 摘要
> JavaScript 的原型继承是语言的核心特性，通过隐藏属性 ``Prototype`` 实现对象间的属性和方法共享。本章涵盖原型链机制、构造函数的 `F.prototype`、原生内建对象原型、以及现代的 `Object.create` / `Object.getPrototypeOf` 等原型操作方法。

## 核心概念

### 原型链 (Prototype Chain)
- 每个 JavaScript 对象都有一个隐藏的 ``Prototype`` 属性，指向另一个对象或 `null`
- 读取属性时，如果对象自身没有该属性，JavaScript 会自动沿原型链向上查找
- 原型链只能单向，不能形成闭环，一个对象只有一个 ``Prototype``

### 原型继承的写入语义
- 原型仅用于**读取**属性，写入/删除操作直接在对象自身进行
- 访问器属性（getter/setter）是例外：setter 调用会正确触发，不会在对象自身创建同名属性
- 方法调用中的 `this` 始终指向调用对象（`.` 之前的对象），不受原型影响——方法共享，状态不共享

### `F.prototype` 与构造函数
- 每个函数都有默认的 `prototype` 属性，值为 `{ constructor: F }`
- 使用 `new F()` 创建对象时，新对象的 ``Prototype`` 会被设为 `F.prototype`
- `F.prototype` 仅在 `new F` 调用时生效，修改 `F.prototype` 只影响之后创建的对象
- 覆盖 `prototype` 会丢失 `constructor`，需手动保留或重建

### 原生原型
- 所有内建对象（`Array`、`Date`、`Function` 等）的方法存储在对应的 `prototype` 上
- 所有内建原型的顶端是 `Object.prototype`，再往上是 `null`
- 基本类型（String、Number、Boolean）通过临时包装器对象访问方法
- `null` 和 `undefined` 没有对象包装器，也没有原型
- **修改原生原型**一般是不好的做法，唯一例外是 polyfilling（填充规范中已存在但引擎尚未实现的方法）

## 关键细节

### 属性遍历行为
- `for..in` 会遍历自身和继承的可枚举属性
- `Object.keys()` / `Object.values()` 只返回自身属性
- `obj.hasOwnProperty(key)` 可检查属性是否为自有属性（该方法本身来自 `Object.prototype`，但不可枚举）

### 现代的 prototype 方法
- `Object.getPrototypeOf(obj)` — 获取原型（推荐）
- `Object.setPrototypeOf(obj, proto)` — 设置原型（推荐，但避免频繁修改已存在对象的原型，性能极差）
- `Object.create(proto, [descriptors])` — 以指定原型创建对象
- `__proto__` 是历史遗留的 getter/setter，在对象字面量中允许使用 `{ __proto__: ... }`，但不推荐作为 getter/setter

### 无原型的对象
- `Object.create(null)` 或 `{ __proto__: null }` 创建没有原型的"纯字典"对象
- 适用于存储任意（可能包含 `__proto__` 或 `toString` 等）用户提供的键
- 没有 `toString` 等内建方法，但 `Object.keys()` 等静态方法仍然可用

## 与其他系统的关系
- [[JavaScript 类]] 的原型继承本质上是 class 语法的底层实现
- `extends` 关键字在底层设置 `Child.prototype.__proto__ = Parent.prototype`
- [[JavaScript 杂项]] 中 Proxy 的 get/set 捕捉器在原型链查找之前触发

## 注意事项
- 不要混淆 `F.prototype`（构造函数上的普通属性）与 ``Prototype``（对象内部的隐藏属性）
- 方法借用（如 `Array.prototype.join.call(obj)`）可以灵活复用内建方法
- 动态修改已有对象的原型会严重破坏 JavaScript 引擎的内部优化，应避免

## 相关页面
- [[JavaScript 教程概述]]
- [[JavaScript 类]]
- [[JavaScript 杂项]]

## 原始来源
- [原型，继承](raw/zh.javascript.info/1-js/08-prototypes/index.md)
- [原型继承](raw/zh.javascript.info/1-js/08-prototypes/01-prototype-inheritance/article.md)
- [F.prototype](raw/zh.javascript.info/1-js/08-prototypes/02-function-prototype/article.md)
- [原生的原型](raw/zh.javascript.info/1-js/08-prototypes/03-native-prototypes/article.md)
- [原型方法，没有 __proto__ 的对象](raw/zh.javascript.info/1-js/08-prototypes/04-prototype-methods/article.md)
