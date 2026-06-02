---
title: JavaScript 类
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases: []
---

# JavaScript 类

> [!abstract] 摘要
> `class` 是 JavaScript 中面向对象编程的现代语法糖，底层仍然基于原型继承。本章涵盖 class 基本语法、继承（extends/super）、静态成员、私有/受保护字段、扩展内建类、`instanceof` 类型检查和 Mixin 模式。

## 核心概念

### Class 本质
- `class` 在 JavaScript 中本质是函数（`typeof ClassName === "function"`）
- 类方法存储在 `ClassName.prototype` 上
- 与纯函数构造器相比，class 有重要差异：必须用 `new` 调用、方法不可枚举、自动严格模式

### Class 字段 (Class Fields)
- 类字段声明直接挂载在实例对象上，而非原型上
- 使用箭头函数声明类字段 `methodName = () => {}` 可以**永久绑定 `this`**，解决回调中丢失 `this` 的问题

### Getter/Setter 和计算属性
- 类中支持 `get` / `set` 语法，底层在 `ClassName.prototype` 上创建
- 支持计算属性名 `[expression]() {}`

## 关键细节

### extends 继承
- `class Child extends Parent` 建立原型链：`Child.prototype.__proto__ = Parent.prototype`
- `extends` 后可跟任意表达式，支持动态生成父类
- 通过 `super.method()` 调用父类方法，`super(...)` 调用父类构造器
- 派生类的 constructor **必须先调用 `super()`** 才能使用 `this`（因为派生构造器不会自动创建 `this` 对象）
- 箭头函数没有自己的 `super`，会从外部作用域获取

### `super` 与 ``HomeObject``
- 方法定义时会记录其 ``HomeObject``（定义所在的对象/类）
- `super` 通过 ``HomeObject`` 向上查找父原型，而非通过 `this`
- 只有简写方法语法 `method() {}`（非 `method: function() {}`）才会设置 ``HomeObject``
- 将有 `super` 的方法从一个对象复制到另一个对象会导致错误行为（``HomeObject`` 不可变）

### 类字段继承的特殊行为
- 父类构造器中访问的字段永远使用**父类的字段值**，而非子类重写的值
- 原因在于字段初始化顺序：基类在构造器前初始化字段，派生类在 `super()` 之后才初始化字段

### 静态属性和方法
- 用 `static` 关键字标记，直接挂载在类本身上
- 静态成员可被继承：`Child.__proto__ = Parent`
- 适用于工厂方法（如 `Article.createTodays()`）、比较方法、数据库操作等

### 封装：私有与受保护
- **受保护字段**：`_` 前缀约定，可被继承类访问，语言层面不强制执行
- **私有字段**：`#` 前缀，语言层面强制私有，不可从外部或继承类访问
- 私有字段不能用 `this['#name']` 访问，确保严格的语法隐私

### 扩展内建类
- 可以 `extends Array` 等方式扩展内建类
- 内建方法（如 `filter`、`map`）返回的是子类的实例，通过 `.constructor` 决定
- 通过 `static get [Symbol.species]()` 可定制内建方法返回的类型
- **内建类之间不继承静态方法**（如 `Array.keys()` 不存在），仅实例方法通过原型链继承

### instanceof 类型检查
- `obj instanceof Class` 检查 `Class.prototype` 是否在 `obj` 的原型链上
- 可通过 `static [Symbol.hasInstance](obj)` 自定义检查逻辑
- `Object.prototype.toString.call(obj)` 可获取更精确的类型字符串（`[object Type]`）
- `Symbol.toStringTag` 可自定义 `toString` 的输出标签

### Mixin 模式
- JavaScript 不支持多重继承，Mixin 通过将方法拷贝到原型实现行为组合
- 使用 `Object.assign(Class.prototype, mixin)` 混入方法
- Mixin 内部可以有自己的继承和 `super` 调用（通过 ``HomeObject``）
- 典型应用：EventMixin 为多个类提供事件订阅/触发能力

## 与其他系统的关系
- [[JavaScript 原型与继承]] 是 class 语法的底层实现基础
- [[JavaScript Promise 与异步]] 中 async class method 依赖 class 语法
- [[JavaScript 错误处理]] 中自定义 Error 类通过 extends Error 实现

## 注意事项
- 类方法之间不要加逗号（与对象字面量不同）
- Mixin 命名需谨慎避免覆盖现有方法
- `for..in` 不会遍历类方法（因为它们不可枚举）
- 派生类 constructor 务必先 `super()` 再访问 `this`

## 相关页面
- [[JavaScript 教程概述]]
- [[JavaScript 原型与继承]]
- [[JavaScript 错误处理]]
- [[JavaScript Promise 与异步]]
- [[Trait 系统]]

## 原始来源
- [类](raw/zh.javascript.info/1-js/09-classes/index.md)
- [Class 基本语法](raw/zh.javascript.info/1-js/09-classes/01-class/article.md)
- [类继承](raw/zh.javascript.info/1-js/09-classes/02-class-inheritance/article.md)
- [静态属性和静态方法](raw/zh.javascript.info/1-js/09-classes/03-static-properties-methods/article.md)
- [私有的和受保护的属性和方法](raw/zh.javascript.info/1-js/09-classes/04-private-protected-properties-methods/article.md)
- [扩展内建类](raw/zh.javascript.info/1-js/09-classes/05-extend-natives/article.md)
- [类检查："instanceof"](raw/zh.javascript.info/1-js/09-classes/06-instanceof/article.md)
- [Mixin 模式](raw/zh.javascript.info/1-js/09-classes/07-mixins/article.md)
