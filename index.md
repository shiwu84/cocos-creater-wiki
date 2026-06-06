---
title: Wiki 内容目录
date: 2026-06-10
tags:
  - layer/tool
  - type/meta
  - status/stable
aliases:
  - 内容目录
  - Wiki 索引
---

# Wiki 内容目录

Wiki 目录 — 所有 LLM 生成的 Wiki 页面索引，按学习层由底向上组织。

> [!abstract] 摘要
> `index.md` 是 Wiki 的内容地图：它不是知识正文，而是帮助 LLM 和读者按学习依赖链定位页面、发现领域入口、确认页面类别的导航层。Query 时先读目录再进入具体页面；Ingest 后必须更新目录，保证持久 Wiki 不退化为散落文件集合。

> 本文件由 LLM 自动维护，每次 Ingest 操作后更新。

## Linux / OS 基础

`#layer/linux` — 系统底层、进程管理、文件系统、权限模型、网络栈

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[Linux 概述]] | Linux 知识体系总览——三层架构、七大知识域、跨领域连接，是整个 Wiki 的地基层入口 | 概述 |
| [[Linux 文件系统]] | "一切皆文件"哲学、VFS 抽象层、inode 模型、FSH 目录标准、硬链接/符号链接、mount 挂载机制 | 概念 |
| [[Linux 内核架构]] | 特权级别分离（Ring 0/3）、宏内核+模块化、系统调用表、内核内存不可换出——操作系统的最底层抽象 | 概念 |
| [[Linux 进程模型]] | fork+exec 创建、五态调度（R/S/D/Z/T）、信号通信——"一块 CPU 如何同时运行数百个程序"的完整答案 | 概念 |
| [[Linux 权限体系]] | DAC 自主访问控制、rwx 三元位、SUID/SGID/粘滞位、三重 UID 机制、passwd/shadow/group 三文件体系 | 概念 |
| [[Linux 网络协议栈]] | TCP/IP 四层模型、socket 接口、sk_buff 数据包传递、封装→传输→解封装全路径 | 概念 |
| [[systemd 服务管理]] | 现代 Linux 默认 init 系统、单元驱动+依赖解析+并行启动、服务编排/日志/定时器/设备管理工具集 | 概念 |
| [[Shell 与命令行]] | 命令解释器、管道与重定向、文本处理（grep/sed/awk）、环境变量、Shell 脚本基础 | 概念 |
| [[Linux 包管理]] | apt/yum/dnf/pacman、依赖解析、底层 vs 上层工具、编译安装 vs 仓库分发——与 Cargo/npm 的概念共鸣 | 概念 |

## JavaScript 语言

`#layer/js` — JS 语言核心（运行时 + 浏览器 API）

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[JavaScript 教程概述]] | 《现代 JavaScript 教程》中文版总览（174 篇文章），JS 语言 + 浏览器 API 完整体系 | 概述 |
| [[JavaScript 入门]] | 语言特性、浏览器能力与安全限制、JS 引擎（V8/SpiderMonkey）、开发环境 | 概念 |
| [[JavaScript 基础知识]] | 变量声明（let/const/var）、八种数据类型、运算符、条件与循环、函数基础、箭头函数 | 概念 |
| [[JavaScript 代码质量]] | 浏览器调试、编码风格、BDD 测试（Mocha/Chai）、Polyfill 与转译器 | 概念 |
| [[JavaScript 对象基础]] | 对象创建与属性、引用与拷贝、this 绑定、构造器、可选链、Symbol、类型转换 | 概念 |
| [[JavaScript 数据类型]] | 数字/Math、字符串、数组原型方法、Map/Set/WeakMap/WeakSet、解构、Date、JSON | 概念 |
| [[JavaScript 函数进阶]] | 递归、Rest/Spread、闭包与词法环境、var 提升、setTimeout/setInterval、call/apply/bind、装饰器 | 概念 |
| [[JavaScript 对象属性]] | 属性标志与描述符、Object 全局方法（freeze/seal）、访问器属性 getter/setter | 概念 |
| [[JavaScript 原型与继承]] | Prototype 链、F.prototype、原生原型、现代原型操作方法 | 概念 |
| [[JavaScript 类]] | class 语法糖、extends/super、静态成员、私有字段 #、instanceof、Mixin 模式 | 概念 |
| [[JavaScript 错误处理]] | try...catch...finally、throw、Error 层次体系、包装异常模式 | 概念 |
| [[JavaScript Promise 与异步]] | 回调→Promise 链→async/await、Promise API 六大静态方法、微任务队列 | 概念 |
| [[JavaScript Generator 与迭代器]] | function* yield 双向通信、Symbol.asyncIterator、async generator 分页数据应用 | 概念 |
| [[JavaScript 模块]] | ES6 export/import 语法体系、命名/默认导出、模块作用域、动态 import() | 概念 |
| [[JavaScript 杂项]] | Proxy 拦截、eval、柯里化、Reference Type、BigInt、Unicode 编码 | 概念 |
| [[JavaScript Document]] | DOM 树结构、节点搜索与遍历、特性/属性、文档修改、样式操作、尺寸与坐标系统 | 概念 |
| [[JavaScript 事件]] | 事件处理程序分配、冒泡/捕获传播、事件委托、阻止默认行为、自定义事件 | 概念 |
| [[JavaScript 事件细节]] | 鼠标事件类型与坐标、键盘 event.key/code、移动事件与拖拽、滚动事件优化 | 概念 |
| [[JavaScript 表单与控件]] | 表单导航与属性、focus/blur 焦点管理、change/input 事件、提交与验证 | 概念 |
| [[JavaScript 数据加载]] | 页面生命周期事件、async vs defer 脚本加载、动态资源 onload/onerror | 概念 |
| [[JavaScript Frame 与 Window]] | 弹窗操作、同源策略、跨窗口通信（postMessage）、iframe、点击劫持防御 | 概念 |
| [[JavaScript 二进制数据]] | ArrayBuffer/TypedArray/DataView、Blob/File、TextDecoder/Encoder 编解码 | 概念 |
| [[JavaScript 网络请求]] | fetch API、WebSocket 实时通信、FormData、请求控制（中止/进度/跨域） | 概念 |
| [[JavaScript 浏览器存储]] | Cookie、localStorage/sessionStorage、IndexedDB 事务性客户端数据库 | 概念 |
| [[JavaScript 动画]] | 贝塞尔曲线、CSS 过渡/动画、requestAnimationFrame、自定义时序函数 | 概念 |
| [[JavaScript Web Components]] | Custom Elements、Shadow DOM 封装、Template 元素、Slot 插槽 | 概念 |
| [[JavaScript 正则表达式]] | 修饰符、字符类/量词/锚点/分组/环视断言、match/matchAll/replace 方法体系 | 概念 |

## TypeScript 类型系统

`#layer/ts` — 静态类型系统、泛型、类型体操

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[TypeScript 概述]] | TypeScript 静态类型系统总览：从类型基础到进阶类型编程，与 JS/Cocos/Rust/SWE 的跨领域连接 | 概述 |
| [[泛型 (TypeScript)]] | 类型参数化——让函数/接口/类操作类型变量，工具类型（Partial/Readonly/Record）是内置的"类型级函数" | 概念 |
| [[类型窄化]] | 编译器通过控制流分析（typeof/instanceof/真值检查）自动将联合类型收窄为精确类型——类型系统适应代码 | 概念 |
| [[结构化类型与类型兼容性]] | 名义 vs 结构类型、对象/函数/类/泛型兼容规则、过剩属性检查、跨语言对比（Rust/Go） | 概念 |
| [[TypeScript 模块系统]] | 模块语法、node/classic 解析策略、模块输出目标、与 CommonJS 互操作、命名空间历史 | 概念 |
| [[声明文件]] | .d.ts 结构、DefinitelyTyped/@types、为 JS 库编写类型声明、declare 语法、模块扩充 | 概念 |
| [[tsconfig 配置]] | strict 家族/模块/输出三层选项体系、include/exclude、项目引用、tsconfig 继承 | 概念 |

## Rust 程序设计

`#layer/rust` — 系统编程、编译期安全、所有权模型

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[Rust 概述]] | 编译期内存安全无 GC、所有权+借用+生命周期三大规则、trait 零成本多态——C/C++ 的现代替代 | 概述 |
| [[所有权与借用]] | Rust 内存安全的根本方案——每个值有且仅有一个所有者，引用（&T/&mut T）由编译器验证安全，消除悬垂/竞争/双重释放 | 概念 |
| [[Trait 系统]] | Rust 无继承的全部多态机制——trait 定义共享行为、孤儿规则/blanket impl、单态化静态分发 vs dyn trait 动态分发 | 概念 |
| [[智能指针]] | Box（堆分配）、Rc/Arc（共享所有权）、RefCell（内部可变性）——拓展所有权规则边界的高阶抽象，Deref + Drop 共性基础 | 概念 |
| [[Rust 并发模型]] | 编译期消除数据竞争——Send/Sync trait 标记 + 三种范式：线程+move、消息传递、共享状态（Arc<Mutex<T>>） | 概念 |
| [[Rust 生命周期]] | 引用的时间维度——借用检查器、生命周期省略三规则、'static、结构体/方法中的生命周期注解 | 概念 |
| [[Rust 错误处理]] | 无异常：Result<T,E> + panic! 二分法、? 运算符自动错误类型转换、Result 组合方法 | 概念 |
| [[迭代器与闭包]] | 零成本函数式抽象——Fn/FnMut/FnOnce trait 三级、惰性迭代器链、消费适配器 vs 迭代器适配器 | 概念 |
| [[Cargo 与 crate 生态]] | 构建系统与包管理器——包/crate/模块三层、发布配置、工作空间、crates.io 生态 | 概念 |
| [[Rust 宏系统]] | 编译期元编程——macro_rules! 声明宏、自定义 derive/属性/类函数三种过程宏、与函数的取舍 | 概念 |
| [[unsafe Rust]] | 五种超能力——裸指针、FFI、静态变量、unsafe trait、union；安全封装原则 | 概念 |

## Cocos Creator 引擎

`#layer/cocos` — 游戏引擎架构、核心系统、资源管理

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[Cocos Creator 概述]] | Cocos Creator 3.8 引擎的整体介绍、版本演进、核心特性、系统架构总览 | 概述 |
| [[引擎架构]] | ECS 架构详解、子系统关系、更新循环、与 Unity 的对比 | 概述 |
| [[Cocos Creator 2.x]] | Cocos Creator 2.x 与 3.x 的主要差异对比 | 概述 |
| [[Cocos Creator 3.x]] | Cocos Creator 3.x 统一 2D/3D 开发的新架构 | 概述 |
| [[Cocos Creator 3D]] | 已被 3.x 统一取代的独立版本说明 | 概述 |
| [[图形渲染]] | 渲染管线、相机、光照阴影、材质、辅助渲染特性 | 概念 |
| [[脚本系统]] | TypeScript 脚本、装饰器、生命周期、事件系统、资源加载 | 概念 |
| [[物理系统]] | 3D 物理（Bullet）、2D 物理（Box2D）、刚体、碰撞体、关节 | 概念 |
| [[动画系统]] | Animation 组件、动画剪辑、骨骼动画、Marionette 动画图、程序化动画 | 概念 |
| [[发布系统]] | 双内核跨平台发布、构建面板与选项、Web/小游戏/原生多平台、命令行自动化构建 | 概念 |
| [[Shader 系统]] | Cocos Shader（YAML+GLSL）、CCEffect/CCProgram 结构、Surface Shader、宏定义、内置 Uniform | 概念 |
| [[资源系统]] | 资源类型、导入工作流、Asset Manager、加载方式、分包策略 | 概念 |
| [[编辑器界面]] | 编辑器面板布局、场景编辑器、层级管理器、资源管理器、属性检查器、偏好设置、项目设置 | 概念 |
| [[进阶主题]] | 引擎定制、热更新、数据存储、多语言国际化、网络通信、JSB 原生桥接、动态合图 | 概念 |
| [[场景与节点系统]] | 场景管理、节点树、组件挂载、坐标系、渲染顺序 | 实体 |
| [[音频系统]] | AudioSource 组件、AudioClip 资源、Web Audio / DOM Audio 双模式、playOneShot 音效 | 实体 |
| [[缓动系统]] | Tween 链式 API、内置缓动函数、队列/并行/重复控制、v3.8.4 自定义动作与翻转 | 实体 |
| [[材质系统]] | 材质作为着色器数据集、EffectAsset 关联、共享材质与材质实例、程序化材质操作 | 实体 |
| [[UI 系统]] | Canvas 画布、多分辨率适配、Widget/Layout 布局、渲染合批、20+ UI 组件 | 实体 |
| [[粒子系统]] | 3D/2D 粒子、模块化架构（主模块、发射器、9 个动画子模块、渲染器）、CPU vs GPU 渲染 | 实体 |
| [[场景资源]] | 场景文件（.scene）的结构与属性 | 实体 |
| [[预制资源]] | 预制体（Prefab）的创建、编辑与实例化 | 实体 |
| [[材质资源]] | 材质的资源文件格式与属性面板 | 实体 |
| [[纹理贴图]] | 图片资源与纹理的导入格式、过滤模式、压缩纹理 | 实体 |
| [[精灵帧]] | 精灵帧资源与图集管理 | 实体 |
| [[模型资源]] | 3D 模型（FBX/glTF）的导入、Mesh 与材质提取 | 实体 |
| [[动画剪辑]] | 动画剪辑资源的创建、编辑与曲线控制 | 实体 |
| [[音频资源]] | 音频剪辑的格式兼容性与加载模式 | 实体 |
| [[字体资源]] | 位图字体、TTF 字体与 LabelAtlas 资源 | 实体 |
| [[JSON 文本资源]] | JSON 和文本资源的加载与解析 | 实体 |
| [[Spine 动画]] | Spine 骨骼动画资源的导入与使用 | 实体 |
| [[DragonBones 动画]] | DragonBones 动画资源的导入与使用 | 实体 |
| [[TiledMap]] | Tiled 地图编辑器导出的 TMX 地图资源 | 实体 |
| [[Cocos Creator vs Unity]] | 架构设计、脚本语言、编辑器工作流、渲染管线、物理系统、动画系统的全方位对比 | 对比 |

## 游戏开发实践

`#layer/game-dev` — 实践方案、项目经验、性能优化

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[游戏开发概述]] | 游戏开发实践层入口——架构模式、性能优化、编辑器扩展、跨平台适配、项目工作流 | 概述 |

## 软件工程

`#layer/sw-eng` — 横向贯通层：设计模式、测试、CI/CD、架构

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[软件工程概述]] | 软件工程总览——设计模式、测试策略、CI/CD、架构设计，跨所有领域层的横向贯通概念 | 概述 |
| [[并发模型对比]] | Linux 进程隔离 / JS 事件循环 / Rust Send+Sync / Cocos 帧同步——四种并发模型的调度、隔离、通信三轴对比 | 对比 |
| [[错误处理策略对比]] | 异常 vs 错误值——Linux 信号、JS try-catch/Promise、Rust Result/panic、Cocos 混合模型的哲学分歧与组合性对比 | 对比 |
| [[内存管理对比]] | 虚拟内存(OS) → 所有权(Rust) → GC(JS)——三个层次的分层协作与同构问题 | 对比 |
| [[模块化对比]] | Rust crate → TS/ES 模块 → Linux LKM——从默认私有到内核态模块的严格度光谱 | 对比 |
| [[设计模式]] | 跨语言模式的形态差异——观察者在 JS/Cocos/Rust 中因语言范式而表现不同 | 概念 |
| [[测试策略]] | 测试金字塔、Rust 语言内建测试 vs JS 生态拼装（Mocha/Jest）的设计哲学差异 | 概念 |

## 工具与参考

`#layer/tool` — 开发工具、版本控制、Shell

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[Jujutsu VCS]] | Jujutsu (jj) 实验性分布式版本控制系统，兼容 Git，操作日志、自动快照 | 概念 |
| [[Git 与版本控制]] | Git 基础工作流和与 Jujutsu 的对比 | 概念 |

---

*最后更新：2026-06-10 • Wiki 页面数：98*
