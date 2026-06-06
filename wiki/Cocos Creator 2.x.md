---
title: Cocos Creator 2.x
date: 2026-06-02
tags:
  - type/overview
  - layer/cocos
  - status/stable
aliases:
  - Cocos Creator 2
  - Creator 2.x
---

# Cocos Creator 2.x

> [!abstract] 摘要
> Cocos Creator 2.x (2018-2023) 是 Cocos 从纯代码引擎走向可视化编辑器的关键一代。基于 Cocos2d-x C++ 运行时 + JS 脚本绑定，它在 2D 手游领域积累了数十万计的商业项目。理解 2.x 的核心在于认识到它解决了“如何降低 2D 游戏开发门槛”的问题——通过可视化的场景编辑器、组件化脚本和属性检查器，让设计师和程序员可以并行工作，大幅缩短了迭代周期。但它的架构上限也在此：Cocos2d-x 自身的设计局限（仅 OpenGL ES 2.0、无原生 3D 管线、JS 与 C++ 的跨语言开销）注定了它无法承载现代 3D 渲染需求。

## 根本问题：为什么先有 2.x？

在 Cocos Creator 诞生之前，开发者使用 Cocos2d-x 纯代码引擎开发游戏——所有 UI、场景结构、动画都要通过 C++ 代码构建。这对不具备编程能力的设计师和策划极不友好，团队协作效率低下。Cocos Creator 2.x 首次将 Cocos2d-x 封装为 **可视化编辑器 + 组件化脚本** 的一体化工具，从根本上解决了三个问题：

1. **内容创作门槛**：设计师可以直接拖拽资源搭建场景，无需理解代码
2. **分工协作**：程序员编写可挂载的组件脚本，设计师通过属性检查器调整参数——两拨人可以并行工作
3. **快速迭代**：浏览器一键预览，手机上扫码即可真机测试

Cocos2d-x 作为底层提供了经过千万级设备验证的 C++ 渲染和跨平台能力，这让 2.x 在发布之初就具备了极高的稳定性和兼容性。

## 技术架构与局限

2.x 的核心架构是 **JS 脚本层 + Cocos2d-x C++ 引擎层** 的双层设计。JS 负责游戏逻辑，C++ 负责渲染和平台适配，两者通过 JSB（JavaScript Binding）桥接。这种设计在当时是合理的折中——脚本化开发保证了开发效率，C++ 内核保证了运行性能。

但这一架构也埋下了天花板：

- **渲染管线固定**：编辑器对所有材质使用内置 shader，开发者无法自定义渲染效果
- **3D 缺位**：引擎没有原生的 3D 渲染管线、PBR 光照和阴影系统；仅支持 2.5D 正交投影伪 3D
- **物理系统简陋**：2D 物理基于 Box2D，3D 物理通过 Cannon.js（JS 实现，性能差）
- **架构耦合**：节点上直接挂载坐标、颜色、透明度等属性，没有明确的 UI 组件分离；`cc.Class()` 闭包式组件定义与 ES6 类语法格格不入

## 向 3.x 迁移

### 策略

| 项目状态 | 建议 |
|---------|------|
| 已上线 2.x 项目 | **无需强制升级**。v2.4 作为 LTS 版本提供到 2022 年的缺陷修复 |
| 开发中的项目 | 评估项目周期。若临近上线，继续用 2.4；若尚在早期，可考虑迁移 |
| 新项目 | **一律使用 3.x**。2.x 已停止更新，没有新的平台支持 |

### 关键 API 变更

从 2.x 到 3.x 的迁移不是小幅升级，而是大量 API 的破坏性变更。核心变化包括：

- **组件类名**：去除 `Component` 后缀（如 `cc.Sprite` 而非旧版的 `cc.SpriteComponent`）
- **节点属性分离**：`node.x/y`、`node.color`、`node.opacity` 等直接属性全部移除，改为通过 `UITransform`、`Sprite`、`UIOpacity` 等组件访问
- **动作系统**：`cc.Action` 全部移除，统一为 Tween 系统
- **动态加载路径**：`resources.load('sprite')` 必须指定子资源路径 `resources.load('sprite/spriteFrame', SpriteFrame, ...)`
- **全局变量**：`CC_BUILD` → `BUILD`，`CC_EDITOR` → `EDITOR`（去掉 CC_ 前缀）
- **物理组件**：2D 物理组件普遍加 `2D` 后缀（`Collider2D`），3D 物理组件去掉 `3D` 后缀
- **分组系统**：`node.group`（字符串名）变为 `node.layer`（指数幂位掩码值）
- **settings 格式**：`settings.js` 改为 `settings.json`

### 迁移工具能力与局限

3.0 内置的 **文件 → 导入 Cocos Creator 2.x 项目** 工具可以：
- 自动导入所有资源文件并转换格式
- 将 JS 代码转换为 TS，自动添加 `@ccclass` 和 `@property` 装饰器
- 保留组件在场景中的引用

但工具**无法自动迁移**以下内容：
- 函数内部的业务逻辑代码（会被注释保留，需要手动重写）
- 材质系统和自定义 shader（需要参考[材质升级指南](raw/material-system/effect-2.x-to-3.0.md)手动迁移）
- 物理碰撞矩阵配置（需在项目设置中手动重建）
- 外部类型引用和第三方插件

> [!warning] 停止更新
> v2.x 已于 **2023 年** 停止更新。已有 v2.4 项目可继续维护，但不会获得新的平台支持、渲染改进或功能更新。所有新项目请使用 [[Cocos Creator 概述|Cocos Creator 3.x]]。

## 相关页面

- [[Cocos Creator 概述]] — 当前主力版本的全面介绍
- [[Cocos Creator 3.x]] — 3.x 大版本详解
- [[Cocos Creator 3D]] — 曾独立存在的 3D 分支（后并入 3.0）
- [[引擎架构]] — 2.x 到 3.x 的架构演变
- [[脚本系统]] — JS/TS 脚本系统差异
- [[图形渲染]] — 渲染管线的代际跳跃
- [[材质系统]] — 材质系统的完全重构

## 原始来源

- [Cocos Creator 3.0 升级指南](raw/release-notes/upgrade-guide-v3.0.md)
- [产品简介 - 版本关系说明](raw/index.md)
- [产品简介](raw/getting-started/introduction/index.md)
- [v3.0 材质升级指南](raw/material-system/effect-2.x-to-3.0.md)
