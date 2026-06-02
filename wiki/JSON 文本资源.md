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
> JSON 资源是导入到 Cocos Creator 中的 JSON 文件，类型为 `cc.JsonAsset`，支持编辑器挂载和代码动态加载两种使用方式。

## 概述

Cocos Creator 支持将 JSON 文件作为资源导入，类型为 `cc.JsonAsset`。可通过 `@property(JsonAsset)` 在脚本中声明属性并拖拽赋值，也可用 `resources.load` 动态加载。加载后通过资源的 `.json` 属性获取已解析的 JSON 对象数据。

## 相关页面

- [[资源系统]] — 资源管理总览
- [[脚本系统]] — 脚本中动态加载 JSON 数据
- [[引擎架构]] — 资源在引擎中的架构位置
- [[JavaScript 数据类型]] — JS JSON 对象的原生支持

## 原始来源
- [JSON 资源](raw/asset/json.md)
