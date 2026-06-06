---
title: tsconfig 配置
date: 2026-06-10
tags:
  - layer/ts
  - type/concept
  - status/stable
aliases:
  - tsconfig.json
---

# tsconfig 配置

> [!abstract] 摘要
> `tsconfig.json` 是 TypeScript 项目的"指挥中心"——它告诉编译器哪些文件应被编译、以什么规则编译、输出到哪里。如果说 `tsc` 是编译器，`tsconfig.json` 就是"编译策略"。解决的根本问题是：TypeScript 有 100+ 个编译选项，没有人能在命令行记住它们，也没有人应该为每个项目手工配置所有选项。核心心智模型：**`strict` 家族**（类型安全）、**`module` 家族**（模块系统）、**`target` 家族**（输出兼容性）是三个独立的决策轴，分别回答"多安全"、"怎么组织"、"跑在哪里"。

## 根本问题：项目编译策略的集中表达

一个 TS 项目编译时至少需要回答：
- 哪些文件要编译？（`include`/`exclude`/`files`）
- 生成什么版本的 JS？（`target`）
- 用什么模块系统？（`module`）
- 类型检查多严格？（`strict` 家族）
- 输出到哪里？（`outDir`）

把这些写进 `tsconfig.json`，团队成员和 CI 就能用 `tsc` 或 `tsc -p tsconfig.json` 获取一致的编译行为。

## 三层结构模型

可以将 tsconfig 选项分为三个决策层：

### 第一层：文件范围

- `include`：glob 模式，指定哪些文件被编译
- `exclude`：排除的文件（默认排除 `node_modules`）
- `files`：精确的文件列表（当 include 不够精确时）
- `rootDir`：源码根目录（影响编译输出的目录结构）

### 第二层：严格性控制（`strict` 家族）

`"strict": true` 会同时开启以下 8 个子选项：

| 子选项 | 作用 |
|--------|------|
| `noImplicitAny` | 禁止隐式 `any`——函数参数必须标注类型 |
| `strictNullChecks` | `null`/`undefined` 不能赋值给其他类型 |
| `strictFunctionTypes` | 函数类型参数逆变检查 |
| `strictBindCallApply` | `bind`/`call`/`apply` 的类型检查 |
| `strictPropertyInitialization` | 构造函数中必须初始化所有类属性 |
| `noImplicitThis` | 禁止隐式 `any` 类型的 `this` |
| `alwaysStrict` | 以 `use strict` 模式解析，并在输出中添加 |
| `useUnknownInCatchVariables` | catch 变量默认为 `unknown` 而非 `any` |

> **推荐**：新项目始终开启 `"strict": true`。如果老项目仍需迁移，至少先开启 `noImplicitAny` 和 `strictNullChecks`——这两个是 TS 类型安全的核心增益。

### 第三层：输出控制

- `target`：编译目标 JS 版本（`ES2015` → `ES2024`）
- `module`：输出模块格式（`commonjs`、`esnext`、`nodenext`）
- `moduleResolution`：模块解析策略（`node`、`bundler`）
- `outDir`：编译输出目录
- `declaration`：生成 `.d.ts` 声明文件
- `sourceMap`：生成 source map

### 其他重要选项

- `baseUrl` + `paths`：自定义模块路径映射（配合路径别名）
- `types`：限制自动引入的 `@types/` 包
- `jsx`：JSX 处理方式（`react`、`preserve`、`react-jsx`）
- `lib`：指定内置 API 声明文件集（`["ES2021", "DOM"]`）

## 项目引用

对于 monorepo（一个仓库多个 TS 项目），`references` 选项建立项目间的依赖关系：

```json
{
  "references": [
    { "path": "../shared" }
  ]
}
```

配合 `composite: true`，`tsc --build` 能智能增量编译整个依赖图——只重新编译变更的项目。

## tsconfig 继承

```json
{
  "extends": "@tsconfig/node20/tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist"
  }
}
```

`extends` 可以从 npm 包（如 `@tsconfig/node20`）或本地文件继承基础配置。TSConfig Bases 项目提供了为不同运行时优化的预配置。

## 注意事项

- **`tsconfig.json` 的存在标记项目根目录**：`tsc` 会在当前目录及父目录查找它
- **`exclude` 默认排除 `node_modules`**：无需手动添加；但通过 `include` 包含的路径会覆盖 `exclude` 的行为
- **声明文件生成**：如果 `declaration: true` 且 `outDir` 设置，`tsc` 会生成 `.d.ts` 文件——这是库发布的标准实践
- **不要混用 `files` 和 `include`**：`files` 指定的文件必须存在于项目中（否则报错），`include` 用 glob 模式更宽松

## 相关页面

- [[TypeScript 概述]] — TS 类型系统全局导航
- [[TypeScript 模块系统]] — `module`/`moduleResolution` 选项的详细背景
- [[声明文件]] — `declaration` 选项生成的 `.d.ts` 文件
- [[软件工程概述]] — CI/CD 中的构建配置管理

## 原始来源

- [What is a tsconfig.json](raw/TypeScript-Website/packages/documentation/copy/en/project-config/tsconfig.json.md)
- [Compiler Options](raw/TypeScript-Website/packages/documentation/copy/en/project-config/Compiler Options.md)
- [Project References](raw/TypeScript-Website/packages/documentation/copy/en/project-config/Project References.md)
