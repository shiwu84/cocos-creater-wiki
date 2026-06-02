---
title: JavaScript 模块
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases: []
---

# JavaScript 模块

> [!abstract] 摘要
> ES6 模块是 JavaScript 语言级的模块系统，通过 `export` 和 `import` 实现跨文件的代码组织与功能共享。本章涵盖模块核心功能（严格模式、模块级作用域、单次加载）、导出/导入的各种语法形式、动态导入，以及构建工具的作用。

## 核心概念

### 模块基本特性
- 一个文件就是一个模块
- 浏览器中通过 `<script type="module">` 启用模块
- 模块始终运行在**严格模式**（use strict）下
- 每个模块拥有独立的**顶级作用域**：变量和函数不会泄漏到全局
- 模块代码**仅在第一次导入时执行**：多次导入同一模块只执行一次，导出内容在导入之间共享
- 模块内顶层 `this` 是 `undefined`（非模块脚本中 `this` 是全局对象）

### 浏览器模块行为
- 模块脚本默认是**延迟（deferred）**的：不阻塞 HTML 解析，等文档完全加载后才执行
- 外部模块脚本相同 `src` 只执行一次
- 跨源加载模块脚本需要 CORS header (`Access-Control-Allow-Origin`)
- 裸模块（bare module，如 `import "lodash"`）在浏览器中不被允许
- `async` 特性也适用于内联模块脚本
- `nomodule` 特性可为旧浏览器提供回退脚本

### 模块配置模式
- 利用"单次执行"特性：导出配置对象，首次导入时初始化，所有导入共享配置
- 适用于身份验证凭证、全局配置等场景

### import.meta
- `import.meta` 包含当前模块的元信息
- 在浏览器中 `import.meta.url` 为当前脚本 URL

## 关键细节

### 导出 (export) 语法
| 形式 | 说明 |
|------|------|
| `export class/function/variable` | 声明时导出 |
| `export { name1, name2 }` | 声明后集中导出 |
| `export { name as alias }` | 导出时重命名 |
| `export default class/function/...` | 默认导出（每模块最多一个） |
| `export { name as default }` | 将命名导出转为默认导出 |
| `export { name } from './module'` | 重新导出（当前模块不可用） |
| `export * from './module'` | 重新导出全部命名导出（不含 default） |
| `export { default } from './module'` | 重新导出默认导出 |

### 导入 (import) 语法
| 形式 | 说明 |
|------|------|
| `import { name } from './module'` | 命名导入 |
| `import { name as alias } from './module'` | 导入并重命名 |
| `import defaultName from './module'` | 默认导入（无花括号） |
| `import { default as name } from './module'` | 将默认导出作为命名导出导入 |
| `import * as obj from './module'` | 全部导入为命名空间对象 |
| `import './module'` | 仅执行模块（不导入任何导出） |

### 命名导出 vs 默认导出
- **命名导出**：强制使用正确名称，利于 tree-shaking，跨模块名称一致
- **默认导出**：导入者随意命名，每个文件只导出一个实体，适合"一个模块一件事"
- 团队间倾向使用命名导出以保持一致性

### 动态导入
- `import(modulePath)` 返回 promise，可在代码任意位置调用
- 支持条件加载、按需加载、运行时动态路径
- 结合 `await` 使用：`const module = await import('./module.js')`
- 默认导出通过 `.default` 属性访问
- 不需要 `script type="module"`，可在普通脚本中使用

### 构建工具
- 生产环境中常用 Webpack 等打包工具替换原生模块导入
- 功能：依赖分析、打包合并、tree-shaking（移除未使用代码）、语法转译（Babel）、代码压缩
- 打包后输出常规 `<script>` 文件，不再需要 `type="module"`

## 与其他系统的关系
- [[JavaScript Promise 与异步]] 中 `import()` 动态导入返回 promise
- [[JavaScript 类]] 中的 class 常作为模块的默认导出
- [[JavaScript 教程概述]] 覆盖了脚本加载的其他前置知识

## 注意事项
- 模块必须通过 HTTP(S) 加载，本地 `file://` 协议下无法工作
- `import`/`export` 语句只能在模块顶层使用，不能放在条件块或函数中（静态导入）
- 导入语句位置不影响执行（通常放顶部只是约定），但会被整体提升
- `export default` 重新导出需要特殊语法：`export { default as name } from './module'`
- `export *` 不会重新导出默认导出

## 相关页面
- [[JavaScript 教程概述]]
- [[JavaScript Promise 与异步]]
- [[JavaScript 类]]
- [[Trait 系统]]
- [[TypeScript 概述]]

## 原始来源
- [模块](raw/zh.javascript.info/1-js/13-modules/index.md)
- [模块 (Module) 简介](raw/zh.javascript.info/1-js/13-modules/01-modules-intro/article.md)
- [导出和导入](raw/zh.javascript.info/1-js/13-modules/02-import-export/article.md)
- [动态导入](raw/zh.javascript.info/1-js/13-modules/03-modules-dynamic-imports/article.md)
