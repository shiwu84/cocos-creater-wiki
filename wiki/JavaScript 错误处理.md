---
title: JavaScript 错误处理
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases: []
---

# JavaScript 错误处理

> [!abstract] 摘要
> JavaScript 通过 `try...catch...finally` 结构实现运行时错误捕获，通过 `throw` 操作符生成自定义错误，并支持基于 `Error` 类构建层次化的自定义错误体系。还包括全局错误处理和"包装异常"等高级模式。

## 核心概念

### try...catch 结构
- `try` 块中发生错误时，执行立即停止，控制权转至 `catch(err)` 块
- `catch` 可以省略参数：`catch { ... }`（当不需要 error 详情时）
- `finally` 块**总是执行**：无论是正常退出、return 还是 throw
- `try...catch` 只捕获**运行时错误**（runtime errors），解析时错误（语法错误）无法捕获
- `try...catch` **同步工作**：异步操作（如 `setTimeout`）内部的错误不会被外部的 `try...catch` 捕获

### Error 对象属性
- `name` — 错误类型名称（如 `ReferenceError`、`SyntaxError`）
- `message` — 人类可读的错误描述
- `stack` — 调用栈字符串（非标准但广泛支持）

## 关键细节

### throw 与内建 Error 类型
- `throw` 可以抛出任意值，但推荐使用 Error 对象（或继承自 Error）
- 内建错误类型：`Error`、`SyntaxError`、`ReferenceError`、`TypeError`、`URIError` 等
- 创建：`new Error(message)` / `new SyntaxError(message)`

### 再次抛出 (Rethrowing)
- `catch` 只应处理它能处理的错误，未知错误应重新 `throw`
- 通过 `err instanceof ErrorType` 或 `err.name` 判断错误类型
- 配合外部 `try...catch` 可形成多层错误处理架构

### 自定义 Error 类
- 通过 `class MyError extends Error` 实现，constructor 中必须调用 `super(message)`
- 自定义 `name` 属性（或用 `this.name = this.constructor.name` 自动化）
- 可创建错误层次结构：`ValidationError` → `PropertyRequiredError`
- `instanceof` 检查兼容派生类，比 `err.name` 比较更具扩展性

### 包装异常 (Wrapping Exceptions)
- 是一种 OOP 模式：将低级别异常包装为更高级别的抽象（如 `ReadError`）
- 原始错误通过 `cause` 属性保留引用
- 使外部调用者只需检查一种错误类型，降低耦合
- 低级别错误内容仍可通过 `err.cause` 获取

### 全局错误处理
- 浏览器：`window.onerror` 或 `window.addEventListener('error', ...)`
- Node.js：`process.on("uncaughtException")`
- 全局处理主要用于日志记录和错误上报，而非恢复执行

## 与其他系统的关系
- [[JavaScript Promise 与异步]] 中 `.catch()` 是 Promise 版的错误处理，也有隐式的 try...catch 包裹
- [[JavaScript 类]] 中的 extends 用于构建自定义 Error 层次结构
- [[JavaScript 教程概述]] 涵盖了 JavaScript 基础知识全景

## 注意事项
- `catch` 中通过 `instanceof` 而非 `err.name` 判断类型更具可扩展性
- `finally` 中的 `return` 会覆盖 `try` 中的返回值
- 不要用 `try...catch` 替代正常的条件判断——它是为真正的异常情况设计的
- 异步回调中的错误需要在回调内部处理，不会被外层 try...catch 捕获

## 相关页面
- [[JavaScript 教程概述]]
- [[JavaScript Promise 与异步]]
- [[JavaScript 类]]
- [[所有权与借用]]

## 原始来源
- [错误处理](raw/zh.javascript.info/1-js/10-error-handling/index.md)
- [错误处理，"try...catch"](raw/zh.javascript.info/1-js/10-error-handling/1-try-catch/article.md)
- [自定义 Error，扩展 Error](raw/zh.javascript.info/1-js/10-error-handling/2-custom-errors/article.md)
