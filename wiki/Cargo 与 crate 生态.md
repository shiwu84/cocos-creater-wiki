---
title: Cargo 与 crate 生态
date: 2026-06-10
tags:
  - layer/rust
  - type/concept
  - status/stable
aliases: []
---

# Cargo 与 crate 生态

> [!abstract] 摘要
> Cargo 是 Rust 的构建系统和包管理器，解决的根本问题是"代码如何从源文件到达可执行产物"。它在一条命令中整合了编译、依赖解析、测试、文档生成、发布——这种"一切都装在一个工具里"的设计使得 Rust 项目从创建到分发不需要拼凑多个第三方工具。crate 是 Rust 的编译单元和代码分发单位，通过 crates.io 注册中心形成了统一的开源生态。这一生态相比 npm（JS）和 pip（Python）的差异在于：Cargo 是语言标准工具链的一部分，不是生态拼装。

## 根本问题：组织与分发

随着项目增长，一个 .rs 文件很快变得不可维护。Rust 的模块系统回答了"如何组织代码"，Cargo 回答了"如何构建、测试、依赖管理和分发这些组织好的代码"。

### 模块系统的层级结构

```
包 (package)            ← Cargo.toml 定义，可含多个 crate
  ├── 库 crate            ← src/lib.rs（可选）
  └── 二进制 crate         ← src/main.rs，也可多个（src/bin/）
       └── 模块 (module)   ← mod 关键字声明，可跨文件
```

- **包**：Cargo.toml 描述的项目，包含一个或多个 crate
- **crate**：编译的最小单位，可以是库或二进制
- **模块**：用 `mod` 声明的命名空间，控制作用域和私有性
- **路径**：`crate::module::function` 这样的命名解析规则
- **use**：将路径引入当前作用域

### 私有性边界

默认所有项（函数、结构体、字段）是**私有**的。`pub` 关键字将项暴露给父模块。这种"默认私有"的设计与大多数语言（默认公开）相反——鼓励深思熟虑地暴露 API。

## Cargo 的核心能力

### 发布配置

`Cargo.toml` 中通过 `[profile]` 节控制编译行为：
- `dev`：开发模式，快速编译，少量优化
- `release`：发布模式，完全优化（比 dev 模式慢 10 倍编译，但运行更快）

### 依赖管理

Cargo.toml 中声明依赖，Cargo 从 crates.io 自动拉取并解析版本冲突。使用语义化版本：`^1.2.3` 表示兼容 1.2.3 到 <2.0.0。

### 工作空间

对于包含多个包的大型项目（如 Web 前端 + 后端 + 共享库），Cargo 工作空间允许多个包共享一个 `target/` 目录和一个 `Cargo.lock` 文件，避免重复编译。

### 测试集成

`cargo test` 执行单元测试（与源码混写）和集成测试（位于 `tests/` 目录，只能调用公共 API）。测试本身是一等公民：`#[test]` 属性标注，没有额外的测试框架依赖。

## crates.io 生态

crates.io 是 Rust 的包注册中心，所有 crate 通过统一的注册中心分发（类似 npm registry，不像 C++ 的头文件生态那样分散）。

对比表：

| 特性 | Cargo | npm | pip |
|------|-------|-----|-----|
| 语言内置 | 是（官方工具链） | 非语言标准 | 非语言标准 |
| 锁文件 | `Cargo.lock` | `package-lock.json` | `requirements.txt` |
| 私有性边界 | 模块级 pub | 文件 export | 命名约定 _ |
| 文档生成 | `cargo doc` | JSDoc（外挂） | Sphinx（外挂） |
| 测试 | `cargo test` | jest/mocha | pytest |

## 注意事项

- **孤儿规则限制**：Rust 不允许在外部类型上实现外部 trait（见 [[Trait 系统]]），这与 npm 包能自由 monkey-patch 其他模块形成鲜明对比
- **crate 间版本冲突**：如果一个依赖树中同一 crate 的两个不兼容版本同时出现，Cargo 会报错——这是 Rust 类型安全的必然要求（同一名字代表两个不同的类型）
- **`cargo install` 安装的是二进制**：不要与依赖管理混淆；要安装开发依赖用 `cargo add`

## 相关页面

- [[Rust 概述]] — Rust 语言全局导航
- [[所有权与借用]] — 模块系统中的所有权传递
- [[Trait 系统]] — crate 生态的核心抽象机制
- [[Rust 并发模型]] — 跨 crate 的并发模式
- [[Linux 包管理]] — 操作系统级包管理与语言级包管理的概念共鸣
- [[软件工程概述]] — 模块化设计作为 SWE 通用概念

## 原始来源

- [包、Crates 与模块](raw/trpl-zh-cn/src/ch07-00-managing-growing-projects-with-packages-crates-and-modules.md)
- [进一步认识 Cargo 和 Crates.io](raw/trpl-zh-cn/src/ch14-00-more-about-cargo.md)
- [Cargo 工作空间](raw/trpl-zh-cn/src/ch14-03-cargo-workspaces.md)
