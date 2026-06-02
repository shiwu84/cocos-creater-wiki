---
title: JavaScript 入门
date: 2026-06-02
tags:
  - javascript
  - concept
aliases:
  - JS 简介
---

# JavaScript 入门

> [!abstract] 摘要
> JavaScript 最初为"使网页更生动"而创建，如今已成为全平台通用的编程语言。本章介绍 JavaScript 的特性、浏览器能力边界、开发环境（编辑器与开发者控制台）以及语言规范与手册。

## 核心概念

### JavaScript 是什么
- 一种脚本语言，可直接写在 HTML 中，页面加载时自动执行
- 不需要特殊的准备或编译即可运行，以纯文本形式提供和执行
- 最初叫 "LiveScript"，后因 Java 的流行改名 JavaScript，现已有独立语言规范 ECMAScript
- 由浏览器内嵌的 JavaScript 引擎（V8、SpiderMonkey 等）解析、编译、执行

### 浏览器中的能力
- 操作 DOM：添加/修改 HTML、样式
- 响应用户行为：鼠标点击、指针移动、键盘按键
- 网络通信：AJAX、COMET 技术，下载/上传文件
- 客户端存储：Cookie、LocalStorage
- 受同源策略限制：不同标签页/窗口通常不能互相访问

### 浏览器安全限制
- 不能直接读写硬盘文件，需通过用户行为触发（如 `<input>` 文件选择）
- 访问摄像头/麦克风需明确用户许可
- 同源策略：不同源的标签页不能互相通信
- 从其他域获取数据需服务器端明确允许（CORS）

## 关键细节

### JavaScript 引擎
| 引擎 | 浏览器 |
|------|--------|
| V8 | Chrome、Opera、Edge |
| SpiderMonkey | Firefox |
| JavaScriptCore/Nitro | Safari |

引擎工作流程：解析脚本 → 编译为机器语言 → 机器码执行（运行时持续优化）。

### 开发者控制台
- Chrome/Edge：`F12`，Mac 用 `Cmd+Opt+J`
- Firefox：`F12`
- Safari：需先在偏好设置中开启"开发菜单"，用 `Cmd+Opt+C` 打开
- 功能：查看错误、执行 JavaScript 命令、多行输入（`Shift+Enter` 换行）

### JavaScript 上层语言
许多语言可编译为 JavaScript：
- **TypeScript**：添加严格数据类型，由微软开发
- **CoffeeScript**：语法糖，更简洁的语法
- **Flow**：添加数据类型，由 Facebook 开发
- **Dart**：独立语言，可编译为 JS
- **Kotlin**：可编译为 JS 运行在浏览器和 Node

## 与其他系统的关系

- [[JavaScript 基础知识]] — 接续本章的语言基础学习
- [[JavaScript 代码质量]] — 调试和代码风格
- 与 HTML/CSS 完全集成，是创建浏览器界面的最广泛工具
- JavaScript 超越浏览器：Node.js 支持服务端编程，也可用于移动端开发

## 注意事项

- JavaScript 与 Java 现在没有任何关系，是两门完全独立的语言
- `typeof` 和 `language` 特性在 `<script>` 标签中不再必需
- 浏览器中的 JavaScript 是"安全的"，不提供对内存或 CPU 的底层访问
- 不同的 JavaScript 引擎实现可能导致细微的行为差异

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 基础知识]]
- [[JavaScript 代码质量]]

## 原始来源

- [简介](raw/zh.javascript.info/1-js/01-getting-started/index.md)
- [JavaScript 简介](raw/zh.javascript.info/1-js/01-getting-started/1-intro/article.md)
- [手册与规范](raw/zh.javascript.info/1-js/01-getting-started/2-manuals-specifications/article.md)
- [代码编辑器](raw/zh.javascript.info/1-js/01-getting-started/3-code-editors/article.md)
- [开发者控制台](raw/zh.javascript.info/1-js/01-getting-started/4-devtools/article.md)
