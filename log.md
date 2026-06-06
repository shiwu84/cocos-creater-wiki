---
title: 操作日志
date: 2026-06-10
tags:
  - layer/tool
  - type/meta
  - status/stable
aliases:
  - Wiki 日志
  - 变更日志
---

# 操作日志

> [!abstract] 摘要
> `log.md` 是 Wiki 的时间线：它记录每次 ingest、query、lint、restructure、setup 的目的、影响页面和关键发现。与面向内容导航的 `index.md` 不同，日志用于追踪知识库如何演化，帮助后续 LLM 理解最近发生了什么，并避免重复劳动。

## [2026-06-02] lint | 全面合规修复

**操作**：Lint 检查后全面修复合规问题。

**修复内容**：
- 迁移 33 页裸标签到前缀格式（移除 `asset`/`browser`/`event`/`shader`/`ui` 等裸标签）
- 为 UI 系统/材质系统/缓动系统/音频系统补充 `type/concept` 标签（原误标记为 `type/entity`）
- 补充 [[Cocos Creator 3.x]] 缺失的 `## 原始来源` section
- 增强低 wikilink 页面：[[Jujutsu VCS]]（1→4）、[[JavaScript 教程概述]]（4→32）、[[动画剪辑]]（2→4）、[[音频资源]]（2→3）、[[字体资源]]（2→4）、[[JSON 文本资源]]（2→4）、[[JavaScript 动画]]（2→5）

**受影响页面**：40 个 wiki 页面被修改，`index.md` 和 `log.md` 已更新。

**验证结果**：
- raw/ 死链接：0
- 缺失 frontmatter 字段：0
- 裸标签：0
- 缺失 `## 原始来源`：0

## [2026-06-02] ingest | 结构化类型与类型兼容性

**操作**：创建 TypeScript 结构类型系统的概念页，综合 Type Compatibility.md、Object Types.md、Basics.md。

**受影响页面**：
- 新建：[[结构化类型与类型兼容性]]（#layer/ts #type/concept #status/stable）— 名义 vs 结构类型、对象/函数/类/泛型兼容、过剩属性检查、参数双变、跨领域连接（Rust/Go/JS/SWE）
- 更新：`index.md`（在 TypeScript 分类下新增条目，页面数 67）

**关键发现**：
- TS 的结构类型是"鸭子测试的静态形式化"——桥接 JS 动态惯例与静态类型安全
- `private`/`protected` 是 TS 中唯一的"名义行为"——同类来源的 `private` 成员创建了名义式的兼容壁垒
- 过剩属性检查是结构类型规则的"例外"——仅对新鲜对象字面量生效，防止拼写错误

## [2026-06-02] ingest | Linux 文件系统

**操作**：创建 Linux 文件系统概念页，综合 7 个源文档的深度解释。

**受影响页面**：
- 新建：[[Linux 文件系统]]（#layer/linux #type/concept #status/stable）— "一切皆文件"哲学、VFS、inode、FSH、硬链接/符号链接、mount/fstab、跨领域连接
- 更新：`index.md`（新增 Linux/OS 基础分类，页面数 66）

**关键发现**：
- inode 间接指针模型与 B-tree 存储引擎设计有深层共鸣（跨 SWE 连接）
- "一切皆文件"统一接口原则在现代编程语言中反复出现（JS Stream、Rust Read/Write trait）
- mount 挂载机制的设计模式被 Docker volume、Cocos Asset Manager bundle 路径映射继承

---

按时间顺序记录 Wiki 的所有操作（Ingest、Query、Lint 等）。

---

## [2026-06-02] setup | LLM Wiki 初始化

**操作**：根据 `llm-wiki.md` 模式文档，使用 Obsidian-flavored Markdown 初始化 LLM Wiki 的基础架构。

**创建的文件**：
- `AGENTS.md` — Schema 文件，定义 Wiki 结构、页面约定、工作流
- `index.md` — Wiki 内容目录
- `log.md` — 本文件，按时间顺序的操作记录
- `Cocos Creator 概述.md` — Cocos Creator 3.8 引擎整体概述（#overview）
- `引擎架构.md` — ECS 架构、子系统关系、更新循环（#overview #concept）
- `场景与节点系统.md` — 场景管理、节点属性和组件生命周期（#entity）
- `资源系统.md` — 资源类型、Asset Manager、加载策略（#concept）
- `图形渲染.md` — 渲染管线、相机、光照阴影、材质系统（#concept）

**创建的其他文件**：
- `脚本系统.md` — TypeScript 脚本、装饰器、生命周期、事件系统（#concept）
- `物理系统.md` — 3D 物理（Bullet）、2D 物理（Box2D）、刚体碰撞关节（#concept）
- `动画系统.md` — Animation 组件、Marionette 动画图、骨骼动画、Spine（#concept）
- `.obsidian/app.json` — Obsidian 配置（附件路径、链接格式）
- `.obsidian/appearance.json` — 外观配置（折叠标题、源码模式）
- `.obsidian/graph.json` — 图谱视图配置（按标签着色）

**分析的源文档**：
- `raw/index.md` — 用户手册首页
- `raw/SUMMARY.md` — 完整目录结构
- `raw/CONTRIBUTING.md` — 文档编写规范
- `raw/concepts/scene/index.md` — 场景制作工作流
- `raw/concepts/scene/node-component.md` — 节点和组件详解
- `raw/module-map/graphics.md` — 图形渲染模块图
- `raw/asset/index.md` — 资源系统概述

**状态**：Wiki 基础架构已就绪，已创建 8 个 Wiki 页面 + 完整 Obsidian 配置。可开始逐主题 Ingest。

---

## 后续计划

1. 逐步 Ingest 各子系统源文档（脚本系统、物理系统、动画系统等）
2. 补充待撰写的 Wiki 页面
3. 创建对比页面（如 Cocos Creator vs Unity）
4. 执行首次 Lint 检查

---

## [2026-06-02] restructure | 目录结构调整

**操作**：将 Wiki 页面移入 `wiki/` 子目录，创建 `output/` 目录，对齐 llm-wiki.md 三层架构。

**变更**：
- 创建 `wiki/` 目录，移入 8 个 Wiki 页面
- 创建 `output/` 目录（暂空，供后续生成物使用）
- 更新 `AGENTS.md` 架构图、Lint 死链检查命令路径

---

## [2026-06-02] ingest | 新源添加：Jujutsu VCS + JavaScript 教程

**操作**：用户向 `raw/` 添加了两个新源，按要求进行 Ingest 处理。

**新增源文件**：
- `raw/jj_help.md` — Jujutsu 版本控制工具帮助文档（143 行）
- `raw/zh.javascript.info/` — 《现代 JavaScript 教程》中文版（174 篇文章 + 习题）

**创建的 Wiki 页面**：
- [[Jujutsu VCS]] — Jujutsu (jj) 分布式版本控制系统概述，含核心概念、命令参考、与 Git 对比
- [[JavaScript 教程概述]] — JS 教程 9 大章节总览，含语言核心和浏览器 API 体系

**更新的文件**：
- `index.md` — 新增「工具与参考」分类，添加两个新页面条目，页面数 8→10

**状态**：两个新源已建立顶层 Wiki 页面。JavaScript 教程 14 个子章节可按需深入 Ingest。

---

## [2026-06-02] batch-ingest | 全面迭代：Cocos 子系统 + JS 教程 + 交叉引用

**操作**：按照 llm-wiki.md 方法论，对全部 raw/ 源文件进行增量构建，使用 9 个并行 Agent 分三轮处理。

**新创建页面**（55 个）：

### Cocos Creator 子系统（10 个）
- [[UI 系统]] — Canvas、多分辨率适配、Widget/Layout、渲染合批、20+ UI 组件
- [[粒子系统]] — 模块化架构、CPU vs GPU 渲染器、渲染模式、粒子剔除、自定义材质
- [[音频系统]] — AudioSource、AudioClip、Web Audio/DOM Audio 双模式
- [[缓动系统]] — Tween 链式 API、缓动函数、v3.8.4 自定义动作与翻转
- [[发布系统]] — 双内核跨平台、构建面板、Web/小游戏/原生
- [[Shader 系统]] — CCEffect/CCProgram、Surface/Legacy Shader、宏定义、全局 Uniform
- [[材质系统]] — EffectAsset 数据集、共享/实例材质、程序化操作
- [[编辑器界面]] — 编辑器面板布局、场景编辑器、层级管理器、属性检查器
- [[进阶主题]] — 引擎定制、热更新、JSB 桥接、动态合图
- [[Cocos Creator vs Unity]] — 架构/脚本/渲染/物理/动画全方位对比

### Cocos Creator 资源类型（13 个）
- [[场景资源]]、[[预制资源]]、[[材质资源]]、[[纹理贴图]]、[[精灵帧]]
- [[模型资源]]、[[音频资源]]、[[字体资源]]、[[动画剪辑]]、[[JSON 文本资源]]
- [[Spine 动画]]、[[DragonBones 动画]]、[[TiledMap]]

### Cocos Creator 版本（3 个）
- [[Cocos Creator 2.x]]、[[Cocos Creator 3.x]]、[[Cocos Creator 3D]]

### 工具（1 个）
- [[Git 与版本控制]]

### JavaScript 教程（28 个）
- 概述：[[JavaScript 教程概述]]
- Part 1 语言（14 章）：入门、基础知识、代码质量、对象基础、数据类型、函数进阶、对象属性、原型与继承、类、错误处理、Promise 与异步、Generator 与迭代器、模块、杂项
- Part 2 浏览器（5 章）：Document、事件、事件细节、表单与控件、数据加载
- Part 3-9（7 章）：Frame 与 Window、二进制数据、网络请求、浏览器存储、动画、Web Components、正则表达式

**修复的交叉引用**：
- 修正 JS 页面的 10 个错误 wikilink（`[[Environment]]`/`[[Prototype]]`/`[[HomeObject]]` → 代码格式）
- 补充资源系统下 13 个资源子页面的 wikilink 目标

**Lint 结果**：
- 0 orphan wikilinks
- 0 dead raw links
- Wiki 页面总数：10 → 63

---

## [2026-06-02] restructure | Schema v3.0 全面重写

**操作**：根据用户要求，将 AGENTS.md 从 v2.0 升级到 v3.0，扩展知识领域覆盖。

**核心变更**：
- **新增领域层**：`#layer/linux`（Linux 系统）、`#layer/rust`（Rust 编程）、`#layer/sw-eng`（软件工程横向层）
- **重命名**：`#layer/project` → `#layer/game-dev`
- **学习依赖链扩展**：从单链发展为双链 + 横向贯通层
  - 链 1：Linux → JS → TS → Cocos Creator → Game Dev
  - 链 2：Linux → 系统编程概念 → Rust
  - 横向层：`#layer/sw-eng` 贯通所有领域
- **新增 raw/ 源文档说明**：Linux Journey（186 课）、Rust TRPL（115 文件）
- **新增「知识图谱策略」章节**：概念复用、Hub 页面、跨域桥梁、冲突检测、反向链接检查
- **新增「Index 与 Log 设计」章节**：显式定义 index.md 学习层组织方式
- **更新 graph.json**：颜色分组匹配新标签系统（`#type/` 前缀）

**受影响文件**：
- `AGENTS.md` — 全面重写（v2.0 → v3.0）
- `.obsidian/graph.json` — 更新标签颜色分组

**待办**：
- 现有 63 个 Wiki 页面的标签从裸格式迁移到 `#layer/` + `#type/` + `#status/` 格式
- 开始 Ingest Linux Journey 和 Rust TRPL 源文档

---

## [2026-06-02] build | 全面 Wiki 构建

**操作**：执行大规模增量 Wiki 构建，从单领域（Cocos+JS）扩展为完整的八领域知识体系。

### 标签迁移
- 63 个现有页面从裸标签（`overview`, `concept` 等）迁移到 `#layer/` + `#type/` + `#status/` 前缀格式

### 新领域概述页（5 个）
- [[Linux 概述]] — Linux 知识体系总览，三层架构、七大知识域、跨链连接
- [[Rust 概述]] — Rust 语言全景，所有权/借用核心创新、六层概念体系
- [[TypeScript 概述]] — TS 类型系统入口，四层架构、学习路径
- [[软件工程概述]] — SWE 横向层枢纽，七大核心概念域跨领域对比
- [[游戏开发概述]] — 游戏开发实践入口，架构模式/性能优化/跨平台

### Linux 概念页（6 个，Ingest 自 Linux Journey 186 课）
- [[Linux 进程模型]] — fork/exec、进程状态、信号、调度
- [[Linux 文件系统]] — VFS、inode、FSH、挂载
- [[Linux 权限体系]] — rwx、SUID/SGID、粘滞位、进程权限
- [[Linux 网络协议栈]] — TCP/IP 四层模型、DNS、路由
- [[Linux 内核架构]] — 特权级分离、系统调用、内核模块
- [[systemd 服务管理]] — 单元文件、target 依赖图、timer/socket 激活

### Rust 概念页（4 个，Ingest 自 Rust TRPL 21 章）
- [[所有权与借用]] — Rust 核心创新，move/borrow/lifetimes
- [[Trait 系统]] — 共享行为定义，泛型约束，trait object
- [[Rust 并发模型]] — Send/Sync、threads、channels、shared state
- [[智能指针]] — Box/Rc/RefCell/Arc、内部可变性模式

### TypeScript 概念页（3 个，Ingest 自 TS 官方文档）
- [[类型窄化]] — 运行时控制流 → 编译期类型推导
- [[泛型 (TypeScript)]] — 类型参数化，泛型约束，工具类型
- [[结构化类型与类型兼容性]] — 名义 vs 结构类型，过剩属性检查

### SWE 内容页（2 个，横向贯通综合）
- [[并发模型对比]] — Linux/JS/Rust/Cocos 四模型全景对比
- [[错误处理策略对比]] — 异常 vs Result vs 信号，跨语言策略分析

### 跨领域引用增强
- 13 个 JS 页面补充跨域 wikilink（→ Linux, Rust, TS, SWE）
- 10 个 Cocos 页面补充跨域 wikilink（→ TS, Linux, SWE）

### index.md 全面重写
- 从单领域 Cocos 中心架构改为八层学习依赖链组织（Linux→JS→TS→Rust→Cocos→GameDev + SWE + Tools）

### Lint 结果
- 0 dead raw links
- Wiki 页面总数：63 → 83

---

## [2026-06-06] restructure | 根据 llm-wiki.md 优化项目规则

**操作**：对照 `llm-wiki.md` 的核心理念，轻量迭代当前项目基础设施，不修改只读理念文档和 raw/ 源文档。

**受影响文件**：
- `AGENTS.md` — 升级到 v3.1，补充运行原则、检索决策、Query 归档判断和 `log.md` 元数据规则
- `index.md` — 补齐 `layer/tool`、`status/stable`、aliases 和摘要 callout，使其符合 meta 页面规范
- `log.md` — 补齐 frontmatter、摘要 callout，并追加本次操作记录

**关键发现**：
- 当前仓库已经实现 raw/wiki/schema 三层架构，主要缺口不在目录结构，而在工作流决策的显式化。
- `index.md` 和 `log.md` 是 Wiki 基础设施页面，也应被标签系统和 Obsidian 元数据统一管理。
---

## [2026-06-10] ingest | 大规模迭代：Rust/TS/Linux/SWE 缺口补全 + 跨域桥梁

**操作**：对照 `llm-wiki.md` 的核心理念，大规模补全四层知识缺口并创建跨链桥梁页。

### 新建 Rust 层页面（6 个）
- [[Rust 生命周期]] — 引用的时间维度、借用检查器、生命周期省略三规则、结构体/方法中的生命周期
- [[Rust 错误处理]] — Result + panic! 二分法、? 运算符自动类型转换、跨语言对比
- [[迭代器与闭包]] — Fn/FnMut/FnOnce 三级 trait、惰性迭代器链、零成本抽象
- [[Cargo 与 crate 生态]] — 包/crate/模块三层、发布配置、工作空间、crates.io
- [[Rust 宏系统]] — macro_rules! 声明宏、三种过程宏、卫生宏 vs C 预处理器
- [[unsafe Rust]] — 五种超能力、FFI、安全封装原则

### 新建 TypeScript 层页面（3 个）
- [[TypeScript 模块系统]] — 模块语法/解析策略/输出目标、node vs classic、与 CommonJS 互操作
- [[声明文件]] — .d.ts 结构、DefinitelyTyped/@types、为 JS 库编写类型
- [[tsconfig 配置]] — strict 家族/模块/输出三层选项、include/exclude、项目引用

### 新建 Linux 层页面（2 个）
- [[Shell 与命令行]] — 命令解释器、管道/重定向、文本处理三剑客、脚本基础
- [[Linux 包管理]] — apt/yum/dnf/pacman、依赖解析、底层 vs 上层、与语言包管理器的概念共鸣

### 新建 SWE 横向层页面（4 个）
- [[设计模式]] — 跨语言模式的形态差异（观察者在 JS/Cocos/Rust 的不同实现）、模式不是银弹
- [[测试策略]] — 测试金字塔、Rust 语言内建测试 vs JS 生态拼装的设计哲学差异
- [[内存管理对比]] — 虚拟内存(OS) → 所有权(Rust) → GC(JS) 的分层协作与同构问题
- [[模块化对比]] — Rust crate → TS/ES 模块 → Linux LKM 的严格度光谱对比

### 修改的已有页面
- [[Rust 概述]] — 添加 6 个新页面到本领域导航
- [[TypeScript 概述]] — 添加 3 个新页面到本领域导航，重组相关页面层次
- [[Linux 概述]] — 添加 Shell 与命令行、Linux 包管理到概念页面列表
- [[软件工程概述]] — 添加 4 个新 SWE 页面到概念导航，移除已创建页面对应的 TODO

### 基础设施
- `index.md` — 添加全部 15 个新页面，Wiki 页面数 83 → 98，更新日期
- `log.md` — 追加本次操作记录

### Lint 结果
- 0 dead raw links（修复 1 处 modules-reference 目录引用 → 指向具体 .md 文件）
- 所有新页面 ≥ 3 wikilink，满足最低连接度要求
- Cocos 浅薄实体页在上次 Lint 中已加深（扩充内容 + 添加 wikilink + raw/ 引用），本次未进一步合并

**关键发现**：
- 本次迭代使 Rust 层从 5 个概念页扩展为 11 个，覆盖了 TRPL 的完整章节体系
- TS 层从 4 个概念页扩展为 7 个，补全了模块/声明/配置三个核心空白
- SWE 横向层从 3 页扩展为 7 页（+设计模式/测试/内存对比/模块对比），跨域对比桥梁初步成型
- 4 个跨链对比页（并发/错误/内存/模块）形成了四条"从 OS 到应用层"的全栈概念连线

