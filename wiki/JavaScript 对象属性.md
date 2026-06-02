---
title: JavaScript 对象属性
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases:
  - JS 对象属性配置
  - 属性标志与描述符
---

# JavaScript 对象属性

> [!abstract] 摘要
> 深入理解 JavaScript 对象属性的底层机制。本章讲解属性的三个标志（writable、enumerable、configurable）、属性描述符的获取与设置（`Object.defineProperty`/`Object.defineProperties`），以及访问器属性（getter/setter），实现对象属性的精细控制和计算属性。

## 核心概念

### 属性标志（Flags）
除 `value` 外，每个属性还有三个标志：

| 标志 | 含义 | 默认值（常规方式创建） |
|------|------|------------------------|
| `writable` | 值是否可修改 | `true` |
| `enumerable` | 是否在 `for...in` 循环中出现 | `true` |
| `configurable` | 属性是否可被删除、标志是否可被修改 | `true` |

### 属性描述符
**获取描述符**：`Object.getOwnPropertyDescriptor(obj, propertyName)`

返回对象包含：
```json
{
  "value": "...",
  "writable": true,
  "enumerable": true,
  "configurable": true
}
```

**设置/创建属性**：`Object.defineProperty(obj, propertyName, descriptor)`
- 修改已有属性时，只更新指定的标志，其他标志保持原值
- 创建新属性时，未指定的标志默认为 `false`

**批量设置**：`Object.defineProperties(obj, { prop1: descriptor1, ... })`
**获取所有描述符**：`Object.getOwnPropertyDescriptors(obj)`

### 只读属性（writable: false）
- 属性值不可被重新赋值，严格模式下赋值为会抛出错误，非严格模式静默失败
- 注意：属性值为对象时，对象内部属性仍可修改（只限制属性引用不可变）

### 不可枚举（enumerable: false）
- 属性不会出现在 `for...in` 循环中
- 也不会被 `Object.keys()` 返回
- 内置方法如 `toString`、`length` 等默认不可枚举
- `Object.getOwnPropertyNames(obj)` 可获取所有自有属性（含不可枚举）
- 常用于隐藏内部实现细节

### 不可配置（configurable: false）
- 属性不能被 `delete` 删除
- 标志不能再被修改（`configurable` 和 `enumerable` 永久锁定）
- 指向不可配置但 `writable` 为 `true` 时，`writable` 可以改为 `false`
- 一旦设为不可配置，无法撤销

### 全局封闭方法
- `Object.preventExtensions(obj)`：禁止添加新属性
- `Object.seal(obj)`：禁止添加/删除属性，所有属性 `configurable: false`
- `Object.freeze(obj)`：禁止添加/删除/修改属性，所有属性 `writable: false` + `configurable: false`
- 对应的检查方法：`isExtensible()`、`isSealed()`、`isFrozen()`

### 访问器属性（Getter/Setter）
两种对象属性类型：
- **数据属性**：直接存储值
- **访问器属性**：通过 getter/setter 函数间接访问

语法：
```js
let obj = {
  get propName() { return ...; },    // 读取时调用
  set propName(value) { ... }        // 赋值时调用
};
```
- 访问器属性没有 `value` 和 `writable`，取而代之的是 `get` 和 `set`
- 属性描述符中访问器描述符有：`get`、`set`、`enumerable`、`configurable`
- 可以只有 getter（只读属性），也可以只有 setter（只写属性）
- 使用场景：数据验证、计算属性、兼容性包装、封装内部状态

```js
// 通过 defineProperty 定义访问器
Object.defineProperty(obj, 'fullName', {
  get() { return `${this.name} ${this.surname}`; },
  set(value) {
    [this.name, this.surname] = value.split(' ');
  }
});
```

## 关键细节

### 数据属性 vs 访问器属性
- 描述符类型互斥：不能同时拥有 `value`/`writable` 和 `get`/`set`
- 通过 `defineProperty` 创建的属性，默认标志全为 `false`
- 字面量方式创建的属性，默认标志全为 `true`
- 访问器属性不能使用常规赋值语法批量覆盖

### 命名冲突与 getter/setter
- 直接给访问器属性赋值会触发 setter
- getter 中返回自身值会触发无限递归（需用不同内部属性名）

## 与其他系统的关系

- [[JavaScript 对象基础]] — 对象创建与属性访问的基础知识
- [[JavaScript 函数进阶]] — 装饰器模式常用于封装 getter/setter 行为
- [[JavaScript 数据类型]] — 数组、Map 等数据结构的属性控制
- 访问器属性是 Vue.js 响应式系统的核心机制之一

## 注意事项

- `configurable: false` 的操作不可逆，使用时需慎重
- `Object.defineProperty` 创建的新属性默认所有标志为 `false`，与字面量行为不同
- 严格模式下对只读属性赋值会报错，非严格模式静默忽略
- `for...in` 和 `Object.keys` 均不列出 Symbol 属性（即使 enumerable 为 true）
- `Object.freeze` 是浅冻结，嵌套对象内部属性仍可修改
- 避免在 getter 中执行副作用操作（违背 getter 语义）

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 对象基础]]
- [[JavaScript 函数进阶]]
- [[JavaScript 数据类型]]

## 原始来源

- [对象属性配置](raw/zh.javascript.info/1-js/07-object-properties/index.md)
- [属性标志和属性描述符](raw/zh.javascript.info/1-js/07-object-properties/01-property-descriptors/article.md)
- [属性的 getter 和 setter](raw/zh.javascript.info/1-js/07-object-properties/02-property-accessors/article.md)
