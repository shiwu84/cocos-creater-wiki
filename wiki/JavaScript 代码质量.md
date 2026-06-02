---
title: JavaScript 代码质量
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases:
  - JS 代码质量
  - JS 调试与测试
---

# JavaScript 代码质量

> [!abstract] 摘要
> 代码质量是编程的核心素养之一。本章涵盖 Chrome 开发者工具的调试功能、JavaScript 编码风格指南、注释的规范写法、自动化测试（Mocha/Chai）以及 Polyfill 和转译器的概念，帮助编写可维护、可测试的高质量代码。

## 核心概念

### 浏览器调试
- **Sources 面板**：查看文件、设置断点、单步执行
- **Console 面板**：查看错误、执行命令、检查变量
- 断点调试：条件断点、在 DOM 变更/网络请求时断点、异常断点
- 追踪执行：Step Over（跳过函数）、Step Into（进入函数）、Step Out（跳出函数）

### 编码风格
- **花括号**：Egyptian 风格（K&R），左花括号与关键词同行，前加空格
- **缩进**：水平缩进 2 或 4 个空格；垂直空行分割逻辑块
- **分号**：语句末尾需加分号，避免自动分号插入（ASI）导致的问题
- **行长度**：通常在 80-120 字符间，长表达式应拆分
- **嵌套层级**：避免过深嵌套（建议不超过 3 层），用提前返回减少嵌套
- **一行一个变量**：提高可读性
- 推荐使用 ESLint 等 Linter 工具自动化检查代码风格

### 注释
- 好的注释解释 **为什么** 这么做，而非 **做了什么**
- 函数注释：描述功能、参数、返回值（可使用 JSDoc 格式）
- 避免注释"显而易见"的代码；用描述性的变量/函数名替代注释
- 复杂算法或特殊方案应添加说明性注释

### 自动化测试（BDD）
行为驱动开发（BDD）三要素：测试、文档和示例。

核心工具：
- **Mocha**：测试框架，提供 `describe` 和 `it` 组织测试
- **Chai**：断言库，提供 `assert`、`expect`、`should` 风格
- **Sinon**：函数监视、模拟与桩工具

开发流程（TDD 迭代）：
1. 编写初始规范（spec）
2. 实现最小可行代码
3. 运行测试，修正至通过
4. 添加更多用例（包括边界情况）
5. 循环迭代，直到功能完善

### Polyfill 与转译器
- **Polyfill**：用 JS 实现新特性，使老浏览器支持现代 API（如 `Array.prototype.includes`）
- **转译器（Transpiler）**：将新语法编译为旧版兼容代码（如 Babel 将 ES6 转为 ES5）
- 使用前确认特性是否需要补丁（参考 caniuse.com）

## 关键细节

### 忍者代码的反面教材
一些常见的"反模式"应避免：
- 使用 `_`、`__` 或单字母变量名
- 缩写不直观（如 `str` → 到底是 string 还是 strong？）
- 函数名与功能不符
- 修改外部变量产生副作用
- 函数过长，一个函数做多件事

### 测试的最佳实践
- 一个 `it` 块测试一个用例
- 覆盖正常路径和边界情况
- 测试应可独立运行、可重复执行
- 先用测试定义期望行为，再实现代码
- `assert` 失败会抛出错误，Mocha 捕获并展示结果

## 与其他系统的关系

- [[JavaScript 基础知识]] — 前提：测试对象是需要已掌握语言基础
- [[JavaScript 函数进阶]] — 装饰器模式常用于测试中的 mock/sinon
- 代码风格与 TypeScript 的严格类型检查形成互补
- Linter/Formatter 工具链（ESLint、Prettier）与 CI/CD 集成

## 注意事项

- 编码风格约定是团队习惯，并非绝对的教条；关键是团队内部一致
- `debugger` 命令可在代码中设置断点，但上线前应移除
- 测试不能替代代码审查，二者互补
- Polyfill 会增加页面体积，按需引入
- 编写 `return` 时注意不要换行，否则 JS 会在 `return` 后自动添加分号

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 基础知识]]
- [[JavaScript 函数进阶]]

## 原始来源

- [代码质量](raw/zh.javascript.info/1-js/03-code-quality/index.md)
- [在 Chrome 中调试](raw/zh.javascript.info/1-js/03-code-quality/01-debugging-chrome/article.md)
- [代码风格](raw/zh.javascript.info/1-js/03-code-quality/02-coding-style/article.md)
- [注释](raw/zh.javascript.info/1-js/03-code-quality/03-comments/article.md)
- [忍者代码](raw/zh.javascript.info/1-js/03-code-quality/04-ninja-code/article.md)
- [使用 Mocha 进行自动化测试](raw/zh.javascript.info/1-js/03-code-quality/05-testing-mocha/article.md)
- [Polyfill 和转译器](raw/zh.javascript.info/1-js/03-code-quality/06-polyfills/article.md)
