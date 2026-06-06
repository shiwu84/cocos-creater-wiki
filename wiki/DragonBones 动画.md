---
title: DragonBones 动画
date: 2026-06-02
tags:
  - layer/cocos
  - type/entity
  - status/stable
aliases: []
---

# DragonBones 动画

> [!abstract] 摘要
> DragonBones 骨骼动画资源是开源的 2D 骨骼动画解决方案，解决与 Spine 相同的根本问题——用骨骼驱动的方式替代帧动画，实现高效、可混合、可换肤的角色动画。DragonBones 由白鹭引擎团队开发并开源，完全免费，与 Cocos Creator 深度集成。引擎支持 DragonBones v5.6.3 及以下版本导出的数据格式，包含 JSON 和二进制（dbbin）两种骨骼数据格式。

## 核心概念

### DragonBones 文件结构

DragonBones 动画资源由三部分组成：

| 文件 | 格式 | 内容 |
|------|------|------|
| **骨骼数据** | `.json` 或 `.dbbin`（二进制） | 骨架定义、动画数据、IK 约束、网格蒙皮 |
| **图集数据** | `.json` | 纹理区域定义 |
| **图集纹理** | `.png` | 角色各部位的纹理图集 |

与 Spine 的关键区别：DragonBones 的图集数据使用 `.json` 格式（Spine 使用 `.atlas`），二进制骨骼数据使用 `.dbbin`（Spine 使用 `.skel`）。两者在功能上高度相似，均支持骨骼层级、关键帧动画、IK 约束和网格自由变形（FFD）。

### 两步装配流程

DragonBones 的组件配置相比 Spine 多一个步骤，因为骨骼数据和图集数据在组件中是分开的两个属性：

1. **设置骨骼数据**：在节点上添加 `DragonBones` 组件，将骨骼数据（.json/.dbbin）拖入 `Dragon Asset` 属性。
2. **设置图集数据**：将图集数据文件（.json）拖入同一个组件的 `Dragon Atlas Asset` 属性。

这种分离设计允许同一套骨骼数据搭配不同的图集（换肤），为角色换装系统提供了灵活性。

### 动画播放与控制

DragonBones 组件挂载后，通过 `playAnimation(name, playTimes)` 方法播放动画。引擎自动处理骨骼变换、网格蒙皮和渲染管线集成。支持标准骨骼动画特性：关键帧插值、动画混合过渡、IK 反向运动学以及网格自由变形。

## 实践注意事项

- **版本上限 v5.6.3**：仅支持 DragonBones v5.6.3 及以下版本导出的数据。使用 DragonBones 编辑器导出时需确认版本兼容性，较新版本的 DragonBones 格式可能不被引擎识别。
- **与 Spine 的选型**：DragonBones 完全免费开源，适合预算有限的独立开发者或需要源码级定制的项目。Spine（付费）生态更成熟，工具链完善，社区资源和第三方素材更丰富。两者在运行时性能和核心动画功能上差异不大，选型更多取决于团队工具链偏好和预算。
- **文件存放**：建议将骨骼数据、图集数据和纹理存放在独立目录中，避免与其他资源混合，提高管理效率。
- **纹理压缩兼容**：DragonBones 的图集纹理需与引擎的纹理压缩流程协调，确保在目标平台的压缩格式兼容。

## 相关页面

- [[资源系统]] — 资源管理总览
- [[动画系统]] — 引擎动画系统架构（含骨骼动画）
- [[Spine 动画]] — 另一款骨骼动画方案，两者对比选型参考

## 原始来源

- [DragonBones 骨骼动画资源](raw/asset/dragonbones.md)
