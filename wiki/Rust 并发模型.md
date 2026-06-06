---
title: Rust 并发模型
date: 2026-06-02
tags:
  - layer/rust
  - type/concept
  - status/stable
aliases:
  - Fearless Concurrency
  - 无畏并发
  - Send Sync
---

# Rust 并发模型

> [!abstract] 摘要
> Rust 在**编译期**消除数据竞争——不是通过禁止并发，而是通过所有权系统和 `Send`/`Sync` 两个标记 trait 让编译器静态验证所有跨线程操作的安全性。这一机制支撑了三种并发范式：**线程 + move 语义**（所有权转移 = 天然的线程隔离）、**消息传递**（mpsc channel，数据通过发送转移所有权，杜绝竞争）、**共享状态**（`Arc<Mutex<T>>`，原子引用计数 + 互斥锁，编译器强制你获取锁才能访问数据）。三种范式并非互斥——你可以混合使用，而编译器始终在编译时验证安全性。这使得 Rust 在保证"不出现数据竞争"的同时，允许你充分利用多核并行能力。

## 数据竞争：Rust 解决的根本问题

**数据竞争**（Data Race）的定义是：两个或更多线程同时访问同一块内存，且至少有一个是写操作，且没有同步机制来协调访问顺序。它不同于竞态条件（Race Condition）——竞态条件是逻辑层面的时序依赖问题，任何语言都可能出现；数据竞争则是内存安全层面的问题，一旦发生就属于**未定义行为**。

传统语言的应对方案：

| 方案 | 代表语言 | 代价 |
|------|---------|------|
| 不加限制，程序员自己保证 | C / C++ | 极易出错，数据竞争导致的 bug 极难复现和调试 |
| 单线程事件循环，避免并行 | JavaScript | 放弃了多核并行能力 |
| 运行时锁 + GC | Java / Go | GC 保护内存不释放，锁保护数据一致性——但锁的正确使用全靠程序员 |
| 无共享消息传递 | Erlang / Elixir | 不允许共享内存，纯消息传递——消除了数据竞争但限制了某些模式 |

**Rust 的方案**：允许并行 + 允许共享 + 编译期验证安全性。你写多线程代码时，编译器会在编译时检查所有权和类型标记——如果代码存在数据竞争的可能性，它根本**编译不过**。这不是运行时检查（如 Java 的 `synchronized` 失败抛异常），更不是"建议"——是硬约束。

## 三种并发范式

Rust 标准库提供了三种并发原语，分别对应三种不同的编程模型。关键洞察：**三种范式的安全性都建立在所有权系统之上**。

### 1. 线程 + 所有权隔离

Rust 使用 1:1 线程模型：每个 `std::thread::spawn` 创建一个 OS 线程。线程之间默认**不共享任何数据**——因为所有权的存在，同一个值不可能同时被两个线程拥有。

```rust
use std::thread;

let handle = thread::spawn(|| {
    for i in 1..10 {
        println!("spawned: {i}");
    }
});

for i in 1..5 {
    println!("main: {i}");
}

handle.join().unwrap(); // 等待子线程结束，阻塞当前线程
```

#### move 闭包：所有权转移进线程

直接在线程闭包中捕获外部变量会导致**编译错误**——因为编译器不知道线程会执行多久，无法保证引用始终有效：

```rust
let v = vec![1, 2, 3];

// 编译错误：闭包可能比 v 活得更久，借用可能失效
let handle = thread::spawn(|| {
    println!("{:?}", v);
});
```

`move` 关键字强制闭包获取捕获变量的**所有权**，将数据"移交"给新线程：

```rust
let v = vec![1, 2, 3];

let handle = thread::spawn(move || {
    // v 的所有权已移入此闭包，主线程不能再使用 v
    println!("{:?}", v);
});

// println!("{:?}", v); // 编译错误：v 已被移动
handle.join().unwrap();
```

> move 闭包的本质：不是"让闭包变成移动语义"，而是**强制闭包按值捕获**所有环境变量。没有 move 时，编译器按最小权限原则，能借用就借用；move 则一律夺取所有权。

#### JoinHandle：等待与返回值

`thread::spawn` 返回 `JoinHandle<T>`，其中 `T` 是闭包的返回值类型。调用 `join()` 会阻塞当前线程，等待子线程结束，并返回线程的返回值。`join()` 的位置影响并发行为：放在主线程的其他工作之前 → 完全串行；放在之后 → 并行执行。

### 2. 消息传递："通过通信来共享内存"

Go 语言的名言 "Do not communicate by sharing memory; instead, share memory by communicating" 在 Rust 中通过 `std::sync::mpsc::channel` 实现。mpsc = **多生产者，单消费者**（multiple producer, single consumer）。

```rust
use std::sync::mpsc;
use std::thread;

let (tx, rx) = mpsc::channel(); // tx = 发送端, rx = 接收端

thread::spawn(move || {
    tx.send("hello".to_string()).unwrap();
    // tx 的所有权已移入此线程，发送端在此使用
});

let received = rx.recv().unwrap(); // 阻塞等待消息
println!("Got: {received}");
```

#### 信道的核心机制

- `send()` 获取参数的所有权并移动给接收端——发送后，发送者不能再使用该值。
- `recv()` 阻塞当前线程直到收到消息；`try_recv()` 非阻塞，立即返回 `Result`。
- 当发送端被 drop 时，信道关闭，接收端收到 `Err`。
- `rx` 实现了 `Iterator` trait——可以用 `for received in rx` 遍历所有消息，信道关闭时自动结束。

```rust
let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let messages = vec!["hi", "from", "the", "thread"];
    for msg in messages {
        tx.send(msg.to_string()).unwrap();
        thread::sleep(Duration::from_millis(500));
    }
});

for received in rx { // 迭代器自动等待
    println!("Got: {received}");
}
```

#### 多个生产者

`mpsc` 的 "mp" 在于发送端可以克隆（`tx.clone()`），每个克隆移入不同线程，所有消息汇聚到同一个接收端：

```rust
let (tx, rx) = mpsc::channel();
let tx1 = tx.clone();

thread::spawn(move || { tx1.send("from tx1").unwrap(); });
thread::spawn(move || { tx.send("from tx2").unwrap(); });

for msg in rx {
    println!("{msg}");
}
```

> [!note] 信道与所有权
> "发送即移交所有权"是消息传递安全性的核心：你在发送端发送了一个值后就不能再使用它——因为它的所有权已经转移给了接收端。这从语言层面防止了"发送后仍在本地修改"这类并发 bug。这与 Erlang/Elixir 的理念相同，但 Erlang 通过不可变数据实现，Rust 通过所有权实现。

### 3. 共享状态：`Mutex<T>` + `Arc<T>`

消息传递适合流水线式通信，但某些场景下多线程**需要读写同一块数据**——例如共享计数器、缓存、配置。Rust 通过 `Mutex<T>`（互斥锁）和 `Arc<T>`（原子引用计数）实现线程安全的共享。

#### Mutex<T>：类型系统强制你获取锁

``Mutex<T>` 包裹一个值，要访问内部数据必须先调用 `lock()` 获取锁。锁的释放是**自动的**——`MutexGuard`（lock 返回的智能指针）在离开作用域时自动解锁：

```rust
use std::sync::Mutex;

let m = Mutex::new(5);

{
    let mut num = m.lock().unwrap();
    *num = 6;
} // MutexGuard 离开作用域，锁自动释放

println!("m = {:?}", m.lock().unwrap()); // 可以再次获取锁
```

关键设计：你不能"忘记"获取锁——`Mutex<i32>` 和 `i32` 是不同的类型。类型系统在编译期阻止你绕过锁直接访问数据。

> `lock()` 返回 `LockResult<MutexGuard<T>>`。如果持有锁的线程 panic，`lock()` 返回 `Err`（毒化状态），防止使用可能不一致的数据。

#### 为什么需要 `Arc<T>` 而不能用 `Rc<T>`

要在多个线程间共享 `Mutex<T>` 的所有权，直觉上会想到引用计数——`Rc<T>`。但这**编译不过**：

```rust
let counter = Rc::new(Mutex::new(0)); // 编译错误！
// Rc<T> 没有实现 Send —— 不能安全地在线程间传递
```

`Rc<T>` 的引用计数操作不是原子性的——两个线程同时 clone/drop 可能导致计数错乱，造成内存泄漏或提前释放。`Arc<T>`（Atomic Reference Counted）使用原子操作更新引用计数，因此线程安全：

```rust
use std::sync::{Arc, Mutex};

let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter); // 原子地增加引用计数
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("Result: {}", *counter.lock().unwrap()); // 10
```

#### `Mutex<T>` 提供内部可变性

`counter` 是不可变绑定（`let counter = Arc::new(...)`），但通过 `Mutex<T>` 仍然可以修改其内部值。这与 `RefCell<T>` 的理念相同——**内部可变性**（Interior Mutability）。区别在于：
- `RefCell<T>` 在运行时检查借用规则（单线程），`Rc<T>` + `RefCell<T>` 组合用于单线程场景。
- `Mutex<T>` 在运行时通过锁保证互斥访问（多线程），`Arc<T>` + `Mutex<T>` 组合用于多线程场景。

| 单线程 | 多线程 |
|--------|--------|
| `Rc<T>` | `Arc<T>` |
| `RefCell<T>` | `Mutex<T>` (或 `RwLock<T>`) |

> [!warning] `Mutex<T>` 不能防止死锁
> Rust 的编译期检查只防止**数据竞争**，不防止**死锁**（两个线程互相等待对方释放锁）。死锁属于逻辑错误，任何支持锁的语言都存在。Rust 通过清楚的所有权模型让你更容易追踪锁的持有路径，但不能自动避免死锁。

#### 简单数值操作的更优选择：`std::sync::atomic`

对于 `i32`、`u64`、`bool` 等基本类型的原子操作，标准库的 `std::sync::atomic` 模块提供了比 `Mutex<T>` 更轻量的选择，如 `AtomicI32`、`AtomicBool`，它们使用 CPU 原子指令，无锁开销。

## Send 和 Sync：编译期线程安全标记

`Send` 和 `Sync` 是两个**标记 trait**（marker trait）——它们没有方法体，只用于向编译器表达"这个类型具备某种线程安全属性"。它们是 Rust 并发安全的理论基石。

### Send：可以安全转移所有权到另一个线程

```rust
// 伪代码：定义在 std::marker
pub unsafe auto trait Send {}
```

一个类型实现了 `Send`，意味着其值的所有权可以在线程间安全地移动。**几乎所有 Rust 类型都是 `Send` 的**，除了：

- `Rc<T>`：非原子引用计数，clone/drop 可能被打断导致计数错乱
- 裸指针（raw pointer）：`*const T`、`*mut T`——没有所有权规则保护
- 其他非 `Send` 类型组成的复合类型

**任何完全由 `Send` 类型组成的类型，编译器会自动为其实现 `Send`**。

### Sync：可以安全地在多个线程间共享引用

```rust
// 伪代码
pub unsafe auto trait Sync {}
```

一个类型实现了 `Sync`，意味着其不可变引用 `&T` 实现了 `Send`——即可以被安全地发送到另一个线程。换言之：**多个线程可以同时持有 `&T`**。

没有实现 `Sync` 的类型：
- `Rc<T>`：同 `Send`
- `RefCell<T>` 和 `Cell<T>`：运行时借用检查不是线程安全的——两个线程可能同时通过 `borrow_mut()` 获得可变引用
- 裸指针、`UnsafeCell<T>`（内部可变性的底层基础）

### 自动推导与手动实现

```rust
// Send 和 Sync 是 auto trait —— 编译器自动推导
struct MyStruct {
    x: i32,        // Send + Sync
    y: String,     // Send + Sync
}
// MyStruct 自动实现 Send + Sync

struct NotSync {
    cell: std::cell::RefCell<i32>, // !Sync
}
// NotSync 自动实现 Send，但不实现 Sync
```

通常不需要手动实现 `Send` 和 `Sync`。当创建包含非 `Send`/`Sync` 组件的自定义并发类型时，需要使用 `unsafe impl`——这要求你向编译器承诺"我知道自己在做什么，我保证安全性"。

## 跨领域连接

### ↔ Linux：从运行时线程到编译期安全

Rust 的并发原语直接映射到 Linux 的底层机制：

- **`std::thread::spawn` → `clone()` 系统调用**：Linux 中线程本质上是共享地址空间的"轻量级进程"，通过 `clone()` 创建。Rust 的 1:1 线程模型就是对 Linux 原生线程的薄封装。
- **`Mutex<T>` → 内核 futex**：Linux 上的 `Mutex<T>` 底层通常使用 futex（快速用户态互斥）——无竞争时在用户态完成锁定，有竞争时才陷入内核。这与 Linux 的互斥锁语义一致，区别在于 Rust 用 `lock()` 返回守卫、RAII 自动释放，而 C 中你手动 `pthread_mutex_lock`/`unlock`。
- **Send/Sync ≈ 编译期的线程安全验证**：Linux 在**运行时**提供同步原语（锁、条件变量、屏障），但正确使用它们全靠程序员。Rust 将这些验证提前到了**编译期**——Send/Sync 是"如果不符合要求就编译不过"的静态标注。这是一个根本性的范式转换：Linux 给你武器但不教你怎么用；Rust 在你开枪之前就检查你姿势是否正确。
- **任务调度**：Linux CFS 调度器决定哪个线程何时运行；Rust 标准库的线程模型完全不介入调度，完全交由 OS 管理。

### ↔ JavaScript：单线程事件循环 vs 多线程并行

JS 和 Rust 代表了解决数据竞争问题的两个极端：

| 维度 | JavaScript | Rust |
|------|-----------|------|
| 并发模型 | 单线程 + 事件循环 | 多线程 + 编译器验证 |
| 避免数据竞争的方式 | **不作并行**——同一时刻只有一个函数执行 | **编译期验证**——允许并行但禁止不安全操作 |
| 异步 | `Promise`（热启动）、微任务队列 | `Future`（惰性）、需外部执行器 |
| CPU 密集型任务 | 阻塞事件循环，需要用 Worker | 天然多线程，利用所有核心 |
| I/O 密集型任务 | 事件循环 + 回调 / async/await | tokio 等运行时 + async/await |

JS 的"单线程事件循环"本质上是通过**放弃并行来换取安全**——你永远不会有两个线程同时修改同一个变量，因为根本没有两个线程。Rust 则是"允许并行，但编译器替你盯着"。

> [!abstract] 实例对比：并发计数
>
> **JavaScript 版本**（单线程，通过异步模拟并发）：
>
> ```javascript
> async function concurrentCounter() {
>     let counter = 0;
>     const tasks = Array.from({ length: 10 }, () =>
>         new Promise(resolve => {
>             // 在事件循环的微任务中递增，但本质上是顺序执行
>             counter += 1;
>             resolve();
>         })
>     );
>     await Promise.all(tasks);
>     console.log(counter); // 一定是 10 —— 因为单线程
> }
> ```
>
> **Rust 版本**（多线程，真正并行）：
>
> ```rust
> use std::sync::{Arc, Mutex};
> use std::thread;
>
> let counter = Arc::new(Mutex::new(0));
> let mut handles = vec![];
>
> for _ in 0..10 {
>     let counter = Arc::clone(&counter);
>     handles.push(thread::spawn(move || {
>         let mut num = counter.lock().unwrap(); // 编译器强制你获取锁
>         *num += 1;
>     }));
> }
> // join all...
> ```
>
> JS 的答案：不并行，所以不需要锁。Rust 的答案：并行，但编译器保证你正确使用了锁。

### ↔ 软件工程：Actor 模型、消息传递与设计模式

Rust 的并发模型映射了几种经典的软件工程并发架构：

- **mpsc channel → Actor 模型**：Erlang/Elixir 的 Akka、Go 的 channel、Rust 的 mpsc——都是"不共享状态，通过消息通信"的 Actor 模式变体。区别在于 Erlang 进程是虚拟的（BEAM 调度），Rust 线程是真实的 OS 线程。
- **"Share memory by communicating"**：这不是 Rust 原创——来自 Go 的并发哲学。但 Rust 通过所有权系统**强制执行**这一原则：信道发送即移交所有权，发送后无法访问旧值。
- **`Arc<Mutex<T>>` ≈ 线程安全的共享集合**：类似于 Java 的 `synchronized` 集合或 `ConcurrentHashMap`，但 Rust 的包装更明确——你无法"忘了"加锁。
- **Send/Sync ≈ C++ 的 `is_trivially_copyable` 等类型属性**：都是编译期类型标记，但 Send/Sync 服务于并发安全，C++ 的属性服务于内存布局。

## 相关页面

- [[Rust 概述]] — Rust 语言全景，本页是并发分支的深入
- [[所有权与借用]] — 所有权系统是一切并发安全的基础
- [[Trait 系统]] — Send/Sync 的本质是标记 trait
- [[智能指针]] — `Rc<T>` / `Arc<T>` / `RefCell<T>` / `Mutex<T>` 的关系
- [[Linux 进程模型]] — 线程、进程隔离、虚拟内存，并发安全的 OS 基础
- [[JavaScript Promise 与异步]] — JS 单线程事件循环 vs Rust 多线程并行，两种解决数据竞争的对立方案
- [[并发模型对比]] — Linux/JS/Rust/Cocos 四种并发模型的调度、隔离、通信三轴全景对比
- [[软件工程概述]] — Actor 模型、消息传递、线程安全的架构模式

## 原始来源

- [使用线程同时运行代码](raw/trpl-zh-cn/src/ch16-01-threads.md)
- [使用消息传递在线程间传送数据](raw/trpl-zh-cn/src/ch16-02-message-passing.md)
- [共享状态并发](raw/trpl-zh-cn/src/ch16-03-shared-state.md)
- [使用 Send 和 Sync trait 的可扩展并发](raw/trpl-zh-cn/src/ch16-04-extensible-concurrency-sync-and-send.md)
