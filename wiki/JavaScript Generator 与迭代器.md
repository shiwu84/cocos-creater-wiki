---
title: JavaScript Generator 与迭代器
date: 2026-06-02
tags:
  - javascript
  - concept
aliases: []
---

# JavaScript Generator 与迭代器

> [!abstract] 摘要
> Generator 是可以暂停和恢复执行的函数，通过 `yield` 按需产出多个值。本章涵盖 generator 函数、双向通信、组合委托、异步迭代和异步 generator，以及它们在实际场景（如分页数据获取）中的应用。

## 核心概念

### Generator 函数
- 使用 `function*` 声明，调用时**不执行**函数体，而是返回一个 generator 对象
- 通过 `generator.next()` 恢复执行，遇到 `yield` 暂停并返回值
- `next()` 返回 `{ value: yieldedValue, done: boolean }`
- generator 是**可迭代的**（实现了 `Symbol.iterator`），可直接用于 `for..of`
- `for..of` 循环不会输出 `done: true` 时的 `value`，因此应使用 `yield` 而非 `return` 来产出最后一个值
- spread 语法 `...generator` 也适用
- 可使用 `*[Symbol.iterator]()` 作为简写实现对象的迭代器

### yield 双向通信
- `yield` 向外产出值，同时可以接收外部通过 `next(value)` 传入的值
- 第一次 `next()` 的参数会被忽略（此时还没有 yield 在等待值）
- 通信模式类似"乒乓球"：`next(value)` 传值进 generator → 从下一个 `yield` 产出值

### generator 组合（yield*）
- `yield* anotherGenerator` 将执行委托给另一个 generator，透明转发其产出的值
- 无需额外内存存储中间结果，适合构建组合型数据流

### generator.throw 和 generator.return
- `generator.throw(err)` 在 `yield` 位置抛出错误（可在 generator 内部用 `try...catch` 捕获）
- `generator.return(value)` 提前终止 generator，返回指定值

## 关键细节

### 异步迭代 (Async Iteration)
- 适用于值通过异步方式（如网络请求、定时器）产生的情况
- 实现 `Symbol.asyncIterator` 替代 `Symbol.iterator`
- `next()` 返回 `Promise`（而非普通值）
- 使用 `for await (let value of iterable)` 遍历
- Spread 语法、`for..of`（不带 await）不适用于异步迭代器

### 异步 Generator
- 声明方式：`async function*`
- 内部可使用 `await`，适用于异步获取数据后 `yield` 产出
- 可用作 `Symbol.asyncIterator` 的实现
- `generator.next()` 返回 promise

### 实际应用：分页数据获取
- 使用异步 generator 封装分页 API：内部处理分页逻辑，外部通过 `for await..of` 简洁遍历
- 隐藏底层请求细节，每次 yield 一条数据，自动请求下一页
- 适用于 GitHub API 等分页接口

## 与其他系统的关系
- [[JavaScript Promise 与异步]] 中 async/await 与异步 generator 高度相关
- [[JavaScript 模块]] 中动态 `import()` 返回 promise，可与异步迭代结合使用
- [[JavaScript 教程概述]] 的可迭代对象（Iterable）是 generator 的前置概念

## 注意事项
- generator 在现代 JavaScript 中使用较少，但在创建自定义迭代器时非常简洁
- 异步 generator 适合处理数据流（flows chunk-by-chunk），在 Web 开发中实用价值高
- `for..of` 不输出 `return` 返回的最终值——始终用 `yield` 产出所有需要输出的值
- generator 可以生成无限序列（需配合 `break` 或 `return` 终止循环）

## 相关页面
- [[JavaScript 教程概述]]
- [[JavaScript Promise 与异步]]
- [[JavaScript 模块]]

## 原始来源
- [Generator，高级 iteration](raw/zh.javascript.info/1-js/12-generators-iterators/index.md)
- [generator](raw/zh.javascript.info/1-js/12-generators-iterators/1-generators/article.md)
- [异步迭代和 generator](raw/zh.javascript.info/1-js/12-generators-iterators/2-async-iterators-generators/article.md)
