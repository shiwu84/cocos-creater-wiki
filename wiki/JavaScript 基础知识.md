---
title: JavaScript 基础知识
date: 2026-06-02
tags:
  - javascript
  - concept
aliases:
  - JS 基础
  - JS 语法基础
---

# JavaScript 基础知识

> [!abstract] 摘要
> 本章系统讲解 JavaScript 的语法基础：在 HTML 中插入脚本、变量声明（`let`/`const`/`var`）、八种数据类型、类型转换、运算符、条件分支、循环结构，以及函数的声明与使用。是所有后续学习的必要前提。

## 核心概念

### 脚本标签与加载
- 使用 `<script>` 标签将 JavaScript 插入 HTML 文档的任意位置
- 外部脚本通过 `<script src="path/to/script.js">` 加载，有 `src` 时标签内部代码会被忽略
- 外部脚本的优势：浏览器缓存机制，只需下载一次
- 现代标记中 `type` 和 `language` 特性不再必需

### 变量声明
三种声明方式：
- **`let`** — 现代变量声明方式，块级作用域
- **`const`** — 声明常量，值不可修改；大写命名常用于硬编码值（如 `COLOR_RED`）
- **`var`** — 老旧方式，函数级作用域，存在变量提升

命名规则：仅含字母、数字、`$`、`_`，首字符非数字；采用驼峰式 `camelCase`；保留字不可用作变量名。

### 八种数据类型
- **7 种原始类型**：
  - `number`：整数和浮点数，±(2^53-1) 范围内安全；特殊值 `Infinity`、`NaN`
  - `bigint`：任意长度整数，后缀 `n`（如 `123n`）
  - `string`：三种引号（单引号、双引号、反引号）；反引号支持 `${...}` 模板嵌入
  - `boolean`：`true` / `false`
  - `null`：独立类型，表示"无"或"值未知"
  - `undefined`：独立类型，表示"未赋值"
  - `symbol`：唯一标识符
- **1 种引用类型**：`object` — 存储数据集合和更复杂实体

JavaScript 是动态类型语言，变量可在运行中更改类型。用 `typeof` 运算符检查类型，注意 `typeof null` 返回 `"object"`（历史遗留错误）。

### 运算符与比较
- 数学运算符：`+`、`-`、`*`、`/`、`%`、`**`
- 比较运算符：`>`、`<`、`>=`、`<=`、`==`（带类型转换）、`===`（严格相等）
- 字符串比较按 Unicode 逐字符进行
- 逻辑运算符：`||`（或）、`&&`（与）、`!`（非）、`??`（空值合并）

### 条件与循环
- `if...else`、`else if` 条件分支
- 三元运算符 `condition ? value1 : value2`
- `switch` 多条件分支
- 循环：`while`、`do...while`、`for(;;)`
- 跳出：`break`、`continue`（支持标签）

### 函数基础
- 函数声明：`function name(param1, param2) { ... }`
- 局部变量在函数内可见，函数可访问并修改外部变量（同名局部变量会遮蔽外部变量）
- 参数传递为值传递（复制），默认参数：`function f(param = defaultValue)`
- `return` 返回值，无 `return` 时返回 `undefined`；`return` 后不可直接换行
- 函数命名：动词前缀（`get`、`calc`、`create`、`check`、`show`），一个函数一个行为
- 函数表达式：`let f = function() { ... }`（可匿名）
- 箭头函数：`let f = (a, b) => a + b`，简洁，无自己的 `this`

## 关键细节

### 类型转换
- **字符串转换**：`String(value)` 或 `"" + value`
- **数字转换**：`Number(value)`，非数字返回 `NaN`
- **布尔转换**：假值 = `0`、`""`、`null`、`undefined`、`NaN`；其他为真值

### 空值合并运算符 `??`
```js
let result = a ?? b;  // 当 a 为 null/undefined 时返回 b
```
与 `||` 的区别：`??` 仅对 `null`/`undefined` 触发，不将 `0`、`""` 视为假值。

### 严格模式
在脚本顶部添加 `"use strict"` 启用严格模式，禁止未声明的变量赋值等不安全操作。

## 与其他系统的关系

- [[JavaScript 入门]] — 前置章节，介绍开发环境
- [[JavaScript 代码质量]] — 代码风格与调试
- [[JavaScript 对象基础]] — 对象类型的深入学习
- [[JavaScript 数据类型]] — 数据类型的深入研究
- [[JavaScript 函数进阶]] — 函数的高级特性

## 注意事项

- 一行声明一个变量，提高可读性
- 减少全局变量的使用，优先使用函数内的局部变量
- 变量应先声明再使用，避免隐式全局变量
- `typeof null === "object"` 是语言设计错误，不要基于此做类型判断
- 数学运算在 JS 中是安全的：除以 0 返回 `Infinity`，不会崩溃
- `NaN` 具有粘性：任何与 `NaN` 的运算都返回 `NaN`

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 入门]]
- [[JavaScript 对象基础]]
- [[JavaScript 数据类型]]
- [[JavaScript 函数进阶]]

## 原始来源

- [基础知识](raw/zh.javascript.info/1-js/02-first-steps/index.md)
- [Hello, world!](raw/zh.javascript.info/1-js/02-first-steps/01-hello-world/article.md)
- [代码结构](raw/zh.javascript.info/1-js/02-first-steps/02-structure/article.md)
- [现代模式 "use strict"](raw/zh.javascript.info/1-js/02-first-steps/03-strict-mode/article.md)
- [变量](raw/zh.javascript.info/1-js/02-first-steps/04-variables/article.md)
- [数据类型](raw/zh.javascript.info/1-js/02-first-steps/05-types/article.md)
- [交互：alert、prompt、confirm](raw/zh.javascript.info/1-js/02-first-steps/06-alert-prompt-confirm/article.md)
- [类型转换](raw/zh.javascript.info/1-js/02-first-steps/07-type-conversions/article.md)
- [基础运算符](raw/zh.javascript.info/1-js/02-first-steps/08-operators/article.md)
- [值的比较](raw/zh.javascript.info/1-js/02-first-steps/09-comparison/article.md)
- [条件分支：if 和 '?'](raw/zh.javascript.info/1-js/02-first-steps/10-ifelse/article.md)
- [逻辑运算符](raw/zh.javascript.info/1-js/02-first-steps/11-logical-operators/article.md)
- [空值合并运算符 '??'](raw/zh.javascript.info/1-js/02-first-steps/12-nullish-coalescing-operator/article.md)
- [循环：while 和 for](raw/zh.javascript.info/1-js/02-first-steps/13-while-for/article.md)
- ["switch" 语句](raw/zh.javascript.info/1-js/02-first-steps/14-switch/article.md)
- [函数](raw/zh.javascript.info/1-js/02-first-steps/15-function-basics/article.md)
- [函数表达式](raw/zh.javascript.info/1-js/02-first-steps/16-function-expressions/article.md)
- [箭头函数，基础知识](raw/zh.javascript.info/1-js/02-first-steps/17-arrow-functions-basics/article.md)
- [JavaScript 特性](raw/zh.javascript.info/1-js/02-first-steps/18-javascript-specials/article.md)
