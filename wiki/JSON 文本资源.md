---
title: JSON 文本资源
date: 2026-06-02
tags:
  - layer/cocos
  - type/entity
  - status/stable
aliases: []
---

# JSON 文本资源

> [!abstract] 摘要
> JSON 文本资源解决游戏中的数据驱动问题——如何将配置数据、关卡定义、本地化文本等结构化或纯文本信息作为引擎资源管理，而非硬编码在脚本中。Cocos Creator 将文本文件导入为两种资源类型：`cc.JsonAsset`（JSON 文件，自动解析为 JavaScript 对象）和 `cc.TextAsset`（纯文本及 XML、YAML、CSV 等格式，保留原始字符串），分别对应结构化数据和自由文本两种场景，均支持编辑器挂载和代码动态加载。

## 核心概念

### 两种资源类型

Cocos Creator 通过资源导入将文本文件转换为两类引擎资源：

| 资源类型 | 支持格式 | 访问属性 | 典型用途 |
|---------|---------|---------|---------|
| **`cc.JsonAsset`** | `.json` | `.json`（已解析的 JS 对象） | 游戏配置、关卡数据、道具表、技能定义 |
| **`cc.TextAsset`** | `.txt`, `.xml`, `.plist`, `.yaml`, `.ini`, `.csv`, `.md` | `.text`（原始字符串） | 对话文本、XML 配置、CSV 数据表 |

两者的核心区别：JsonAsset 在导入时自动完成 JSON 解析，使用时直接获得 JavaScript 对象；TextAsset 保留原始文本内容，开发者需自行解析非 JSON 格式的数据。

### 两种使用模式

**编辑器挂载**：在脚本中用 `@property(JsonAsset)` 或 `@property(TextAsset)` 声明属性，然后在编辑器中将资源拖入属性栏。适用于数量固定的配置文件，如单个道具表或技能配置。

**代码动态加载**：通过 `resources.load('path', callback)` 在运行时异步加载。适用于按需加载的场景，如按关卡加载对应的敌人配置、按语言加载本地化文本。加载路径相对于 `resources/` 目录，不含文件扩展名。

### 与引擎组件的集成

JSON 文本资源不直接与渲染组件关联，而是作为数据源由脚本消费。典型集成路径：
- 配置文件 → JsonAsset → 脚本解析 → 初始化游戏系统（道具系统、技能系统）
- 本地化文本 → TextAsset → 脚本读取 → 设置 Label 组件文本
- CSV 数据表 → TextAsset → 脚本解析 → 填充数据驱动的 UI 列表

## 实践注意事项

- **resources.load 路径规则**：动态加载时路径相对于 `resources/` 目录，不包含文件扩展名。例如 `resources/items.json` 应写为 `resources.load('items', ...)`。
- **JsonAsset 安全性**：JSON 文件在导入时即被解析，不经过运行时 `JSON.parse`，避免了注入风险，但格式错误在编辑器阶段就会报错。
- **编辑器挂载 vs 动态加载**：固定配置优先编辑器挂载（简单直观），大量按需加载的数据使用动态加载（节省首屏内存）。
- **TextAsset 手动解析**：非 JSON 格式（XML、CSV 等）的 TextAsset 需要开发者自行实现解析逻辑，这部分工作不受引擎框架约束。

## 相关页面

- [[资源系统]] — 资源管理总览，含导入和动态加载流程
- [[脚本系统]] — 脚本中声明和消费资源属性
- [[UI 系统]] — 文本资源常作为 Label 组件的文字渲染数据源

## 原始来源

- [JSON 资源](raw/asset/json.md)
- [文本资源](raw/asset/text.md)
