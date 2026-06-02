---
title: JavaScript 教程概述
date: 2026-06-02
tags:
  - type/overview
  - layer/js
  - status/stable
aliases:
  - 现代 JavaScript 教程
---

# JavaScript 教程概述

> [!abstract] 摘要
> 《现代 JavaScript 教程》中文版是 React 官方文档和 MDN 共同推荐的前端教程，覆盖从语言基础到浏览器 API 的完整知识体系。本 Wiki 作为该教程的二次知识整理，按章节构建结构化页面。

## 教程结构

教程共 9 大部分，从 JavaScript 语言核心到浏览器环境和特定主题：

### 第一部分：JavaScript 语言

14 个章节，涵盖 JS 语言的完整知识体系：

| 章节 | 内容 | 原始来源 |
|------|------|----------|
| 简介 | 语言特性、开发环境、手册规范 | [introduction](raw/zh.javascript.info/1-js/01-getting-started/index.md) |
| 基础知识 | 变量、类型、运算符、条件、循环、函数 | [first-steps](raw/zh.javascript.info/1-js/02-first-steps/index.md) |
| 代码质量 | 调试、风格指南、注释、测试、Polyfill | [code-quality](raw/zh.javascript.info/1-js/03-code-quality/index.md) |
| Object 基础 | 对象、引用、方法、构造器、Symbol | [object-basics](raw/zh.javascript.info/1-js/04-object-basics/index.md) |
| 数据类型 | 数字、字符串、数组、Map/Set、WeakMap/WeakSet、JSON | [data-types](raw/zh.javascript.info/1-js/05-data-types/index.md) |
| 函数进阶 | 递归、闭包、箭头函数、装饰器、调度、bind | [advanced-functions](raw/zh.javascript.info/1-js/06-advanced-functions/index.md) |
| 对象属性 | 属性标志/描述符、getter/setter | [object-properties](raw/zh.javascript.info/1-js/07-object-properties/index.md) |
| 原型与继承 | 原型链、F.prototype、原生原型 | [prototypes](raw/zh.javascript.info/1-js/08-prototypes/index.md) |
| 类 | class 语法、继承、静态、私有成员、Mixin | [classes](raw/zh.javascript.info/1-js/09-classes/index.md) |
| 错误处理 | try-catch、自定义错误 | [error-handling](raw/zh.javascript.info/1-js/10-error-handling/index.md) |
| Promise/异步 | Promise 链、async/await、微任务 | [async](raw/zh.javascript.info/1-js/11-async/index.md) |
| Generator/迭代器 | Generator、异步迭代 | [generators-iterators](raw/zh.javascript.info/1-js/12-generators-iterators/index.md) |
| 模块 | import/export、动态导入 | [modules](raw/zh.javascript.info/1-js/13-modules/index.md) |
| 杂项 | Proxy、Eval、柯里化、BigInt | [misc](raw/zh.javascript.info/1-js/99-js-misc/index.md) |

### 第二部分：浏览器 — 文档、事件、接口

5 个章节，覆盖浏览器 DOM 和事件系统：

| 章节 | 内容 |
|------|------|
| Document | DOM 遍历、搜索、修改、样式、尺寸滚动、坐标 |
| 事件 | 事件冒泡/捕获、委托、默认行为、自定义事件 |
| 事件细节 | 鼠标、键盘、指针事件、拖放、滚动 |
| 表单与控件 | 表单属性、验证、提交、焦点 |
| 数据加载 | 脚本、资源加载、DOMContentLoaded |

### 第三部分：Frame 和 Window

跨窗口通信、iframe 操作、跨域策略。

### 第四部分：二进制数据

ArrayBuffer、TypedArray、TextDecoder/Encoder、Blob、FileReader。

### 第五部分：网络请求

Fetch、FormData、CORS、WebSocket、Server-Sent Events、长轮询。

### 第六部分：浏览器数据存储

Cookie、LocalStorage、SessionStorage、IndexedDB。

### 第七部分：动画

CSS 动画、JavaScript 动画、requestAnimationFrame。

### 第八部分：Web Components

自定义元素、Shadow DOM、模板元素、插槽。

### 第九部分：正则表达式

模式、字符类、量词、贪婪/惰性、捕获组、反向引用。

## 核心概念

### JavaScript 语言特性
- **动态类型** — 变量无固定类型，同变量可赋不同类型值
- **原型继承** — 基于原型链的对象继承模型（非传统类继承）
- **一等函数** — 函数可作为值传递、赋值、返回
- **事件循环** — 单线程 + 异步任务队列（宏任务/微任务）
- **闭包** — 函数可访问其外部词法环境中的变量

### 浏览器环境
- **DOM** — 文档对象模型，页面的结构化表示
- **BOM** — 浏览器对象模型（navigator、location、history 等）
- **同源策略** — 安全限制，不同源的标签页/窗口隔离
- **Web API** — 浏览器提供的各种接口（Fetch、Storage、Animation 等）

## 与其他系统的关系

- [[Cocos Creator 概述]] — Cocos 使用 TypeScript（JavaScript 超集）作为脚本语言
- [[脚本系统]] — Cocos Creator 的脚本基于 TypeScript/JavaScript 生态
- [[Jujutsu VCS]] — 教程仓库使用 Jujutsu + Git 管理

## 注意事项

- 本教程涵盖 174 篇详细文章和配套练习，Wiki 页面为概念层级整理
- JavaScript 规范（ECMAScript）持续演进，教程聚焦现代特性（ES6+）
- 教程重点关注浏览器环境，Node.js 服务端用法涉及较少
- TypeScript 是该教程推荐了解的 JavaScript 超集，提供类型系统

## 相关页面

### 语言核心（Part 1）
- [[JavaScript 入门]] — 语言特性、开发环境、规范
- [[JavaScript 基础知识]] — 变量、类型、运算符、循环、函数
- [[JavaScript 代码质量]] — 调试、风格、测试、Polyfill
- [[JavaScript 对象基础]] — 对象、引用、构造器、Symbol
- [[JavaScript 数据类型]] — 数字、字符串、数组、Map/Set、JSON
- [[JavaScript 函数进阶]] — 闭包、递归、装饰器、调度
- [[JavaScript 对象属性]] — 属性标志、getter/setter
- [[JavaScript 原型与继承]] — 原型链、F.prototype
- [[JavaScript 类]] — class 语法、继承、Mixin
- [[JavaScript 错误处理]] — try-catch、自定义错误
- [[JavaScript Promise 与异步]] — Promise 链、async/await、微任务
- [[JavaScript Generator 与迭代器]] — Generator、异步迭代
- [[JavaScript 模块]] — ES6 模块、动态导入
- [[JavaScript 正则表达式]] — 模式、量词、捕获组

### 浏览器 API（Part 2-3）
- [[JavaScript Document]] — DOM 遍历、修改、样式
- [[JavaScript 事件]] — 事件冒泡、委托、自定义事件
- [[JavaScript 事件细节]] — 鼠标、键盘、拖放
- [[JavaScript 表单与控件]] — 表单验证、提交
- [[JavaScript 数据加载]] — 脚本加载、页面生命周期
- [[JavaScript Frame 与 Window]] — 跨窗口通信、iframe

### 进阶主题（Part 4-9）
- [[JavaScript 二进制数据]] — ArrayBuffer、Blob、File
- [[JavaScript 网络请求]] — Fetch、WebSocket、CORS
- [[JavaScript 浏览器存储]] — Cookie、LocalStorage、IndexedDB
- [[JavaScript 动画]] — CSS/JS 动画、requestAnimationFrame
- [[JavaScript Web Components]] — 自定义元素、Shadow DOM

### 跨领域连接
- [[TypeScript 概述]] — JS 的类型超集
- [[Cocos Creator 概述]] — Cocos 使用 TypeScript/JS 作脚本语言
- [[软件工程概述]] — JS 生态中的工程实践
- [[Jujutsu VCS]] — 本教程仓库使用 Jujutsu + Git 管理

## 原始来源

- [JavaScript 教程目录](raw/zh.javascript.info/1-js/index.md)
- [浏览器文档接口](raw/zh.javascript.info/2-ui/index.md)
- [JavaScript 简介](raw/zh.javascript.info/1-js/01-getting-started/1-intro/article.md)
