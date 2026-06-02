---
title: JavaScript 函数进阶
date: 2026-06-02
tags:
  - javascript
  - concept
aliases:
  - JS 函数进阶
---

# JavaScript 函数进阶

> [!abstract] 摘要
> 本章深入探讨 JavaScript 函数的进阶特性：递归与调用栈、Rest 参数与 Spread 语法、闭包与词法环境、`var` 的变量提升、全局对象、函数对象的属性（name/length/自定义）、`new Function` 动态创建、`setTimeout/setInterval` 调度、call/apply/bind 的上下文控制，以及箭头函数的特殊性。

## 核心概念

### 递归与堆栈
- 递归：函数调用自身，将复杂任务分解为更简单的同类任务
- 两个要素：**基础条件**（递归终止）和 **递归步骤**（简化问题）
- 递归深度受限于 JavaScript 引擎（通常 ~10000），超过会栈溢出
- 任何递归都可用迭代循环重写，递归通常更短更易读
- 尾递归优化（TCO）：规范支持但多数引擎未实现
- 典型应用：树遍历、分治算法、数学计算（阶乘、斐波那契等）

### Rest 参数与 Spread 语法
- **Rest 参数**：`function f(...args)` 将剩余参数收集为数组（必须是最后一个参数）
- **Spread 语法**：`...arr` 在函数调用或字面量中将可迭代对象展开为元素列表
- `arguments` 变量：旧式的类数组参数集合（箭头函数没有），推荐用 Rest 参数替代
- Spread 可用于合并数组 `[...arr1, ...arr2]` 和浅拷贝对象 `{...obj}`

### 闭包（Closure）
- **词法环境**：每个运行的函数、代码块以及脚本作为整体都有一个内部隐藏的关联对象
- 词法环境由 **环境记录**（存储局部变量）和 **外部词法环境引用** 组成
- **闭包**：函数能记住并访问其词法作用域，即使函数在其词法作用域之外执行
- 每次函数调用创建新的词法环境
- 所有函数在诞生时（创建时）就通过隐藏属性 ``Environment`` 记住了创建位置的词法环境
- 嵌套函数可以访问外部变量，但外部不能访问内部变量
- 闭包常用于：数据封装/私有变量、回调函数、柯里化、模块模式

### var 与变量提升
- `var` 没有块级作用域，只有函数级或全局作用域
- `var` 变量会被提升（hoisting）到函数/全局作用域顶部，声明被提升但赋值保留原位
- `var` 可以重复声明，`let/const` 不行
- 立即执行函数表达式（IIFE：`(function() { ... })()`）在 `let` 出现前用于模拟块级作用域
- 全局作用域中 `var` 声明的变量成为 `window`/`globalThis` 的属性，`let/const` 不会

### 全局对象
- 浏览器：`window`，Node.js：`global`，通用：`globalThis`
- 可用于检测功能是否存在（如 `if (window.XMLHttpRequest)`）
- 避免给全局对象添加过多属性，以免命名冲突

### 函数对象与属性
- 函数是对象，可拥有属性
- `name`：函数名（自动推断，包括默认导出、对象方法名等）
- `length`：函数声明时的参数个数（不包含 Rest 参数）
- 自定义属性：可用于缓存/计次（如 `func.counter`）
- 命名函数表达式（NFE）：内部名只在该函数作用域内可见，可用于自引用

### new Function 语法
- `let func = new Function([arg1, arg2, ...], functionBody)`
- 在全局作用域创建，无法访问创建位置的局部变量（只能访问全局变量）
- 用途：动态编译代码（如从服务器获取函数体），但需注意安全性

### 调度：setTimeout 和 setInterval
- `setTimeout(func, delay, ...args)`：延迟执行一次
- `setInterval(func, delay, ...args)`：周期性重复执行
- 清除：`clearTimeout(timerId)` / `clearInterval(timerId)`
- 嵌套 `setTimeout` 比 `setInterval` 更灵活（可动态调整间隔、确保前一次执行完毕）
- 零延时 `setTimeout(f, 0)`：在当前代码执行完毕后尽快执行
- 浏览器中最小延迟为 4ms（嵌套到一定层数后）

### call、apply、bind
- **`func.call(context, arg1, arg2, ...)`**：以指定 `this` 调用函数，参数逐个传递
- **`func.apply(context, args)`**：以指定 `this` 调用函数，参数作为类数组传递
- **`func.bind(context, ...args)`**：返回新函数，永久绑定 `this` 和部分参数（偏函数）
- 装饰器模式：包装原函数，在不修改原函数的情况下增强功能（缓存、日志、限流）
- `call`/`apply` 对箭头函数无效（箭头函数无自己的 `this`）
- `bind` 返回的绑定函数无法再次绑定 `this`

### 箭头函数再探
- 无自己的 `this`：从外层作用域继承，`bind/call/apply` 无效
- 无 `arguments` 变量
- 不能用作构造函数（不能 `new`）
- 无 `super`（类继承相关）
- 适合短小的回调和需要保留外层 `this` 的场景

## 关键细节

### 装饰器模式
```js
function cachingDecorator(func) {
  let cache = new Map();
  return function(x) {                    // 包装器
    if (cache.has(x)) return cache.get(x);
    let result = func.call(this, x);      // 传递 this
    cache.set(x, result);
    return result;
  };
}
```
- 分离关注点：缓存逻辑与业务逻辑解耦
- 可组合多个装饰器
- 需正确传递 `this`（用 `call`/`apply`）和参数（用 Rest/Spread）

### 闭包内存注意
- 闭包会维持对外部变量的引用，阻止垃圾回收
- 不再需要闭包时应解除引用（设为 `null`）
- V8 等引擎会优化未被嵌套函数使用的变量

## 与其他系统的关系

- [[JavaScript 基础知识]] — 函数声明、参数、返回值等基础
- [[JavaScript 对象基础]] — `this` 和方法的调用上下文
- [[JavaScript 对象属性]] — 属性的 getter/setter 与函数式编程
- 装饰器模式是面向切面编程（AOP）在 JS 中的轻量实现

## 注意事项

- 递归时确保有终止条件，否则栈溢出
- `arguments` 是类数组，不能用数组方法，需转为数组
- `setInterval` 的实际间隔可能因执行耗时而不准确
- 箭头函数不适合需要动态 `this` 的场景（如事件处理器中需要 `this` 指向 DOM 元素）
- `new Function` 创建的闭包指向全局作用域，可能造成内存和安全隐患
- `bind` 多次调用无效，仅第一次的上下文绑定生效

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 基础知识]]
- [[JavaScript 对象基础]]
- [[JavaScript 代码质量]]
- [[JavaScript 对象属性]]

## 原始来源

- [函数进阶内容](raw/zh.javascript.info/1-js/06-advanced-functions/index.md)
- [递归和堆栈](raw/zh.javascript.info/1-js/06-advanced-functions/01-recursion/article.md)
- [Rest 参数与 Spread 语法](raw/zh.javascript.info/1-js/06-advanced-functions/02-rest-parameters-spread/article.md)
- [变量作用域，闭包](raw/zh.javascript.info/1-js/06-advanced-functions/03-closure/article.md)
- [var](raw/zh.javascript.info/1-js/06-advanced-functions/04-var/article.md)
- [全局对象](raw/zh.javascript.info/1-js/06-advanced-functions/05-global-object/article.md)
- [函数对象，NFE](raw/zh.javascript.info/1-js/06-advanced-functions/06-function-object/article.md)
- ["new Function" 语法](raw/zh.javascript.info/1-js/06-advanced-functions/07-new-function/article.md)
- [调度：setTimeout 和 setInterval](raw/zh.javascript.info/1-js/06-advanced-functions/08-settimeout-setinterval/article.md)
- [装饰器模式和转发，call/apply](raw/zh.javascript.info/1-js/06-advanced-functions/09-call-apply-decorators/article.md)
- [函数绑定](raw/zh.javascript.info/1-js/06-advanced-functions/10-bind/article.md)
- [深入理解箭头函数](raw/zh.javascript.info/1-js/06-advanced-functions/12-arrow-functions/article.md)
