---
title: TiledMap
date: 2026-06-02
tags:
  - layer/cocos
  - type/entity
  - status/stable
aliases: []
---

# TiledMap

> [!abstract] 摘要
> TiledMap 瓦片图资源由 Tiled 编辑器导出，Cocos Creator v3.0+ 支持 Tiled v1.4，用于在游戏场景中创建瓦片地图。

## 概述

TiledMap 资源包含地图数据（.tmx）、图集纹理（.png）和 tileset 配置文件（.tsx）。导入后拖拽到 TiledMap 组件的 Tmx File 属性即可。v3.8.5 起引擎不再自动缩进 1 像素，制作图集时需在 TexturePacker 中设置拉伸 1 像素来避免边缘异常。tmx 和 tsx 需放在同一目录。

## 相关页面
- [[资源系统]]
- [[纹理贴图]]
- [[精灵帧]]

## 原始来源
- [瓦片图资源](raw/asset/tiledmap.md)
