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
> Cocos Creator 2.x 是上一代主力版本，基于 Cocos2d-x 运行时，已于 2023 年停止更新。所有功能已由 [[Cocos Creator 概述|Cocos Creator 3.x]] 继承并重构。

## 与 3.x 的主要差异

| 特性 | 2.x | 3.x |
|------|-----|-----|
| 编程语言 | JavaScript（推荐）+ TypeScript | **TypeScript**（主力） |
| 运行时 | Cocos2d-x (C++) | 全新自研跨平台内核 |
| 3D 支持 | 仅限 2.5D 正交场景 | **完整 3D 管线**（PBR、阴影等） |
| 渲染后端 | OpenGL ES 2.0 | Vulkan / Metal / WebGL / WebGPU |
| 物理引擎 | Box2D + Cannon.js | Bullet (3D) + Box2D (2D) |
| 组件架构 | `cc.Component` 旧式 API | ECS 架构，`Component` + `system` |
| 材质系统 | 内置固定 shader | 可编程 Effect Asset |
| 构建管道 | 旧版构建面板 | 可扩展构建插件系统 |

## 迁移

Cocos Creator 3.0 提供了 v2.x 资源导入工具（**文件 -> 导入 Cocos Creator 2.x 项目**），支持资源导入和代码辅助迁移（JS → TS 转换）。

> [!warning]
> v2.x 已于 2023 年停止更新。新项目应使用 [[Cocos Creator 概述|3.x]] 版本。已有 2.4 项目可继续维护，但不再有新功能。

## 相关页面

- [[Cocos Creator 概述]]
- [[引擎架构]]
- [[脚本系统]]
- [[图形渲染]]

## 原始来源

- [Cocos Creator 3.0 升级指南](raw/release-notes/upgrade-guide-v3.0.md)
- [v3.0 材质升级指南](raw/material-system/effect-2.x-to-3.0.md)
