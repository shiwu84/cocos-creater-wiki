---
title: Rust 错误处理
date: 2026-06-10
tags:
  - layer/rust
  - type/concept
  - status/stable
aliases: []
---

# Rust 错误处理

> [!abstract] 摘要
> Rust 不采用异常机制，而是将错误分为两大类：用 `Result<T, E>` 处理可恢复错误，用 `panic!` 处理不可恢复错误。这种显式二分法迫使调用方在调用点面对失败可能，避免了隐式异常传播带来的"假装成功"问题。`?` 运算符结合 `From` trait 实现跨错误类型自动转换，让错误传播既简洁又可组合。这一策略深刻体现了"让错误处理成为类型系统的一部分"的设计哲学——错误不再是意外，而是类型。

## 根本问题：错误应该类型化

传统语言（Java、Python、C++）用异常处理错误有一个核心问题：**调用方不知道一个函数可能抛出什么**。签名为 `fn read()` 的函数可能抛出 `FileNotFoundException`、`IOException`、`SecurityException`……而这些从不体现在类型签名中。结果是：
- 调用者可能忘记处理某些异常
- 异常传播路径隐式、不可追踪
- 代码"看起来正确"直到运行时崩溃

Rust 的解决方案是将"可能失败"编码到返回值类型中：签名 `fn read() -> Result<String, io::Error>` 显式告诉调用方"这个操作可能以 `io::Error` 失败"。编译器会强制你处理 `Err` 分支——不是在文档里写，而是在代码里处理。

## `panic!`：不可恢复错误

当程序遇到**不可能继续**的情况（数组越界、逻辑矛盾、不可恢复的状态损坏），使用 `panic!` 终止执行。

`panic!` 的行为：
- **展开（unwind）**：默认行为，沿调用栈向上清理（调用 `Drop`），最终终止线程
- **终止（abort）**：在 `Cargo.toml` 中设置 `panic = "abort"` 后，直接终止进程，不做清理
- `RUST_BACKTRACE=1` 环境变量可打印回溯信息

何时用 `panic!`：
- 示例代码和测试中（用 `unwrap` 或 `expect` 来简化）
- 当遇到"不可能"的状态时（逻辑 bug，应由开发者修复）
- 调用者无法合理处理错误时（如关键初始化失败）

## `Result<T, E>`：可恢复错误

`Result` 是一个枚举：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

核心处理方式：

**`match` 分支**：最显式的处理方式，适合需要针对不同错误类型做出不同响应的场景。

**`unwrap` / `expect`**：`unwrap` 在 `Err` 时 `panic!`；`expect(msg)` 提供自定义错误信息。适合"我确信不会失败"的场景。

**`?` 运算符**：如果 `Ok` 则取出值，如果 `Err` 则提前返回错误。这是 Rust 错误传播的核心机制。`?` 还会自动调用 `From::from()` 转换错误类型——这意味着函数可以返回一个统一的错误类型，而内部的各种错误会自动转换。

**组合方法**：`map_err`、`and_then`、`or_else`、`unwrap_or`、`unwrap_or_else` 等提供了函数式的错误处理链。

### `?` 运算符的使用限制

`?` 只能用于返回值兼容的函数：
- 对 `Result` 使用 `?` → 函数必须返回 `Result`
- 对 `Option` 使用 `?` → 函数必须返回 `Option`
- 不能混用（但可显式转换：`Result::ok()` / `Option::ok_or()`）
- `main` 函数可返回 `Result<(), E>` 以支持 `?`

### `?` 与 `try!` 的关系

`?` 是 `try!` 宏的后继，语义相同但语法更简洁。`try!` 在 Rust 2018 edition 后不再推荐。

## 跨语言对比

| 语言 | 可恢复错误 | 不可恢复错误 | 错误类型强制 |
|------|-----------|-------------|-------------|
| Rust | `Result<T, E>` + `?` | `panic!` | 编译期（`Result` 在类型签名中） |
| Go | `(val, err)` 多返回值 | `panic` + `recover` | 弱（`err` 可被忽略但不建议） |
| JS | `try-catch` + Promise rejection | `throw` | 无（异常可被忽略） |
| Java | checked exceptions（部分） | unchecked exceptions | 检查型异常强制处理 |

Rust 的 `Result` 是 Go 风格多返回值的**类型安全版**：编译器保证错误不会被忽略（通过 `#[must_use]` 属性），`?` 运算符消除了 Go 中 `if err != nil { return err }` 的样板代码。

## 注意事项

- **不要过早在函数中使用 `panic!` 替代返回 `Err`**：某个函数的"意外"往往是上层调用者的"可处理情况"（如文件不存在是正常的用户输入）
- **`?` 自动类型转换是一把双刃剑**：它简化了代码，但可能掩盖错误类型的不匹配。使用自定义错误类型配合 `thiserror` crate 或手动实现 `From` 可以兼顾简洁和清晰
- **`main` 函数返回 `Result<(), Box<dyn Error>>`** 是快速原型的好技巧，但生产代码应使用具体的错误类型而非 `Box<dyn Error>`

## 相关页面

- [[所有权与借用]] — 错误处理中引用的生命周期约束，如错误复用
- [[Trait 系统]] — `From` trait 是 `?` 运算符自动类型转换的基础
- [[Rust 并发模型]] — `Send`/`Sync` 影响错误类型能否跨线程传递
- [[错误处理策略对比]] — 跨语言错误处理全景对比（Linux/JS/Rust/Cocos）
- [[软件工程概述]] — 错误处理作为软件工程通用概念

## 原始来源

- [错误处理](raw/trpl-zh-cn/src/ch09-00-error-handling.md)
- [Result 处理可恢复错误](raw/trpl-zh-cn/src/ch09-02-recoverable-errors-with-result.md)
- [panic! 还是 Result](raw/trpl-zh-cn/src/ch09-03-to-panic-or-not-to-panic.md)
