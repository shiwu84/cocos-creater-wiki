---
title: Trait 系统
date: 2026-06-02
tags:
  - layer/rust
  - type/concept
  - status/stable
aliases:
  - Trait
  - trait
---

# Trait 系统

> [!abstract] 摘要
> Trait 是 Rust 的接口抽象机制——定义类型间可共享的行为。它融合了 Haskell typeclass 的"类型能力声明"和 OOP 接口的"行为契约"思想，并增加了 Rust 独有的特性：**孤儿规则**确保 trait 实现的全局一致性，**blanket implementation** 允许为满足条件的全体类型一次性实现 trait，**默认实现**让 trait 可以提供部分行为而只需实现者覆盖核心方法。Trait 是 Rust 中唯一的多态机制——Rust 没有继承，所有代码复用和多态都通过 trait 组织。在编译层，trait 约束驱动**单态化**（静态分发，零运行时开销）；在运行时，`dyn Trait` 通过 **vtable** 实现动态分发，让同质容器容纳异质类型。Trait 同时也是 Rust 运算符重载、闭包类型推导、错误传播（`?` 运算符）、并发安全标记（Send/Sync）等一切语言特性的底层机制。

## 根本问题：没有继承，如何共享行为？

面向对象语言（Java、C++、C#）通过**类继承**实现行为共享：子类继承父类的方法，重写需要定制的方法。这套模型有三个问题：

1. **强耦合**：子类依赖父类的实现细节，父类修改可能破坏子类
2. **菱形继承**：多重继承导致"钻石问题"，需要虚继承、接口等机制打补丁
3. **不适合组合**：现实中的行为共享往往是正交的——"能飞"、"能游泳"、"能序列化"这些能力不应绑定到一条继承链上

Rust 的答案是**完全放弃继承，只用 trait**：

- Trait 只定义**行为签名**，不定义数据结构——数据由 struct/enum 负责
- 一个类型可以实现任意多个 trait——每个 trait 代表一个独立的行为维度
- Trait 实现与类型声明分离——可以为已存在的类型添加新行为（受孤儿规则约束）

```rust
// Trait 定义行为（接口），struct 定义数据
trait Fly { fn fly(&self); }
trait Swim { fn swim(&self); }

struct Duck { name: String }

// Duck 同时实现 Fly 和 Swim——正交组合，无继承链
impl Fly for Duck {
    fn fly(&self) { println!("{} flies", self.name); }
}
impl Swim for Duck {
    fn swim(&self) { println!("{} swims", self.name); }
}
```

> 这与 [[JavaScript 原型与继承|JS 的原型链]]形成鲜明对比：JS 通过原型链实现行为共享（运行时查找），Rust 通过 trait 实现行为共享（编译期验证）。两者都不依赖传统类继承，但 Rust 的方案在编译期就完成了所有检查。

## 定义与实现 Trait

### 定义 trait

使用 `trait` 关键字定义一组方法签名，方法体用分号代替：

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

> 注意：`pub` 控制 trait 本身的可见性，让外部 crate 可以引用该 trait。方法签名默认为 trait 的可见性一致。

### 实现 trait

`impl <Trait> for <Type>` 语法为目标类型实现 trait：

```rust
pub struct NewsArticle {
    pub headline: String,
    pub author: String,
}

pub struct SocialPost {
    pub username: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {}", self.headline, self.author)
    }
}

impl Summary for SocialPost {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

调用 trait 方法时必须将 trait 引入作用域——这避免了不同 crate 的 trait 命名冲突：

```rust
use aggregator::Summary;

let post = SocialPost { /* ... */ };
println!("{}", post.summarize());  // Summary 必须在作用域才能调用 summarize
```

### 默认实现

Trait 可以为方法提供默认实现，实现者可以选择保留或覆盖：

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    // 默认实现：依赖 summarize_author
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

// 对外部来说只需实现 summarize_author，summarize 自动可用
impl Summary for SocialPost {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```

> 默认实现可以调用 trait 中的其他方法（包括未提供默认实现的方法），这允许 trait 定义一套"模板方法"——只需实现者提供少量核心方法，派生方法自动生成。

## Trait 约束：约束泛型

Trait 的核心价值在于**约束泛型参数**——限制泛型 `T` 只能是实现了特定 trait 的类型：

```rust
// impl Trait 语法（语法糖）
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// 等价的长形式——trait bound
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

### impl Trait vs Trait Bound 的区别

| 场景 | impl Trait | Trait Bound |
|------|-----------|-------------|
| 单个参数 | `fn f(x: &impl Summary)` | `fn f<T: Summary>(x: &T)` |
| 多个参数允许不同类型 | `fn f(x: &impl Summary, y: &impl Summary)` | — |
| 多个参数强制相同类型 | — | `fn f<T: Summary>(x: &T, y: &T)` |
| 多个 trait 约束 | `&(impl Summary + Display)` | `<T: Summary + Display>` |
| 复杂约束（where） | — | `where` 从句 |

### 多个 trait 约束

使用 `+` 组合多个 trait，或使用 `where` 简化复杂签名：

```rust
// + 语法
fn notify(item: &(impl Summary + Display)) { }
fn notify<T: Summary + Display>(item: &T) { }

// where 从句——避免签名臃肿
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // ...
}
```

### impl Trait 在返回位置

函数可以返回"实现了某 trait 但不暴露具体类型"的值：

```rust
fn returns_summarizable() -> impl Summary {
    SocialPost {
        username: String::from("horse_ebooks"),
        content: String::from("of course..."),
    }
}
```

> [!warning] 限制：只能返回单一具体类型
> `impl Trait` 在返回位置要求函数所有代码路径返回**同一种具体类型**。以下代码不能编译：
> ```rust
> fn returns_summarizable(switch: bool) -> impl Summary {
>     if switch {
>         NewsArticle { /* ... */ }
>     } else {
>         SocialPost { /* ... */ }  // 错误：类型不匹配
>     }
> }
> ```
> 如果需要返回多种类型，使用 **trait object**（`Box<dyn Summary>`）替代。

### 有条件地实现方法

利用 trait 约束，只在泛型参数满足特定条件时才实现方法：

```rust
struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self { Self { x, y } }
}

// 只有 T 实现了 Display + PartialOrd 时，cmp_display 才可用
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

## 静态分发 vs 动态分发

这是理解 trait 在 Rust 中工作方式的核心概念。

### 静态分发：单态化（Monomorphization）

当 trait 通过泛型使用时，编译器为**每种具体类型**生成一份独立的函数副本——这个过程叫**单态化**：

```rust
fn notify<T: Summary>(item: &T) {
    println!("{}", item.summarize());
}

// 编译器实际生成（伪代码）：
// fn notify_NewsArticle(item: &NewsArticle) { ... }
// fn notify_SocialPost(item: &SocialPost) { ... }
```

| 优点 | 缺点 |
|------|------|
| **零运行时开销**——编译器知道确切类型，可直接内联 | **代码膨胀**——每种类型一份代码 |
| **编译器可深度优化**——内联、死代码消除等 | **编译时间变长** |
| 适用于同质集合（所有元素同一类型） | 集合中只能放同一类型 |

### 动态分发：Trait Object

使用 `dyn Trait` 创建 trait object，在**运行时**通过 vtable 决定调用哪个方法：

```rust
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,  // trait object：异质集合
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();  // 运行时通过 vtable 分发到正确实现
        }
    }
}

trait Draw {
    fn draw(&self);
}

struct Button { /* ... */ }
impl Draw for Button { fn draw(&self) { /* ... */ } }

struct SelectBox { /* ... */ }
impl Draw for SelectBox { fn draw(&self) { /* ... */ } }
```

Trait object 的内存布局（对 `Box<dyn Draw>` 而言）：

```
┌─────────────────────────┐
│  data pointer ────────→ │ 实际数据（Button / SelectBox）
│  vtable pointer ──────→ │ vtable: [drop, draw, size, align]
└─────────────────────────┘
```

每个 trait object 包含两个指针：
- **数据指针**：指向堆上或栈上的实际类型实例
- **vtable 指针**：指向一张虚函数表（vtable），表中存储该类型对 trait 中每个方法的实现地址

> Trait object 必须通过指针使用（`&dyn Trait`、`Box<dyn Trait>`、`Arc<dyn Trait>` 等），因为 trait object 的大小在编译期未知（不同具体类型大小不同）。这与 Rust 的 `Sized` trait 机制相关——trait object 是不可知大小类型（DST），不能直接放在栈上。

### 选择指南

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| 集合中所有元素类型相同 | 泛型 + trait bound | 静态分发，零开销，编译器可优化 |
| 集合中需要容纳不同类型 | `Vec<Box<dyn Trait>>` | 动态分发，运行时灵活 |
| 函数需要处理多种类型但每次只处理一种 | 泛型 + trait bound | 性能更优 |
| 库需要让用户扩展类型集合 | trait object | 库不预知所有类型，用户可自由添加 |

## 关联类型 vs 泛型

当 trait 需要引用"某种类型"但该类型因实现而异时，有两种方案：

### 关联类型（Associated Type）

将类型占位符定义为 trait 的一部分，每个实现只能指定一次：

```rust
pub trait Iterator {
    type Item;                          // 关联类型——占位符

    fn next(&mut self) -> Option<Self::Item>;
}

impl Iterator for Counter {
    type Item = u32;                    // 指定 Item 为 u32——只能指定一次

    fn next(&mut self) -> Option<Self::Item> {
        Some(0)
    }
}

// 调用方不需要标注类型
let mut counter = Counter {};
counter.next();  // 返回 Option<u32>，无需类型注解
```

### 泛型参数

将类型参数放在 trait 上，允许多种实现：

```rust
trait Iterator<T> {                     // 泛型参数
    fn next(&mut self) -> Option<T>;
}

impl Iterator<String> for Counter {     // Counter 可以实现多个 Iterator
    fn next(&mut self) -> Option<String> { Some(String::new()) }
}
impl Iterator<u32> for Counter {        // 同一类型的多个实现
    fn next(&mut self) -> Option<u32> { Some(0) }
}

// 调用方必须标注要使用哪个实现
<Counter as Iterator<u32>>::next(&mut counter);
```

### 何时用哪个？

| 关联类型 | 泛型参数 |
|---------|---------|
| 一个类型对每个 trait 只有一个实现 | 一个类型可以有多个实现（不同泛型参数） |
| 调用方无需标注类型 | 调用方可能需要类型注解消歧义 |
| 适合"一个类型有一组固定的输出类型" | 适合"同一操作可作用于多种输入类型" |

> `Add` trait 是一个混合例子——它既有泛型参数 `Rhs`（默认 `= Self`，允许 `Point + Point` 和 `Millimeters + Meters` 两种实现），又有关联类型 `Output`（每次实现只一个输出类型）。

## 超 trait（Supertrait）

当 trait A 依赖 trait B 的功能时，A 以 B 作为前提：

```rust
use std::fmt;

// OutlinePrint 只能在实现了 Display 的类型上实现
trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();  // 可以调用 Display 的方法
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

`trait OutlinePrint: fmt::Display` 的语义是：任何实现 `OutlinePrint` 的类型**必须**同时实现 `Display`。这类似于 trait bound 应用于 trait 本身。

## Blanket Implementation

为满足特定 trait 约束的**所有类型**一次性实现另一个 trait：

```rust
// 标准库中的经典例子：任何实现了 Display 的类型都可以 ToString
impl<T: Display> ToString for T {
    // T 是任意类型，只要它实现了 Display
}

// 结果：i32 实现了 Display → i32 自动获得 to_string()
let s = 3.to_string();  // "3"
```

这是 Rust 标准库中大量便利功能的基础。`ToString`、`IntoIterator`、`From` 等双向 trait 配合提供了很多"自动获得"的行为。

## 孤儿规则（Coherence / Orphan Rule）

孤儿规则是 Rust 类型系统中维持一致性的关键规则：

> **只有当 trait 或类型至少有一方在当前 crate 中定义时，才能为该类型实现该 trait。**

```
                    trait 在本地 crate   trait 在外部 crate
类型在本地 crate        ✅ 允许              ✅ 允许
类型在外部 crate        ✅ 允许              ❌ 禁止（孤儿规则）
```

```rust
// ✅ trait 本地：在自己 crate 为 Vec<T> 实现自己的 Summary
impl Summary for Vec<String> { }

// ❌ 双方都在外部：不能为标准库的 Vec<T> 实现标准库的 Display
// impl Display for Vec<String> { }  // 编译错误

// ✅ 类型本地：外部 trait（Display） + 本地类型（Wrapper）
struct Wrapper(Vec<String>);
impl Display for Wrapper { /* ... */ }
```

### 为什么需要孤儿规则？

没有孤儿规则，两个不相关的 crate（A 和 B）都可以为 `Vec<T>` 实现 `Display`。当第三个 crate C 同时依赖 A 和 B 时，编译器不知道该使用哪个 `Display` 实现——出现了**实现冲突**。

### Newtype 模式

孤儿规则的一个标准绕过方案：用元组结构体包装外部类型，使其成为本地类型：

```rust
struct Wrapper(Vec<String>);  // Wrapper 是本地的

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}
```

> `Wrapper` 不自动拥有内部类型的方法。可通过实现 `Deref` trait 来"透传"内部类型的方法，详见 [[智能指针]]。

## Derive 宏

许多标准库 trait 可以通过 `#[derive(...)]` 属性自动生成实现，无需手写样板代码：

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct Point {
    x: i32,
    y: i32,
}

// 编译器自动生成：
// - Debug::fmt —— 调试打印 "{:?}" 和 "{:#?}"
// - Clone::clone —— 显式深拷贝
// - PartialEq::eq / ne —— == 和 !=
// - Eq —— 标记该类型的相等性满足自反、对称、传递
// - Hash::hash —— 作为 HashMap/HashSet 的键
```

> Derive 宏是一种**过程宏**——编译期根据结构体/枚举的字段自动生成 impl 块。可 derive 的标准 trait 包括：`Debug`、`Clone`、`Copy`、`PartialEq`、`Eq`、`PartialOrd`、`Ord`、`Hash`、`Default`。

### `Copy` 和 `Clone` 的关系

```rust
#[derive(Copy, Clone)]  // Copy 是 Clone 的超 trait，两者常一起 derive
struct Point { x: i32, y: i32 }

let p1 = Point { x: 1, y: 2 };
let p2 = p1;           // 按位复制（Copy），p1 仍然有效
let p3 = p1.clone();   // 显式克隆（Clone），同样有效
```

> `Copy` 只能 derive 在全部字段都是 `Copy` 的类型上，且该类型不能实现 `Drop`。

## 常用标准库 Trait

这些 trait 构成 Rust 日常编程的"基础设施"——几乎所有类型都至少实现了其中一部分：

| Trait | 用途 | 典型实现方式 | 关键点 |
|-------|------|-------------|--------|
| `Debug` | 调试输出 `{:?}` | `derive` 或手动 | 所有公开类型都应实现 |
| `Display` | 用户友好输出 `{}` | 手动实现 | 面向用户，不是调试 |
| `Clone` | 显式深拷贝 `.clone()` | `derive` 或手动 | 可能有开销，显式调用 |
| `Copy` | 按位复制（栈上数据） | `derive` | 隐式触发，不能有 Drop |
| `PartialEq` | 部分相等 `==`、`!=` | `derive` 或手动 | NaN != NaN 是典型反例 |
| `Eq` | 完全相等（自反性） | `derive` | 标记型 trait，无方法 |
| `PartialOrd` | 部分排序 `<`、`>` | `derive` 或手动 | NaN 无法比较 |
| `Ord` | 全序 | `derive` | 要求 Eq + PartialOrd |
| `Hash` | 哈希 `HashMap` 键 | `derive` 或手动 | 若 `a == b` 则 `hash(a) == hash(b)` |
| `Default` | 默认值 `Default::default()` | `derive` 或手动 | `unwrap_or_default()` |
| `From`/`Into` | 类型转换 `.into()` | 手动实现 From | 实现了 `From<A> for B` 自动获得 `Into<B> for A` |
| `Drop` | 析构时清理资源 | 手动实现 | 离开作用域自动调用 |
| `Iterator` | 数据流迭代 | 手动实现 `next()` | 一旦实现了 `next()`，数十个适配器方法自动可用 |

> [!tip] 重要的 trait 关系
> - `Copy: Clone` —— Copy 是 Clone 的 supertrait
> - `Eq: PartialEq` —— Eq 要求完全的相等性（无 NaN 一类例外）
> - `Ord: Eq + PartialOrd`
> - `Into<B> for A` —— 只要实现了 `From<A> for B`，标准库自动提供
> - 关闭 `File` 时自动 `Drop` —— 等价于 RAII（资源获取即初始化），与 Linux 文件描述符的 fd 自动回收机制同源

## 高级用法

### 运算符重载

Rust 通过 trait 实现运算符重载——标准库的 `std::ops` 模块定义了一系列 trait：

```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point { x: i32, y: i32 }

impl Add for Point {
    type Output = Point;  // 关联类型：加法的输出类型

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

let p = Point { x: 1, y: 0 } + Point { x: 2, y: 3 };
```

### 方法消歧义（完全限定语法）

当一个类型实现了多个 trait 且它们有同名方法时，需要显式指定调用哪个：

```rust
trait Pilot { fn fly(&self); }
trait Wizard { fn fly(&self); }

struct Human;
impl Human { fn fly(&self) { println!("*waving arms*"); } }
impl Pilot for Human { fn fly(&self) { println!("This is your captain."); } }
impl Wizard for Human { fn fly(&self) { println!("Up!"); } }

let person = Human;

person.fly();                    // 调用 Human 自己的方法（默认）
Pilot::fly(&person);             // 调用 Pilot trait 的实现
<Human as Wizard>::fly(&person); // 完全限定语法
```

> 完全限定语法：`<Type as Trait>::function(...)`。对于关联函数（无 `self`），当 Rust 无法从上下文推断时，必须使用此语法。

## 跨领域连接

### ↔ TypeScript：Trait ≈ interface + 泛型约束

TypeScript 的 `interface` 和 Rust 的 trait 都定义了"类型必须具备的能力"，但机制不同：

| 维度 | TypeScript | Rust |
|------|-----------|------|
| 类型兼容 | **结构化类型**（鸭子类型）——形状匹配即兼容，无需显式声明 | **标称类型**——必须显式 `impl Trait for Type` |
| 泛型 | 运行时类型擦除 | 编译期单态化，零成本 |
| 孤儿规则 | 无——声明合并（declaration merging）可补充外部接口 | 严格执行，保证全局一致性 |
| 约束语法 | `T extends Interface` | `T: Trait` |
| 动态分发 | 无内置 trait object；需手动实现 discriminator | `dyn Trait` 原生支持 |

> TypeScript 的 declaration merging 允许跨文件扩展接口（相当于在所有 crate 中"补丁"修改外部接口），这提供了灵活性但也潜在引入冲突。Rust 的孤儿规则选择了"一致性优先"——严格限制但消除了冲突可能。

### ↔ Java / C#：trait vs interface vs abstract class

| 特性 | Rust trait | Java interface | Java abstract class | C# interface |
|------|-----------|----------------|---------------------|--------------|
| 方法默认实现 | ✅ | ✅ (Java 8+) | ✅ | ✅ (C# 8+) |
| 关联类型 | ✅ | ❌ | ❌ | ❌ |
| 可包含数据 | ❌（通过关联类型引用数据） | ❌（仅常量） | ✅ | ❌ |
| 多实现 | ✅ | ✅ | ❌（单继承） | ✅ |
| 孤儿规则 | ✅ | ❌ | — | ❌ |
| 运算符重载 | ✅ | ❌ | ❌ | ✅ |
| blanket impl | ✅ | ❌ | ❌ | ❌ |

> Rust 没有类继承——trait 承担了 Java/C# 中 interface + abstract class 的全部职责。Trait 的能力远超 OOP 语言的接口，也正因为如此，Rust 不需要"抽象类"这个中间概念。

### ↔ Haskell：trait ≈ typeclass

Rust 的 trait 直接受 Haskell typeclass 启发，两者概念高度相似：

| 特性 | Rust trait | Haskell typeclass |
|------|-----------|-------------------|
| 行为定义 | `trait Show { fn show(&self) -> String; }` | `class Show a where show :: a -> String` |
| 实例声明 | `impl Show for Point { ... }` | `instance Show Point where ...` |
| 约束 | `fn f<T: Show>(x: &T)` | `f :: Show a => a -> IO ()` |
| 孤儿规则 | 有 | 有（类似规则，GHC 允许孤儿实例但会警告） |
| 返回类型多态 | `fn f() -> impl Show` | `f :: Show a => a`（但含义不同——Haskell 的调用者选类型） |
| 关联类型 | `type Item;` | `type Item`（associated type family） |

> 根本区别：Haskell 的 typeclass 实例是全局唯一的（无命名），而 Rust 的 trait 实例通过 `impl Trait for Type` 显式命名。Rust 的孤儿规则比 Haskell 更严格——Haskell 允许孤儿实例但依赖编译器警告。

### → 软件工程：Dependency Inversion Principle

Trait 是 Rust 中实践 SOLID 原则的核心工具，尤其是**依赖反转原则（DIP）**：

> "依赖抽象而非具体实现"——在 Rust 中表现为"依赖 trait 而非具体类型"。

```rust
// ❌ 依赖具体类型
fn notify(article: &NewsArticle) { /* 只能处理 NewsArticle */ }

// ✅ 依赖 trait 抽象
fn notify(item: &impl Summary) { /* 任何实现了 Summary 的类型 */ }
```

| SWE 模式 | Rust trait 对应 |
|----------|----------------|
| **依赖反转（DIP）** | 函数参数使用 `impl Trait` / `<T: Trait>` |
| **策略模式（Strategy）** | Trait objects：`Box<dyn Strategy>` 在运行时切换算法 |
| **装饰器模式（Decorator）** | Newtype 包装 + trait 实现 + Deref |
| **工厂模式（Factory）** | 返回 `impl Trait` 的函数作为工厂 |

> Trait object 本质上是**编译期安全的策略模式**——`dyn Trait` 允许在运行时切换行为实现，而编译器确保所有切换都是类型安全的。

### ← JavaScript：从鸭子类型到 trait

JavaScript 的"鸭子类型"哲学与 Rust 的 trait 约束在根本问题上是相通的：

```js
// JS：鸭子类型——"如果它能 quack，它就是鸭子"
function makeItQuack(duck) {
    duck.quack();  // 运行时才检查 quack 方法是否存在
}
```

```rust
// Rust：trait 约束——"如果它实现了 Quack trait，它就是鸭子"
fn make_it_quack(duck: &impl Quack) {
    duck.quack();  // 编译期验证 quack 存在
}
```

| 维度 | JavaScript 鸭子类型 | Rust trait |
|------|-------------------|------------|
| 验证时机 | **运行时**——调用不存在的方法导致 TypeError | **编译期**——方法缺失导致编译错误 |
| 表达意图 | 隐式（靠文档/注释） | 显式（trait 签名即契约） |
| 多态方式 | 原型链 + 鸭子类型 | trait + 静态/动态分发 |
| 安全代价 | 运行时错误 + 类型检查 | 编译期约束 + 更严格的编码 |

> 两者都承认"类型的行为由其方法决定"，但 Rust 选择在编译期执行这一哲学，让运行时错误的可能性降为零。

## 常见陷阱

> [!warning] 使用外部 trait 的方法前必须导入
> ```rust
> // ❌ 错误：Summary 不在作用域，不能调用 summarize
> let post = SocialPost { /* ... */ };
> post.summarize();  // 编译错误
> 
> // ✅ 正确
> use aggregator::Summary;
> post.summarize();
> ```

> [!warning] impl Trait 返回位置不能返回不同具体类型
> ```rust
> // ❌ 错误：返回两种不同类型
> fn f(flag: bool) -> impl Display {
>     if flag { 1i32 } else { "hello" }
>     //           ^^^^        ^^^^^^^ 不同类型！
> }
> 
> // ✅ 改用 trait object
> fn f(flag: bool) -> Box<dyn Display> {
>     if flag { Box::new(1i32) } else { Box::new("hello") }
> }
> ```

> [!warning] trait object 不支持所有 trait——需要 dyn 兼容性
> 不是所有 trait 都可以作为 trait object。简单判断：trait 中所有方法如果包含泛型参数、或返回 `Self`（而非 `&self` 等引用）、或使用了 `impl Trait`，则该 trait 不是 dyn 兼容的（以前叫 object-safe）。`Clone` 就是一个典型反例——`clone()` 返回 `Self`，所以不能有 `dyn Clone`。

> [!warning] 孤儿规则的两难
> 不能为外部类型实现外部 trait——这对 `Vec<T>` + `Display`、`Result<T, E>` + 各种自定义 trait 是常见痛点。解决方案：newtype 包装。代价是需要重构调用代码以使用 Wrapper 类型，或实现 `Deref` 透传。

## 相关页面

### 本领域（layer/rust）
- [[Rust 概述]] — Rust 语言全景，trait 是其多态机制的核心
- [[所有权与借用]] — 所有权规则是所有 trait 设计的基础（Drop、Copy、Clone 皆与所有权相关）
- [[生命周期]] — 生命周期标注在 trait 定义与 trait object 中的角色（待创建）
- [[泛型 (Rust)]] — 泛型与 trait 约束的深度配合（待创建）
- [[Rust 并发模型]] — Send/Sync 是最核心的两个标记型 trait（待创建）
- [[智能指针]] — Deref 和 Drop trait 的应用场景（待创建）

### 跨链共鸣
- [[TypeScript 概述]] — TypeScript 的 interface 与 Rust trait 的对比：结构类型 vs 标称类型
- [[Rust 概述]] — 系统编程链终端，trait 在该体系中的位置

### 软件工程横向层
- [[软件工程概述]] — 依赖反转原则、策略模式在 Rust trait 中的体现

### 前提层
- [[JavaScript 原型与继承]] — 原型链 vs trait 的不同行为共享机制
- [[JavaScript 教程概述]] — JS 的鸭子类型 vs trait 的编译期约束

## 原始来源

- [Trait：定义共同行为](raw/trpl-zh-cn/src/ch10-02-traits.md) — trait 定义、实现、默认实现、trait 约束、impl Trait 语法、blanket implementation、孤儿规则
- [使用 trait object 来抽象出共享行为](raw/trpl-zh-cn/src/ch18-02-trait-objects.md) — trait object、动态分发、dyn 兼容性、与泛型静态分发的对比
- [高级 trait](raw/trpl-zh-cn/src/ch20-02-advanced-traits.md) — 关联类型、默认泛型参数与运算符重载、方法消歧义、supertrait、newtype 模式
