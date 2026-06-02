---
title: Rust 概述
date: 2026-06-02
tags:
  - layer/rust
  - type/overview
  - status/stable
aliases:
  - Rust 程序设计语言
---

# Rust 概述

> [!abstract] 摘要
> Rust 是一种系统级编程语言，核心创新是**在编译期实现内存安全而不依赖垃圾回收（GC）**。通过所有权（Ownership）、借用（Borrowing）和生命周期（Lifetime）三大规则，Rust 编译器在程序运行之前就消除了悬垂指针、数据竞争、双重释放等内存错误——而这一切零运行时开销。Rust 不选择"安全或性能"，而是通过类型系统将内存管理与并发安全统一成一个编译期问题，让"安全"成为默认，"不安全"成为显式选择。它的 trait 系统提供了零成本多态，async/await 支持惰性执行的高性能异步编程，宏系统允许编译期元编程。作为本仓库**系统编程链**（Linux → 系统编程概念 → Rust）的终端节点，Rust 将 OS 层的概念（虚拟内存、线程调度、系统调用）封装为编译期安全抽象；同时与 JS/TS 的异步模型、类型系统、闭包等概念形成深层共鸣——它既是 C/C++ 的现代替代，也是 TypeScript 类型系统的"编译期哲学"在底层语言中的极致体现。

## Rust 解决的根本问题

编程语言在半个世纪里一直在"安全"与"控制"之间做取舍：

| 语言类别 | 内存安全 | 性能 | 代价 |
|---------|---------|------|------|
| C / C++ | 无保证（手动管理） | 极高性能 | 悬垂指针、UAF、双重释放 |
| Java / Go / JS | 运行时 GC | 较好 | GC 暂停、运行时开销 |
| Python / Ruby | 运行时 GC + 引用计数 | 较低 | 更大的运行时开销 |

**Rust 引入了第三种方案**：不靠 GC，不靠手动 `malloc`/`free`，而是让编译器在编译期验证所有内存操作的正确性。这套方案的核心是所有权系统。

## 核心创新：所有权、借用与生命周期

### 所有权（Ownership）

Rust 中每个值有且仅有一个**所有者**（变量）。当所有者离开作用域，值被自动释放：

- **移动语义（Move）**：赋值或传参会转移所有权，原变量失效
- **Copy 类型**：简单的栈上数据（整数、布尔等）自动按位复制，不转移所有权
- **作用域释放（Drop）**：所有者离开作用域时，编译器自动插入释放代码

这与 C++ 的 RAII 理念相似，但 Rust 的编译器**强制执行**这些规则——违反则编译不过。

### 借用（Borrowing）

为了在不转移所有权的前提下访问数据，Rust 提供**引用**：

- `&T`：不可变引用（共享引用），允许多个同时存在
- `&mut T`：可变引用（独占引用），同一时刻只能有一个；存在可变引用时不可变引用无效
- 引用必须始终有效——编译器保证引用不会比被引用数据活得更久

### 生命周期（Lifetime）

生命周期标注 `'a` 让编译器追踪引用之间的有效期关系。大多数情况下编译器自动推断（生命周期省略规则），但在涉及多个引用且关系不明确时需要显式标注。

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

> 生命周期的本质不是"让变量活得更久"，而是**告诉编译器两个引用之间的存活关系**，防止返回悬垂引用。

## 概念层次

### 类型系统

**枚举（Enum）**：Rust 的枚举是**代数数据类型（ADT）**，每个枚举值可以携带不同的数据：

```rust
enum Result<T, E> { Ok(T), Err(E) }
enum Option<T> { Some(T), None }
```

模式匹配（`match`）强制覆盖所有可能情况，这是 Rust 编译器消除空指针错误和未处理错误的关键机制。

**Trait**：Rust 的接口抽象机制。Trait 定义行为签名，类型实现 Trait 来提供具体行为。类似于 TypeScript 的接口，但更强大：
- 可以为外部类型实现外部 Trait（孤儿规则限制之一方必须在本 crate）
- Trait bound 约束泛型参数："T 必须实现了某个 Trait"
- 关联类型、默认实现、supertrait 等高级特性

**泛型**：编译期单态化——编译器为每种具体类型生成独立代码，零运行时开销。

### 内存管理

| 智能指针 | 用途 |
|---------|------|
| `Box<T>` | 堆分配，独占所有权 |
| `Rc<T>` | 引用计数，单线程共享所有权（不可变） |
| `Arc<T>` | 原子引用计数，线程安全的共享所有权 |
| `RefCell<T>` | **内部可变性**——在持有不可变引用时修改内部值，运行时检查借用规则 |

`Rc<RefCell<T>>` 组合可以实现多个所有者对同一数据的可变访问——这让 Rust 支持传统面向对象中常见的共享可变状态模式，但所有潜在问题都被显式标注出来。

### 无畏并发

Rust 的所有权系统天然防止数据竞争：**Send**（可以安全地将所有权转移到另一线程）和 **Sync**（可以安全地在多个线程间共享引用）两个 trait 是编译期标记，编译器检查多线程代码是否符合这些约束。

并发模型：
- **消息传递**：`std::sync::mpsc::channel` 实现"通过发消息共享内存，而非通过共享内存通信"
- **共享状态**：`Arc<Mutex<T>>` 提供线程安全的互斥访问
- **Async**：`async`/`await` 语法 + `Future` trait 支持协作式并发

### async/await：惰性 Future

Rust 的异步模型与 JavaScript 的根本区别：

| 特性 | Rust | JavaScript |
|------|------|------------|
| Future/Promise 执行 | **惰性**——不调用 `.await` 永远不会执行 | **急切**——Promise 创建即开始执行 |
| 运行时 | 需外部提供执行者（tokio、async-std 等） | 内置事件循环 |
| 取消 | Future 被 drop 即取消 | AbortController |
| CPU 密集型 vs IO | 明确区分，async 适合 IO 密集型 | 统一事件循环模型 |

Rust 的惰性 Future 设计意味着零开销：不 `.await` 的 Future 不会有任何计算或内存成本。异步运行时（如 tokio）负责向 OS 注册 IO 事件并在数据就绪时唤醒 Future。

### 高级特性

- **Unsafe Rust**：`unsafe` 块允许解引用裸指针、调用 unsafe 函数、访问可变静态变量、实现 unsafe trait。这是 Rust 的"逃生舱"——当编译器无法验证安全性的场景（FFI、直接内存操作、高性能数据结构），开发者承担安全责任。
- **宏**：声明宏（`macro_rules!`）和过程宏（derive、属性宏、函数宏），在编译期进行代码生成和变换。
- **FFI**：通过 `extern "C"` 与 C 库互操作，是 Rust 调用操作系统 API 的主要方式——因为 Linux 系统调用接口本质上是 C ABI。

### 工具链

- **Cargo**：构建系统 + 包管理器 + 测试运行器 + 文档生成器，一站式工具
- **Crates.io**：Rust 社区包注册中心
- **rustc**：编译器，错误信息以友好和精确著称
- **rustfmt / clippy**：代码格式化和 lint 工具
- **内置测试框架**：`#[test]` 标注、`cargo test` 运行，单元测试、集成测试、文档测试三类

## 跨领域连接

### Rust ← Linux：底层到编译期的映射

Rust 的许多设计概念根植于 Linux/OS 基础：

- **所有权 ≈ 进程内存隔离**：Linux 中每个进程有独立虚拟地址空间，一个进程不能随意访问另一个进程的内存。Rust 的所有权系统将这一隔离从进程级别下沉到**变量级别**——一个变量独有数据，不可隐式共享。
- **借用规则 ≈ 读者-写者锁**：`&T` 可共享不可变 ≈ 共享锁，`&mut T` 独占可变 ≈ 排他锁。但在 Rust 中这一切是**编译期验证的**，不涉及运行时锁开销。
- **Drop trait ≈ 内核资源清理**：文件描述符在进程退出时被内核回收，Rust 中值离开作用域时自动调用 `Drop`——同样的 RAII 思想。
- **系统调用 ≈ unsafe 边界**：Rust 的安全抽象最终通过 `unsafe` 块调用操作系统 C ABI。Linux 内核是"安全世界"与"危险世界"的边界线。
- **Send/Sync ≈ 线程安全语义**：Linux 线程共享同一虚拟地址空间因此存在竞争风险；Rust 用 Send/Sync 在编译期消除这些风险。

相关页面：[[Linux 概述]]、[[Linux 进程模型]]、[[Linux 文件系统]]、[[Linux 网络协议栈]]

### Rust ↔ JavaScript：两种异步哲学

两者都有 `async`/`await` 语法，背后是不同的并发哲学：

- **JS Promise 是热启动（eager）**：创建 Promise 时 executor 立即执行，事件循环调度微任务。Rust 的 `Future` 是**惰性的（lazy）**：不 `.await` 就什么也不发生。
- **JS 单线程事件循环 vs Rust 多线程 + async 混合模型**：JS 用 one-thread-per-reactor 模式处理并发；Rust 可以用多线程 + tokio 在多个核心上同时运行多个 Future。
- **闭包**：JS 闭包隐式捕获变量（按引用），Rust 闭包要求显式标注捕获方式——`move` 闭包将变量所有权移入闭包，`||` 闭包按引用捕获。这是所有权哲学在函数式特性中的延伸。

相关页面：[[JavaScript Promise 与异步]]、[[JavaScript 函数进阶]]

### Rust ↔ TypeScript：两种类型哲学的比较

两者都是"类型增强的现代语言"，但方向截然相反：

| 维度 | TypeScript | Rust |
|------|-----------|------|
| 本质 | JS + 类型标注层（编译期擦除） | 独立语言，类型影响代码生成 |
| Trait / 接口 | 结构化类型（鸭子类型）、类型擦除 | 显式实现、单态化、零开销 |
| 泛型 | 类型擦除（运行时无类型信息） | 单态化（为每种类型生成代码） |
| 联合类型 | 任意类型的并集 | Enum 变体（标称类型，携带数据） |
| 空值处理 | `undefined` / `null` + strictNullChecks | `Option<T>` 枚举，编译器强制检查 |

TypeScript 的"编译期安全检查后擦除所有类型"的哲学与 Rust 一致——但 Rust 的类型系统深入到了内存和并发的安全保障。

### Rust ↔ 软件工程

Rust 将软件工程中的最佳实践**内置到了语言和工具链中**：

- **错误处理即类型**：`Result<T, E>` 而不是异常抛出——错误变成了必须要处理的返回值。`?` 运算符优雅传播错误。这比 try-catch 更显式，避免吞没错误。
- **测试文化**：内置 `#[test]`、`cargo test`、文档测试（`/// ``` ` 代码块作为测试运行）。测试不是附加物，是语言标准工作流。
- **Cargo 即构建系统**：每个 Rust 项目有统一的构建方式——不像 C/C++ 生态中 Makefile/CMake/Meson 各自为政。
- **不可变优先**：`let` 声明的变量默认不可变（`let mut` 才可变），与函数式编程和软件工程中的"最小权限原则"契合。

相关页面：[[Jujutsu VCS]]（用 Rust 构建的下一代版本控制系统）、[[Git 与版本控制]]

## 学习路径建议

对于本仓库的体系，建议按以下顺序学习 Rust：

1. 先确保理解 [[Linux 进程模型]]——理解为什么内存隔离是安全的基础
2. 进入 Rust 核心概念：所有权 → 借用 → 生命周期
3. 遍历枚举和模式匹配——理解 Rust 如何消除空值和未处理分支
4. 学习 Trait 和泛型——理解 Rust 的多态和代码复用
5. 掌握智能指针——理解堆分配、引用计数、内部可变性
6. 进入并发：线程与消息传递 → Send/Sync → async/await
7. 最后接触 unsafe Rust 和宏——它们是 Rust 的"高级界面"，与操作系统对话的边界

## 相关页面

### 本领域（layer/rust）
- [[所有权与借用]] — Rust 最核心的概念
- [[Rust 枚举与模式匹配]] — 代数数据类型与穷尽性检查
- [[Rust Trait 与泛型]] — 零成本抽象与接口设计
- [[Rust 生命周期]] — 引用有效性确保
- [[Rust 智能指针]] — Box, Rc, RefCell, Arc
- [[Rust 错误处理]] — Result, panic!, ? 运算符
- [[Rust 并发模型]] — Send/Sync, 线程, 消息传递, async
- [[Rust 模块与 Crate]] — 包管理与代码组织

### JS/TS 领域（跨链共鸣）
- [[JavaScript Promise 与异步]] — eager Promise vs lazy Future
- [[JavaScript 函数进阶]] — 闭包捕获机制对比
- [[JavaScript 模块]] — ES6 模块 vs Cargo crate 模块系统
- [[JavaScript 原型与继承]] — 原型链继承 vs trait 组合
- [[JavaScript Generator 与迭代器]] — Iterator 模式对比

### 工具与基础设施
- [[Jujutsu VCS]] — Rust 构建的下一代版本控制工具
- [[Git 与版本控制]] — 版本控制基础

### 前提层
- [[Linux 概述]] — OS 基础是 Rust 底层抽象的来源
- [[Linux 进程模型]] — 虚拟内存、进程隔离 = 所有权的 OS 原型
- [[Linux 文件系统]] — 文件描述符与 Drop trait 的 RAII 对应
- [[Linux 网络协议栈]] — 异步 IO 的 OS 底层支持

## 原始来源

- [Rust 程序设计语言 - 简介](raw/trpl-zh-cn/src/ch00-00-introduction.md)
- [认识所有权](raw/trpl-zh-cn/src/ch04-00-understanding-ownership.md)
- [使用结构体组织相关联的数据](raw/trpl-zh-cn/src/ch05-00-structs.md)
- [枚举和模式匹配](raw/trpl-zh-cn/src/ch06-00-enums.md)
- [包、Crates 与模块](raw/trpl-zh-cn/src/ch07-00-managing-growing-projects-with-packages-crates-and-modules.md)
- [错误处理](raw/trpl-zh-cn/src/ch09-00-error-handling.md)
- [泛型、Trait 和生命周期](raw/trpl-zh-cn/src/ch10-00-generics.md)
- [编写自动化测试](raw/trpl-zh-cn/src/ch11-00-testing.md)
- [函数式语言特性：迭代器与闭包](raw/trpl-zh-cn/src/ch13-00-functional-features.md)
- [智能指针](raw/trpl-zh-cn/src/ch15-00-smart-pointers.md)
- [无畏并发](raw/trpl-zh-cn/src/ch16-00-concurrency.md)
- [异步编程基础：Async、Await、Future 与 Stream](raw/trpl-zh-cn/src/ch17-00-async-await.md)
- [高级特性](raw/trpl-zh-cn/src/ch20-00-advanced-features.md)
- [模式与模式匹配](raw/trpl-zh-cn/src/ch19-00-patterns.md)
