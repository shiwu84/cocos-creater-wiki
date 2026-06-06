---
title: TypeScript 模块系统
date: 2026-06-10
tags:
  - layer/ts
  - type/concept
  - status/stable
aliases: []
---

# TypeScript 模块系统

> [!abstract] 摘要
> TypeScript 的模块系统建立在 ES Modules 标准之上，但增加了**类型空间**的模块解析——编译器需要在文件系统上找到名字对应的类型定义，而不仅仅是找到要导入的 JavaScript 值。这引出了 TS 独特的三维问题：**模块语法**（用什么语法导入导出）、**模块解析**（模块名如何映射到文件系统中的文件）、**模块输出目标**（编译后生成什么格式的 JS 模块）。这三个维度中，模块解析是 TS 与纯 JS 的最大差异点——`node` 策略优先查找 `.d.ts` 文件，`classic` 策略是历史遗留。

## 根本问题：名字到文件的映射

JS 的模块系统解决的是"如何在不同文件之间共享代码"。TS 在此之上的额外问题是：当编译器看到 `import { Foo } from './bar'` 时，它不仅要找到 `bar.js` 的运行时绑定，还要找到 `Foo` 的**类型信息**——可能来自 `bar.ts`、`bar.d.ts`，或者 `bar/index.ts`。这个"名字到类型定义文件"的映射过程就是模块解析。

## 模块 vs 脚本

TS 沿袭了 JS 的区分：

- **模块**：文件包含顶级 `import` 或 `export` → 拥有自己的作用域，不污染全局
- **脚本**：文件没有 `import`/`export` → 所有声明在全局作用域

关键实践：如果想让一个文件被当作模块（哪怕当前不需要导入导出），添加 `export {};` 即可。

## 模块解析策略

### `node` 策略（现代默认）

模拟 Node.js 的模块解析算法，增加了 TS 特有的查找 `.ts`/`.tsx`/`.d.ts` 文件：

```
import { Foo } from './bar'
  → 查找 ./bar.ts → ./bar.tsx → ./bar.d.ts → ./bar/index.ts → ...
```

对 `node_modules` 中的包，先检查 `package.json` 的 `types`/`typings` 字段，再查找 `index.d.ts`。

### `classic` 策略（历史遗留）

简单的递归目录查找，不考虑 `node_modules`。几乎不应在现代项目中使用。

### 路径映射与别名

`tsconfig.json` 中的 `paths` 选项允许自定义模块名到路径的映射：

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@utils/*": ["src/utils/*"]
    }
  }
}
```

## 模块输出目标

`module` 选项决定编译后生成什么格式的 JS 模块：

| 目标 | 输出 | 适用场景 |
|------|------|---------|
| `commonjs` | `require`/`module.exports` | Node.js |
| `es2015`/`esnext` | 原生 `import`/`export` | 现代浏览器/打包器 |
| `system` | SystemJS 格式 | 已过时 |
| `umd` | 通用模块定义 | 同时支持浏览器和 Node |
| `nodenext` | Node.js 原生 ESM + CJS 互操作 | Node 12+ 最佳选择 |

`moduleResolution` 应与 `module` 配合设置。

## 与 CommonJS 的互操作

TypeScript 支持从 ESM 导入 CJS 模块，以及反方向，但有微妙的语义差异：

```typescript
// CJS 模块
module.exports = function foo() {};

// ESM 中的命名空间导入
import * as foo from './foo';  // foo 本身是函数，不是命名空间
import foo from './foo';       // 等价（当 esModuleInterop 开启时）
```

`esModuleInterop` 选项（推荐开启）使得 TS 更好地模拟 Babel 在 CJS/ESM 互操作时的行为。

## 命名空间（历史包袱）

在 ES6 模块标准出现前，TypeScript 使用 `namespace`/`module` 关键字组织代码：

```typescript
namespace MyApp {
    export class Foo {}
}
```

现在 `namespace` 几乎总是不推荐的——用 ES Modules 替代。唯一的例外是用于声明文件（`.d.ts`）中模拟嵌套全局对象（如 `globalThis.MyApp.Foo`）。

## 与 Rust 模块系统的对比

| 维度 | TypeScript | Rust |
|------|-----------|------|
| 模块文件 | 自动（文件名即模块） | 显式 `mod` 声明 |
| 私有性 | 默认公开 | 默认私有（`pub`） |
| 路径解析 | 文件路径 + node_modules 查找 | `crate::` 前缀 + 文件路径 |
| 类型空间 | JS 运行时 + TS 类型，两个空间 | 编译后无类型信息 |

Rust 的 `mod` 声明是显式的——模块树必须由程序员维护；TS 的文件即模块更自由。Rust 的默认私有性迫使你深思熟虑地决定 API 边界；TS 的默认公开（export）更容易但也更容易泄漏实现细节。

## 注意事项

- **`.d.ts` 文件不会产生运行时代码**：它们只是类型信息，导入它们无需担心包体积
- **`type` 导入**：`import type { Foo } from './bar'` 在编译后被完全移除，不会出现在 JS 输出中
- **`moduleResolution: "bundler"`**：使用 Vite/esbuild 等打包工具时设置此选项，让 TS 按打包器的规则解析

## 相关页面

- [[TypeScript 概述]] — TS 类型系统全局导航
- [[JavaScript 模块]] — ES6 模块语法基础（TS 模块语法的前提）
- [[声明文件]] — `.d.ts` 文件是模块解析的关键查找目标
- [[Cargo 与 crate 生态]] — Rust 模块系统对比
- [[结构化类型与类型兼容性]] — 模块之间类型兼容性的判断

## 原始来源

- [TypeScript Modules](raw/TypeScript-Website/packages/documentation/copy/en/handbook-v2/Modules.md)
- [Module Reference](raw/TypeScript-Website/packages/documentation/copy/en/modules-reference/Introduction.md)
