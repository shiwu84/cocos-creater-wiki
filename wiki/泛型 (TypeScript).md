---
title: 泛型 (TypeScript)
date: 2026-06-02
tags:
  - layer/ts
  - type/concept
  - status/stable
aliases:
  - TS 泛型
  - TypeScript Generics
  - 类型参数
---

# 泛型 (TypeScript)

> [!abstract] 摘要
> 泛型（Generics）让类型本身变成**可参数化的**——函数、接口、类不再操作具体的类型，而是操作**类型变量**，由调用方提供实际类型。它解决了软件工程中的核心问题：如何编写一个能用于任意类型的组件，同时又**不丢失类型信息**？`any` 也能接收任意类型，但它丢弃了类型——函数的输入输出断了联系。泛型捕获并保留这种联系：`identity<T>(x: T): T` 明确地说了"输出类型就是输入类型"。TypeScript 的泛型受 C#/Java 启发，但运行在**结构化类型系统**之上——这使得 TS 泛型天然比名义类型系统（如 Java）的泛型更灵活，几乎不需要考虑协变/逆变等复杂问题。泛型是类型层面的**函数应用**：`Pick<Todo, "title">` 就像把 `Todo` 和 `"title"` 作为参数传给类型函数 `Pick`，得到一个新的类型。TypeScript 内置了大量基于泛型的**工具类型**（`Partial`、`Readonly`、`Record`、`ReturnType` 等），它们本质上是标准库里的"类型级函数"。

## 根本问题：既要通用，又要精确

没有泛型时，编写可复用的函数有两种选择——都令人不满意：

```typescript
// 方案 A：固定具体类型——完全不通用
function identity(arg: number): number {
  return arg;
}
// 只能用于 number，来个 string 就报错

// 方案 B：使用 any——丢失类型信息
function identity(arg: any): any {
  return arg;
}
// 任何类型都能用，但返回值和输入值之间的类型关联完全断了
const result = identity("hello");
// result 的类型是 any——编译器不知道它是 string
```

方案 A 牺牲了通用性，方案 B 牺牲了精确性。泛型同时解决这两个问题：

```typescript
function identity<T>(arg: T): T {
  return arg;
}

const result = identity("hello");
// result 的类型是 "hello" (字面量类型)，至少是 string
// 编译器知道：输入什么类型，输出就是什么类型
```

这里 `T` 是一个**类型变量**（type variable）——它像函数参数，但操作的是**类型**而非值。调用时 `T` 被实际类型替换（推断或显式指定），建立起输入和输出之间的**类型级关联**。

> 泛型的核心价值不是"能接受任意类型"（`any` 也能），而是"接受任意类型的同时**保持类型关系**"。

## 泛型的写法

### 基本泛型函数

```typescript
// 声明泛型函数
function identity<T>(arg: T): T {
  return arg;
}

// 两种调用方式：
identity<string>("hello");  // 显式指定类型参数
identity("hello");          // 类型参数推断——最常用
```

类型参数推断（type argument inference）让编译器从传入的实参自动推导 `T`。大多数场景下无需显式写 `<string>`，但复杂场景（如编译器无法推导时）需要显式指定。

### 操作泛型变量

泛型函数内部，编译器**只允许**所有类型共有的操作：

```typescript
function loggingIdentity<T>(arg: T): T {
  console.log(arg.length);  // 错误！T 不一定有 .length
  return arg;
}
```

因为 `T` 可能是 `number`——它没有 `length`。解决方法：约束 `T` 满足某种形状，或用泛型作为具体类型的**一部分**：

```typescript
// 方式1：将泛型嵌入到具体类型结构中
function loggingIdentity<T>(arg: T[]): T[] {
  console.log(arg.length);  // OK——数组一定有 length
  return arg;
}

// 等价写法
function loggingIdentity<T>(arg: Array<T>): Array<T> {
  console.log(arg.length);
  return arg;
}
```

### 泛型类型别名与接口

泛型函数自身的类型也是泛型的：

```typescript
// 泛型函数类型
let myIdentity: <T>(arg: T) => T = identity;

// 泛型接口——类型参数在调用签名上（描述泛型函数）
interface GenericIdentityFn {
  <T>(arg: T): T;
}

// 泛型接口——类型参数在接口上（描述泛型类型）
interface GenericContainer<T> {
  value: T;
  get: () => T;
}
// 使用时必须指定 T：GenericContainer<string>
```

> **关键区分**：类型参数放在调用签名上 → 描述的是"泛型函数"（具体类型由每次调用决定）；类型参数放在接口/类型别名本身上 → 描述的是"泛型类型"（具体类型在引用该类型时一次性确定）。

### 泛型类

```typescript
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
}

const myNum = new GenericNumber<number>();
myNum.zeroValue = 0;
myNum.add = (x, y) => x + y;

// 同样的类结构，不同的类型参数
const myStr = new GenericNumber<string>();
myStr.zeroValue = "";
myStr.add = (x, y) => x + y;  // 字符串拼接
```

> 注意：泛型类只在**实例侧**是泛型的——静态成员**不能**使用类的类型参数。

## 泛型约束：限制 T 的范围

当需要访问 `T` 的特定属性时，通过 `extends` 添加约束：

```typescript
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);  // OK——T 必须满足 Lengthwise
  return arg;
}

loggingIdentity({ length: 10, value: 3 });  // OK
loggingIdentity("hello");                     // OK——string 有 .length
loggingIdentity(3);                           // 错误！number 没有 .length
```

`T extends Lengthwise` 的意思是"`T` 必须是 `Lengthwise` 的子类型"——在 TS 的结构化类型系统中，只要 `T` 的结构**包含** `length: number` 即可，不需要显式声明 `implements Lengthwise`。

### 类型参数约束类型参数

泛型约束中可以使用另一个类型参数：

```typescript
// K 被约束为 T 的键
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const obj = { a: 1, b: 2, c: 3 };
getProperty(obj, "a");  // OK——返回类型是 number
getProperty(obj, "m");  // 编译错误！"m" 不是 obj 的键
```

这是泛型最强大的模式之一：`K extends keyof T` 确保传入的 `key` 一定是 `T` 的属性名，且返回类型 `T[K]` 精确对应**该属性的类型**——返回 `obj.a` 就是 `number`，返回 `obj.b` 也是 `number`。

### 泛型参数默认值

```typescript
declare function create<T extends HTMLElement = HTMLDivElement, U extends HTMLElement[] = T[]>(
  element?: T,
  children?: U
): Container<T, U>;

const div = create();                          // Container<HTMLDivElement, HTMLDivElement[]>
const p = create(new HTMLParagraphElement());  // Container<HTMLParagraphElement, HTMLParagraphElement[]>
```

规则：
- 有默认值的类型参数是可选的
- 必需的类型参数不能位于可选类型参数之后
- 默认类型必须满足约束条件
- 推断优先于默认值（只有当推断失败时才回退到默认值）

## 泛型与类类型

创建工厂函数时，需要用构造函数签名来描述类型：

```typescript
// { new(): T } 表示"一个返回 T 类型实例的构造函数"
function create<T>(c: { new(): T }): T {
  return new c();
}
```

更高级的用法——结合原型链推断：

```typescript
class Animal { numLegs = 4; }
class Lion extends Animal { keeper: ZooKeeper; }
class Bee extends Animal { keeper: BeeKeeper; }

function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}

createInstance(Lion).keeper.nametag;  // 类型精准推断
createInstance(Bee).keeper.hasMask;   // 类型精准推断
```

这种模式是 mixins 设计模式的基础。

## 内置工具类型：泛型的标准"类型函数"

TypeScript 的标准库提供了一系列基于泛型的**工具类型**，每种工具类型都是一种**类型变换操作**：

### 属性变换

| 工具类型 | 作用 | 本质 |
|---------|------|------|
| `Partial<T>` | T 的所有属性变为可选 | `{ [K in keyof T]?: T[K] }` |
| `Required<T>` | T 的所有属性变为必填 | `{ [K in keyof T]-?: T[K] }` |
| `Readonly<T>` | T 的所有属性变为只读 | `{ readonly [K in keyof T]: T[K] }` |

```typescript
interface Todo { title: string; description: string; }

function updateTodo(todo: Todo, fields: Partial<Todo>) {
  return { ...todo, ...fields };
}

// Readonly 在 freeze 场景中的典型用法
function freeze<T>(obj: T): Readonly<T>;
```

### 属性选取与排除

| 工具类型 | 作用 |
|---------|------|
| `Pick<T, K>` | 从 T 中选取键 K 构成新类型 |
| `Omit<T, K>` | 从 T 中排除键 K 构成新类型 |
| `Record<K, V>` | 用 K 作为键、V 作为值构造对象类型 |

```typescript
interface Todo { title: string; description: string; completed: boolean; }

type TodoPreview = Pick<Todo, "title" | "completed">;
// { title: string; completed: boolean }

type TodoInfo = Omit<Todo, "completed">;
// { title: string; description: string }

type CatName = "miffy" | "boris" | "mordred";
const cats: Record<CatName, CatInfo> = {
  miffy:  { age: 10, breed: "Persian" },
  boris:  { age: 5,  breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
```

### 联合类型操作

| 工具类型 | 作用 |
|---------|------|
| `Exclude<T, U>` | 从联合类型 T 中排除可分配给 U 的成员 |
| `Extract<T, U>` | 从联合类型 T 中提取可分配给 U 的成员 |
| `NonNullable<T>` | 从 T 中排除 `null` 和 `undefined` |

```typescript
type T0 = Exclude<"a" | "b" | "c", "a">;       // "b" | "c"
type T1 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T2 = NonNullable<string | null | undefined>; // string
```

`Exclude` 和 `Extract` 基于**条件类型**（`T extends U ? never : T` 和 `T extends U ? T : never`），展示了泛型如何与条件类型结合产生强大的类型级计算。

### 函数类型提取

| 工具类型 | 作用 |
|---------|------|
| `ReturnType<T>` | 提取函数类型的返回值类型 |
| `Parameters<T>` | 提取函数类型的参数类型（元组） |
| `ConstructorParameters<T>` | 提取构造函数的参数类型 |
| `InstanceType<T>` | 提取构造函数的实例类型 |

```typescript
declare function f1(arg: { a: number; b: string }): void;

type T0 = ReturnType<() => string>;        // string
type T3 = Parameters<typeof f1>;           // [{ a: number; b: string }]
```

这些工具类型依赖于 `infer` 关键字（在条件类型中进行类型推断），是 TypeScript 类型级编程的核心构建块。

### 其他重要工具类型

| 工具类型 | 作用 |
|---------|------|
| `Awaited<T>` | 递归解包 Promise（模拟 `await`） |
| `NoInfer<T>` | 阻止 T 被用于类型推断（TS 5.4+） |
| `ThisParameterType<T>` | 提取函数的 `this` 参数类型 |
| `OmitThisParameter<T>` | 从函数类型中移除 `this` 参数 |
| `ThisType<T>` | 标记上下文中的 `this` 类型（`noImplicitThis` 需开启） |

### 内在字符串转换类型

这些是编译器内置的类型，不依赖映射类型实现：

```typescript
type Greeting = "hello world";

type ShoutyGreeting = Uppercase<Greeting>;      // "HELLO WORLD"
type QuietGreeting = Lowercase<Greeting>;        // "hello world"
type CapitalizedGreeting = Capitalize<Greeting>; // "Hello world"
type UncapitalizedGreeting = Uncapitalize<"HELLO">; // "hELLO"
```

它们常与**模板字面量类型**结合使用，构建字符串级别的类型变换。

## 核心洞察

### 1. 泛型是类型层面的函数

函数操作值，泛型操作类型。`function identity(v: Value): Value` 和 `type Generic<T> = ...` 在结构上完全对应——只是参数从"值"换成了"类型"。TS 的内置工具类型本质上就是一个"类型标准库"：

```
值层面：  identity("hello")         →  "hello"
类型层面：Pick<Todo, "title">       →  { title: string }
```

### 2. 结构化类型系统使泛型天然灵活

TS 的结构化子类型意味着：只要形状匹配，类型关系自动成立。这消除了名义类型系统中泛型协变/逆变的复杂性。

```typescript
// TS 中——自然工作
const cats: Producer<Cat> = ...;
const animals: Producer<Animal> = cats;  // OK——Cat 是 Animal 的子类型

// 名义类型系统（如 Java）中——需要显式声明泛型通配符 Producer<? extends Animal>
```

TS 的泛型**自动推断**方差（协变/逆变），极少数情况才需要手动标注 `in` / `out`。

### 3. 类型推断消除冗余

大多数场景下，TS 能从值推导出类型参数，你不需要写 `<string>`。这让你在使用泛型 API 时几乎感觉不到泛型的存在：

```typescript
const nums = [1, 2, 3];
const doubled = nums.map(n => n * 2);  // map<T, U> 的类型参数被自动推断
```

只有当编译器无法推导（如无参数函数返回泛型值）时，才需要显式指定类型参数。

## 跨领域连接

### ↔ Rust：语法相近，约束机制不同

Rust 的泛型语法与 TS 几乎一致：

```rust
// Rust
fn foo<T: Trait>(x: T) -> T { x }
```

```typescript
// TypeScript
function foo<T extends Interface>(x: T): T { return x; }
```

但底层机制截然不同：
- **Rust**：`T: Trait` 是**名义约束**——类型必须显式实现（`impl`）该 trait，编译器做单态化（为每种具体类型生成专用代码）
- **TypeScript**：`T extends Interface` 是**结构约束**——只要形状匹配即可，编译后被擦除（运行时无泛型）

Rust 泛型编译时单态化 → 零运行时开销 → 每种具体类型独立编译 → 二进制膨胀。TS 泛型在编译后被**擦除** → 运行时无任何泛型信息 → 始终保持一份 JS 代码。

> **深层共鸣**：两者都是在编译期提供类型安全而不牺牲性能。Rust 通过单态化实现零成本抽象，TS 通过擦除实现零运行时负担——两种策略服务于不同目标（系统安全 vs JS 兼容），但在"编译期类型安全 + 运行时高效"这一点上殊途同归。

### → Cocos Creator：泛型的日常实践

Cocos 中最常见的泛型用法是 `getComponent`：

```typescript
const sprite = node.getComponent(Sprite);
// sprite 类型自动推断为 Sprite | null——无需类型断言

const rigidBody = node.getComponent(RigidBody2D);
// rigidBody 类型自动推断为 RigidBody2D | null
```

这依赖 `getComponent<T extends Component>(type: Constructor<T>): T | null` 的泛型签名。写一句 `getComponent` 就完成了类型窄化——这在没有泛型的 JS 中需要手动 `as` 断言。Cocos 脚本开发大量使用这种模式：[场景资源]、节点属性访问、事件系统等。

### ↔ 软件工程：参数多态的工程价值

泛型实现的是**参数多态**（parametric polymorphism）——函数式编程的基石之一。在工程层面：
- 减少代码重复：一套逻辑，多种类型（如 `Array<T>` 不需要为每种元素类型写一套实现）
- 编译期安全：类型错误从运行时崩溃前移到编译期捕获
- 工具类型作为设计模式：`Partial<T>`、`Pick<T, K>` 等工具类型本质上是被提炼出来的"类型级设计模式"——它们在类型层面复现了常见的对象操作模式

### ← JavaScript：为已有的动态模式添加类型安全

JS 中的许多模式天然就是"泛型"的——只是没有类型标注：

```javascript
// JS：map 可以操作任何元素类型的数组
[1, 2, 3].map(n => n * 2);         // number → number
["a", "b"].map(s => s.toUpper());  // string → string

// 这个动态行为的类型签名是：
// map<U>(callback: (value: T) => U): U[]
```

TS 的泛型**没有改变**这些 JS 模式的行为——它们只是把运行时的隐性约束提升为编译期的显式类型。泛型让动态语言的灵活性在静态类型系统中得到保留和验证。

## 相关页面

- [[TypeScript 概述]] — TS 类型系统的全景导航
- [[类型窄化]] — Narrowing 与泛型是 TS 类型推理的两大支柱，一个在控制流层面收窄类型，一个在类型参数层面保持类型关联
- [[结构化类型与类型兼容性]] (planning) — 理解 TS 泛型为何比 Java/C# 泛型更灵活的关键前提
- [[Rust 概述]] — Rust 的 trait 约束 vs TS 的结构约束：两种泛型哲学的对比
- [[Trait 系统]] (planning) — Rust 泛型约束的核心机制
- [[所有权与借用]] — Rust 泛型与生命周期的交织（`fn longest<'a, T>(x: &'a T, y: &'a T) -> &'a T`）
- [[软件工程概述]] — 泛型作为通用软件工程概念：参数多态、类型安全、设计模式

## 原始来源

- [TypeScript Handbook v2 — Generics](raw/TypeScript-Website/packages/documentation/copy/en/handbook-v2/Type Manipulation/Generics.md)
- [TypeScript Reference — Utility Types](raw/TypeScript-Website/packages/documentation/copy/en/reference/Utility Types.md)
