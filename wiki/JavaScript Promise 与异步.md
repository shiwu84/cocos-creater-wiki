---
title: JavaScript Promise 与异步
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases: []
---

# JavaScript Promise 与异步

> [!abstract] 摘要
> Promise 是 JavaScript 处理异步操作的核心机制，解决回调地狱问题。本章涵盖回调模式、Promise 基础、链式调用、错误处理、Promise API 静态方法、Promisification、微任务队列以及现代的 `async/await` 语法。

## 核心概念

### 回调模式与回调地狱
- 异步操作通过回调（callback）在完成时通知调用者
- Error-first callback 约定：`callback(error, result)`，错误优先传递给第一个参数
- 多层嵌套异步调用形成"回调地狱"（callback hell / pyramid of doom），代码难以维护和扩展

### Promise 基础
- Promise 是连接"生产者代码"（executor）和"消费者代码"（handler）的桥梁
- 状态转换：`pending` → `fulfilled`（resolve）或 `rejected`（reject），一旦 settled 不可逆
- `.then(resultFn, errorFn)` 注册成功和失败处理程序
- `.catch(errorFn)` 等价于 `.then(null, errorFn)`
- `.finally(cleanupFn)` 在 settled 后总是执行，无参数，返回值被忽略（除非抛出错误）
- 对已 settled 的 promise 添加处理程序会立即执行

### Promise 链 (Chaining)
- 每个 `.then` / `.catch` / `.finally` 返回一个新的 promise
- 处理程序返回的值（或 thenable/promise）会传递给链中下一个 `.then`
- 返回 promise 时，后续 `.then` 会等待其 settled 后再执行
- 链式调用是扁平的（向下增长），消除了回调金字塔

### Promise 静态方法
| 方法 | 行为 | 场景 |
|------|------|------|
| `Promise.all(promises)` | 全部 resolve 后才 resolve，任一 reject 立即 reject | 并行任务全部需要 |
| `Promise.allSettled(promises)` | 等待全部 settle，返回 `{status, value/reason}[]` | 需要所有结果，不因个别失败中断 |
| `Promise.race(promises)` | 返回第一个 settle 的结果 | 超时竞技 |
| `Promise.any(promises)` | 返回第一个 fulfilled 的结果，全部 reject 则 AggregateError | 多源竞速取最快 |
| `Promise.resolve(value)` | 创建已 resolved 的 promise | 统一返回值类型 |
| `Promise.reject(error)` | 创建已 rejected 的 promise | 统一错误处理 |

## 关键细节

### Promise 错误处理
- Promise executor 和处理程序周围有**隐式 `try..catch`**：抛出的错误自动转为 rejection
- `.catch` 放在链末尾可捕获链中任何地方的错误
- ``.catch` 处理后可继续 `.then`（返回正常值）
- `.catch` 通过 `throw` 可再次抛出，传递到下一个错误处理程序
- **未处理的 rejection**：没有 `.catch` 的 rejected promise 会触发全局 `unhandledrejection` 事件
- 浏览器：`window.addEventListener('unhandledrejection', ...)`

### 微任务队列 (Microtask Queue)
- Promise 处理程序 `.then`/`.catch`/`.finally` 始终是**异步**的
- 已 settled 的 promise 的处理程序在微任务队列中排队，当前代码执行完后才运行
- `unhandledrejection` 在微任务队列清空后检查——异步添加的 `.catch` 无法阻止它

### Promisification
- 将基于回调的函数转换为返回 promise 的函数
- 通用模式：包装函数返回 `new Promise(...)`，在回调中调用 `resolve`/`reject`
- Node.js 内建 `util.promisify`；第三方库如 `es6-promisify`
- 适用于只调用一次回调的函数；多次调用的回调不适合 promisify

### async/await
- `async` 函数始终返回 promise；非 promise 值自动包装
- `await` 暂停函数执行，等待 promise settled 后返回结果
- `await` 只能用在 `async` 函数内（或现代浏览器中的顶层 module 内）
- 错误处理：`await` 前的 promise reject 等价于 `throw`，可用 `try...catch` 捕获
- `async/await` 与 `Promise.all` 结合可实现并行等待
- class 中可声明 `async method()`
- `await` 支持 thenable 对象（有 `.then` 方法的对象）

## 与其他系统的关系
- [[JavaScript 错误处理]] 中 `try...catch` 模式在 async/await 中直接可用
- [[JavaScript Generator 与迭代器]] 中 async generator 是 generator 的异步版本
- [[JavaScript 模块]] 中 `import()` 返回 promise，支持动态模块加载

## 注意事项
- 始终在 promise 链末尾添加 `.catch` 或处理全局 `unhandledrejection`
- 忘记在 promise 链中 `return` 会导致链断裂
- `Promise.all` 中一个失败不会取消其他进行中的 promise
- `async/await` 基本替代了 `.then`/`.catch`，但 `.catch` 仍适用于最外层作用域或顶层代码

## 相关页面
- [[JavaScript 教程概述]]
- [[JavaScript 错误处理]]
- [[JavaScript Generator 与迭代器]]
- [[JavaScript 模块]]
- [[Rust 并发模型]]
- [[并发模型对比]]
- [[Linux 进程模型]]

## 原始来源
- [Promise，async/await](raw/zh.javascript.info/1-js/11-async/index.md)
- [简介：回调](raw/zh.javascript.info/1-js/11-async/01-callbacks/article.md)
- [Promise](raw/zh.javascript.info/1-js/11-async/02-promise-basics/article.md)
- [Promise 链](raw/zh.javascript.info/1-js/11-async/03-promise-chaining/article.md)
- [使用 promise 进行错误处理](raw/zh.javascript.info/1-js/11-async/04-promise-error-handling/article.md)
- [Promise API](raw/zh.javascript.info/1-js/11-async/05-promise-api/article.md)
- [Promisification](raw/zh.javascript.info/1-js/11-async/06-promisify/article.md)
- [微任务（Microtask）](raw/zh.javascript.info/1-js/11-async/07-microtask-queue/article.md)
- [async/await](raw/zh.javascript.info/1-js/11-async/08-async-await/article.md)
