---
title: unsafe Rust
date: 2026-06-10
tags:
  - layer/rust
  - type/concept
  - status/stable
aliases: []
---

# unsafe Rust

> [!abstract] 摘要
> unsafe Rust 是 Rust 中刻意保留的第二语言——它不关闭借用检查器，而是额外提供五种"超能力"：解引用裸指针、调用 unsafe 函数、访问可变静态变量、实现 unsafe trait、访问 union 字段。其存在的根本原因是：编译器的静态分析是保守的——有些操作从程序员角度看是安全的，但编译器无法证明；另外，底层系统编程（操作系统、FFI）本身就不在安全 Rust 的保证范围内。核心哲学是用 `unsafe` 块将不安全操作隔离，对外提供安全的 API 封装。

## 根本问题：保守的编译器与底层现实

Rust 编译器对内存安全的保证基于一套可证明的规则（所有权、借用、生命周期）。但有两个场景这套规则不够：

1. **编译器无法证明的安全操作**：如 `split_at_mut`——安全逻辑上对同一个 slice 的两个不重叠子切片分别创建可变引用是合法的，但编译器看到的是"同一数据的两份可变借用"，拒绝编译
2. **底层硬件和操作系统接口**：裸指针操作、内存映射 I/O、直接汇编、调用 C 库——这些操作本身就不在 Rust 安全模型能建模的范围内

unsafe Rust 是 Rust 对这两个场景的应答：不是假装能建模一切，而是提供一个明确的"我理解并承担后果"的边界。

## 五种超能力

### 1. 解引用裸指针

裸指针 `*const T` / `*mut T` 与引用 `&T` 的区别：

| 特性 | 引用 `&T` | 裸指针 `*const T` |
|------|----------|-------------------|
| 保证有效 | 是（编译期检查） | 否 |
| 可空 | 否 | 是 |
| 独占性 | 可变/不可变互斥 | 可同时存在可变和不可变 |
| 自动清理 | 生命周期结束自动 drop | 无 |

在安全代码中**可以创建**裸指针，但只能在 `unsafe` 块中**解引用**。

### 2. 调用 unsafe 函数

标记为 `unsafe` 的函数意味着"调用方必须满足某些 Rust 无法验证的前置条件"。

```rust
unsafe fn dangerous() {}

fn main() {
    unsafe {
        dangerous(); // 调用方承诺已验证前置条件
    }
}
```

标准实践是将 unsafe 函数封装为安全抽象：

```rust
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    // 安全的公共 API
    unsafe {
        // 内部用 unsafe 实现
    }
}
```

### 3. FFI：外部函数接口

`extern` 关键字允许 Rust 调用其他语言（主要是 C）的函数：

```rust
unsafe extern "C" {
    fn abs(input: i32) -> i32;
}
```

也可以在 Rust 中定义函数让其他语言调用：

```rust
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Called from C!");
}
```

### 4. 访问可变静态变量

Rust 中的全局可变状态（`static mut`）是不安全的，因为多个线程可能竞态访问：

```rust
static mut COUNTER: u32 = 0;
fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc; // 必须 unsafe
    }
}
```

### 5. 实现 unsafe trait / 访问 union

某些 trait（如 `Send`、`Sync`）被标记为 unsafe，因为实现它们需要承诺内存安全保证。`union` 字段访问是不安全的因为 Rust 无法知道当前存储的是哪个变体。

## 安全封装原则

```rust
// 不好的实践：到处 unsafe
fn bad_function() {
    unsafe { /* 十多行 */ }
}

// 好的实践：隔离 unsafe，提供安全抽象
fn safe_wrapper() -> Result<T, E> {
    // 安全检查
    assert!(precondition_met);
    let result = unsafe { minimal_unsafe_operation() };
    // 安全处理结果
    Ok(result)
}
```

`unsafe` 块应尽可能小，且应在代码审查中被格外关注。如果整个函数体都需要 unsafe，应考虑是否有设计问题。

## 与所有权系统的关系

unsafe **不会关闭**借用检查器——只关闭了那五类检查。所以在 unsafe 块中使用 `&T` 引用时，借用规则仍然被强制执行。这种设计使得 unsafe 的爆炸半径受控：你只需要保证裸指针操作正确，其他部分 Rust 仍会替你检查。

## 注意事项

- **`unsafe` ≠ "危险"**：unsafe 代码完全可以安全运行，它只是告诉编译器"我相信自己，你不用检查了"
- **unsafe 是"扩容"而非"降级"**：安全 Rust 是 unsafe Rust 的子集，而非反过来
- **正确的 unsafe 实践更难验证**：因为编译器不帮你了，只能靠人工审查、测试和形式化验证工具（如 Miri）
- **FFI 是最常见的 unsafe 来源**：调用 C 库时，必须确保传过去的指针Valid、长度正确、生命周期不越界

## 相关页面

- [[所有权与借用]] — unsafe 中仍需遵守的规则，以及 unsafe 绕过的检查
- [[智能指针]] — 许多智能指针内部使用 unsafe 实现安全抽象
- [[Rust 并发模型]] — `Send`/`Sync` 是 unsafe trait 的典型例子
- [[Trait 系统]] — unsafe trait 实现需要手动保证安全承诺

## 原始来源

- [不安全 Rust](raw/trpl-zh-cn/src/ch20-01-unsafe-rust.md)
