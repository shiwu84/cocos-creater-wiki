---
title: Cocos Creator 3.x
date: 2026-06-02
tags:
  - type/overview
  - layer/cocos
  - status/stable
aliases: []
---

# Cocos Creator 3.x

> [!abstract] 摘要
> Cocos Creator 3.x (2021-) 代表了 Cocos 引擎从 2D 时代到泛移动端 3D 时代的根本性转折。它彻底抛弃了 Cocos2d-x C++ 运行时，代之以全新自研的跨平台 3D 渲染内核，并统一了原本分立的 2D 和 3D 两条产品线。对开发者而言，3.x 不仅仅是一个大版本升级——它意味着：TypeScript 成为第一语言、ECS 架构替换旧的 `cc.Class()` 模式、可编程材质系统取代硬编码 shader、以及从 Vulkan 到 WebGPU 的多后端现代图形 API 支持。当前最新稳定版为 3.8 LTS。

## 根本问题：为什么需要 3.x？

2.x 的成功背后是 Cocos2d-x 的架构债。到 2020 年前后，几个根本问题变得无法回避：

1. **3D 是不可逆的趋势**：手机 GPU 性能快速增长，重度 3D 手游成为市场主流，而 2.x 完全没有原生 3D 管线
2. **图形 API 换代**：Vulkan 和 Metal 取代 OpenGL ES 成为移动端主流底层 API，继续绑定 OpenGL ES 2.0 等于放弃性能优势
3. **架构天花板**：JS 与 C++ 的 JSB 桥接带来调试困难和性能开销；`cc.Class()` 闭包式组件定义与现代 ES6+ TypeScript 生态格格不入；固定 shader 管线无法满足美术的自定义材质需求
4. **产品线分裂**：2D (Creator 2.x) 和 3D (Creator 3D) 两条产品线各自发展，代码库独立，团队无法在同一个项目中同时使用 2D 和 3D 能力

3.0 的回答是：**完全重写**。不是修补 2.x，而是新建一个面向未来十年的跨平台 3D 引擎内核，然后把 2D 功能以同一套架构重新实现进去。

## 从 2.x 到 3.x：代际跳跃

### 架构层面的根本变化

| 维度 | 2.x | 3.x | 意义 |
|------|-----|-----|------|
| 内核语言 | C++ (Cocos2d-x) | TypeScript + C++ (新内核) | C++ 仍负责底层渲染，但 JSB 桥接层被彻底重构 |
| 编程语言 | JS 为主，TS 可选 | **TS 强制，开启严格模式** | 类型安全覆盖整个代码库 |
| 组件系统 | `cc.Class()` 闭包定义 | `@ccclass` 装饰器 + ES6 class | 与标准 TypeScript 生态完全兼容 |
| 渲染器 | 固定管线，单后端 | **可编程管线**，多后端（Vulkan/Metal/WebGL/WebGPU） | 支持自定义材质、自定义渲染管线 |
| 3D 能力 | 无 | 完整 PBR、阴影、后处理、骨骼动画 | 从引擎层面支持现代 3D |
| 节点属性 | 直接挂在 node 上 | 通过组件访问（UITransform, Sprite 等） | 关注点分离，架构更清晰 |
| 材质 | 固定内置 shader | **Effect Asset** 可编程材质 | 美术和图形程序完全控制渲染效果 |
| 物理 | Box2D + Cannon.js | PhysX/Bullet (3D) + Box2D (2D) | 工业级 3D 物理 |
| 构建 | 固定构建面板 | 插件化构建系统 | 可扩展、可自定义 |
| 编辑器插件 | 功能有限 | 完整插件生态 | 编辑器二次开发 |

### 关键的设计哲学转向

**从"引擎即运行时"到"引擎即平台"**。2.x 时代，Cocos Creator 本质上是 Cocos2d-x 的编辑器外壳。3.x 时代，引擎成为一个自包含的创造平台——编辑器本身就是用 TypeScript + HTML 构建的插件系统，引擎内核通过面向数据的架构设计实现了多线程渲染和负载均衡。

**从"JS 便利"到"TS 严格"**。2.x 推崇 JS 的灵活性和动态调试便利，但这在大型项目中变成维护灾难。3.x 强制 TypeScript 严格模式，通过编译期检查消除整类运行时错误。

## 版本迭代路线

| 版本 | 发布时间 | 关键里程碑 |
|------|---------|-----------|
| 3.0 | 2021 年初 | 统一 2D/3D，全新内核，移除了 Action 系统和旧 API |
| 3.1 | 2021 | `priority` 弃用，改为 `setSiblingIndex`，渲染管线改进 |
| 3.8 (LTS) | 2023+ | 程序化动画、高精度文本、可定制渲染管线、角色控制器 |

## 迁移实践：从 2.x 到 3.x

### 自动迁移工具

3.0 提供 **文件 → 导入 Cocos Creator 2.x 项目**，可完成：
- 资源文件自动导入和格式转换
- JS → TS 代码转换（自动添加 `@ccclass`、`@property` 装饰器、import 语句）
- 组件场景引用保留

### 无法自动迁移的部分

迁移工具只能做语法层面的转换，以下内容必须**手动处理**：

- **业务逻辑代码**：函数体内部代码以注释形式保留，需要逐一手动重写
- **材质和 shader**：从旧的固定 shader 迁移到 Effect Asset 系统（参考[材质升级指南](raw/material-system/effect-2.x-to-3.0.md)）
- **第三方插件**：需要寻找 3.x 版本或自行改写
- **外部类型声明**：工具无法推断 JS 中使用的外部类型，需手动添加类型声明
- **物理碰撞矩阵**：需在项目设置中手动重建
- **构建配置**：settings.js 变为 settings.json，构建管道完全重构

### 常见迁移陷阱

- **忘记指定子资源路径**：`resources.load('img', SpriteFrame)` 需改为 `resources.load('img/spriteFrame', SpriteFrame)`
- **直接访问 node.x/node.color 等属性**：这些在 3.x 中不再挂载在 node 上，必须通过对应组件获取
- **大小写敏感**：`getComponent('脚本名')` 在 3.x 中必须以类名匹配，大小写敏感
- **碰撞回调需要注册**：3.x 的碰撞事件不会自动触发，需要显式注册 `collider.on(Contact2DType.BEGIN_CONTACT, ...)`
- **`audioEngine` 已移除**：统一使用 `AudioSource` 组件控制音频

## 相关页面

- [[Cocos Creator 概述]] — 3.8 LTS 版本的系统性概述
- [[Cocos Creator 2.x]] — 上一代版本的技术背景和迁移对照
- [[Cocos Creator 3D]] — 独立 3D 分支的历史（已并入 3.0）
- [[引擎架构]] — ECS 架构在 Cocos 中的实现
- [[图形渲染]] — 3.x 的全新渲染管线
- [[脚本系统]] — TypeScript 脚本与组件开发
- [[材质系统]] — Effect Asset 可编程材质
- [[物理系统]] — 3.x 的物理引擎升级
- [[Cocos Creator vs Unity]] — 与其他主流引擎的对比

## 原始来源

- [Cocos Creator 3.8 用户手册](raw/index.md)
- [产品简介](raw/getting-started/introduction/index.md)
- [Cocos Creator 3.0 升级指南](raw/release-notes/upgrade-guide-v3.0.md)
