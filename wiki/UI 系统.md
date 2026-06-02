---
title: UI 系统
date: 2026-06-02
tags:
  - entity
  - ui
aliases: []
---

# UI 系统

> [!abstract] 摘要
> Cocos Creator 的 UI 系统是一套完整的 2D 界面开发框架，以 Canvas（画布）为渲染根节点，通过 UITransform + Widget + Layout 三级布局体系实现多分辨率自适应，内置 Button、Label、Sprite 等 20 多种 UI 组件，支持渲染合批优化与自定义材质扩展。

## 核心概念

### Canvas 画布

Canvas 组件继承自 RenderRoot2D，是所有 2D 渲染元素的渲染数据收集入口。场景中可以有多个 Canvas 节点，**所有 2D 渲染元素都必须作为 RenderRoot2D（Canvas）的子节点才能被渲染**。Canvas 同时担任屏幕适配的核心角色，其设计分辨率和适配方案通过项目设置统一配置。

Canvas 本身与相机无关，其渲染取决于与其节点 layer 对应的 camera，可通过相机的属性控制 Canvas 下节点的渲染效果。Canvas 可关联一个 CameraComponent，并通过 `AlignCanvasWithScreen` 属性自动对齐。

### 设计分辨率与多分辨率适配

多分辨率适配是 UI 系统的核心能力。系统通过设计分辨率（内容制作的蓝本）与屏幕分辨率（实际设备屏幕）的关系进行缩放适配：

- **宽高比相同**：等比缩放，直接放大
- **设计分辨率宽高比 > 屏幕分辨率**：使用**适配高度（Fit Height）**，撑满屏幕高度，左右可能裁剪
- **设计分辨率宽高比 < 屏幕分辨率**：使用**适配宽度（Fit Width）**，撑满屏幕宽度，上下可能裁剪
- **完整显示（Show All）**：同时开启适配高度和宽度，按较小的缩放比例显示，可能出现黑边
- **自动选择**：不开启任何适配模式时，引擎根据宽高比自动选择适配高度或适配宽度

设计分辨率通过 **项目 -> 项目设置 -> 项目数据** 统一配置。

### 三级布局体系

UI 系统的布局能力由三个层级组成：

1. **UITransform** — 基础矩形信息（尺寸 `ContentSize`、锚点 `AnchorPoint`），是所有 UI 节点的必备组件
2. **Widget（对齐挂件）** — 智能对齐父节点（默认）或指定目标节点的任意边，支持百分比和像素混合边距
3. **Layout（自动布局容器）** — 自动排列子节点的容器组件，支持水平/垂直/网格三种布局模式

这三者配合使用，可实现从简单贴边按钮到复杂动态列表的完整 UI 布局需求。

### UI 渲染组件分类

UI 渲染组件分为两大类：

**基础渲染组件**（提供视觉表现）：
- Sprite（精灵图）、Label（文本）、Mask（遮罩）、Graphics（绘图）、RichText（富文本）

**交互控件**（处理用户输入）：
- Button（按钮）、EditBox（输入框）、Toggle（开关）、ToggleContainer（开关组）、Slider（滑动条）、ScrollView（滚动视图）、ScrollBar（滚动条）、PageView（页面视图）、PageViewIndicator（页面指示器）、Progress（进度条）

**特殊组件**：
- WebView（内嵌网页）、VideoPlayer（视频播放器）、SafeArea（安全区适配）、BlockInputEvents（输入阻断）、UIMeshRenderer（自定义网格渲染）

## 关键细节

### Widget 对齐策略

Widget 组件使节点能智能感知屏幕边界位置，始终保持正确的相对布局。核心特性：

- 支持 Top/Bottom/Left/Right 四个方向的对齐，可同时开启相反方向实现自动拉伸（如同时勾选 Left + Right 可使节点宽度自适应）
- 对齐距离支持像素值（如 `40px`）和百分比值（如 `50%`），可混合使用
- 对齐参照默认为父节点，可通过 `Target` 属性指定任意目标节点
- `AlignMode` 决定更新时机：`ALWAYS` 每帧更新、`ONCE` 仅初始化时对齐、`ON_WINDOW_RESIZE` 窗口变化时更新
- 对齐开启后，节点的 position 和 size 可能被限制，不能通过 API 或动画自由修改

### Layout 自动布局

Layout 组件可将节点变为有自动布局功能的容器型节点：

- **水平布局（HORIZONTAL）**：子节点横向排列，支持 `LEFT_TO_RIGHT` / `RIGHT_TO_LEFT` 两种方向
- **垂直布局（VERTICAL）**：子节点纵向排列，与水平布局对称
- **网格布局（GRID）**：固定容器内按网格排列，通过 `StartAxis`、`HorizontalDirection`、`VerticalDirection` 决定起点和排列方向
- **ResizeMode** 控制尺寸自适应：
  - `NONE`：子物体和容器大小互不影响
  - `CHILDREN`：子物体随容器大小变化
  - `CONTAINER`：容器随子物体大小变化
- **Constraint** 约束排列数量：`FIXED_ROW`（固定行数）或 `FIXED_COL`（固定列数）

### Label 文字排版

Label 组件是核心渲染组件，其排版基于 UITransform 的约束框：

- **排版模式（Overflow）**：
  - `NONE`：根据文字尺寸和行高固定约束框
  - `CLAMP`（截断）：超出约束框的文字被隐藏
  - `SHRINK`（自动缩小）：超出时自动缩小文字以全部显示
  - `RESIZE_HEIGHT`（自动适应高度）：约束框高度自动贴合文字内容
- **自动换行**：CLAMP 和 SHRINK 模式下可开启，RESIZE_HEIGHT 模式下强制开启
- 中文以字为单位换行，英文以单词为单位换行
- 可配合 Widget 组件实现灵活的多列布局

### 渲染排序

Canvas 下的 UI 节点采用**深度优先的节点树排序**，节点树顺序即为渲染数据提交顺序。通过 `setSiblingIndex` 改变节点在父节点下的顺序来调整渲染先后。渲染顺序在屏幕上的呈现为从下往上（后提交的在上层）。

从 v3.1 起，UITransform 的 `priority` 属性已弃用，统一使用 `setSiblingIndex` 控制渲染顺序。

### UI 渲染合批

2D 渲染组件合批通过将多个渲染元素的顶点合并提交来减少渲染批次。合批必须满足：
- 节点 Layer 相同
- 使用材质相同（注意材质实例化后无法合批）
- BlendState 和 DepthStencilState 设置相同
- 贴图源及采样相同

**无法合批的组件**：Mask、Graphics、UIMeshRenderer、TiledMap、Spine、DragonBones。

**改善合批的方法**：
- Sprite：使用静态合图或动态合图合并纹理
- Label：使用 Bitmap 缓存模式（不可频繁变动文字内容）
- 通过控制材质和节点树布局，配合合图方法达到最佳效果

v3.4.1 起采用了新的 VB 固定 + IB 每帧重录的渲染数据提交设计，MeshBuffer 容量由 `BATCHER2D_MEM_INCREMENT` 宏控制（默认 144KB，约 4096 个标准顶点）。

### UIStaticBatch 静态合批

UIStaticBatch 组件在初始化时将节点树下的渲染数据收集为静态 IA 渲染数据，后续不再遍历节点树。适用于节点树不会改变的 UI（如 2D 地图）。v3.4.1 后因动态合批性能提升，不再建议手动使用此组件。

### 自定义材质

2D 渲染组件支持自定义材质，用于实现溶解、外发光等效果。使用时需注意：
- 必须使用 2D 专用 Shader（如 builtin-sprite）
- 2D 渲染对象不支持多材质
- 启用自定义材质后 Grayscale 属性失效
- 如需与 3D 物体遮挡，需在自定义材质中开启深度检测

### 九宫格精灵（Sliced Sprite）

通过将图片切分为九宫格区域，Sprite 在放大时保持边缘不变形，中间区域拉伸。这是制作可自适应不同尺寸 UI 背景的核心手段。切分操作在 Sprite 编辑器中通过拖拽绿色分割线完成，使用时将 Sprite 的 `Type` 设为 `Sliced`。

### 动态列表生成

通过脚本遍历数据数组，使用 Prefab 模板动态实例化节点，结合 Layout 组件自动排列，是制作物品栏、选人界面、关卡列表等动态内容面板的标准方案。通常配合 ScrollView 在有限空间内展示大量内容。

## 与其他系统的关系

- **[[场景与节点系统]]**：UI 节点是节点树的子集，Canvas 作为 2D 渲染根节点挂载在场景中，所有 UI 节点必须置于 Canvas 下
- **[[资源系统]]**：UI 组件依赖图片（SpriteFrame）、字体（Font）、Prefab 等资源，九宫格切分在 Sprite 编辑器中完成
- **[[图形渲染]]**：UI 渲染使用 2D 渲染管线，涉及合批、材质、Shader、渲染排序；通过自定义材质可与 3D 场景混合渲染
- **[[脚本系统]]**：UI 交互逻辑通过 TypeScript 脚本驱动，动态列表生成、按钮回调、数据绑定等均依赖脚本

## 注意事项

> [!warning] 渲染异常排查
> 如果遇到 UI 渲染出错、花屏、闪屏等现象，首先检查场景中所有相机的 ClearFlag，确保至少有一个相机执行 `SOLID_COLOR` 清屏操作。多层渲染场景中，UI Canvas 的相机应设置为 `DEPTH_ONLY`。

> [!warning] Canvas 节点要求
> 所有 2D 渲染元素必须作为 RenderRoot2D（Canvas）的子节点才能被渲染，非 Canvas 下的 2D 节点不会被渲染。

> [!warning] Widget 性能优化
> 对于有很多 UI 元素的场景，将 Widget 的 AlignMode 设置为 `ON_WINDOW_RESIZE`，可大幅提高运行性能。避免使用 `ALWAYS` 模式除非需要每帧更新对齐。

> [!warning] 九宫格尺寸限制
> Sliced Sprite 的 size 属性值不能为负数，否则无法正常显示九宫格效果。

> [!danger] 内存与合批
> BATCHER2D_MEM_INCREMENT 增大可容纳更多顶点合批，但内存占用也随之增大。其最大值不可超过约 2304KB（对应 65535 个顶点上限）。

## 相关页面

- [[场景与节点系统]]
- [[资源系统]]
- [[图形渲染]]
- [[脚本系统]]

## 原始来源

- [UI 实践指南](raw/ui-system/components/engine/usage-ui.md)
- [多分辨率适配方案](raw/ui-system/components/engine/multi-resolution.md)
- [对齐策略](raw/ui-system/components/engine/widget-align.md)
- [文字排版](raw/ui-system/components/engine/label-layout.md)
- [自动布局容器](raw/ui-system/components/engine/auto-layout.md)
- [制作动态生成内容的列表](raw/ui-system/components/engine/list-with-data.md)
- [制作可任意拉伸的 UI 图像](raw/ui-system/components/engine/sliced-sprite.md)
- [2D 渲染组件合批规则说明](raw/ui-system/components/engine/ui-batch.md)
- [2D 渲染对象自定义材质](raw/ui-system/components/engine/ui-material.md)
- [渲染排序说明](raw/ui-system/components/engine/priority.md)
- [Canvas 组件参考](raw/ui-system/components/editor/canvas.md)
- [UI 组件](raw/ui-system/components/editor/base-component.md)
- [2D 渲染组件介绍](raw/ui-system/components/editor/render-component.md)
- [UI 变换组件](raw/ui-system/components/editor/ui-transform.md)
- [Widget 组件参考](raw/ui-system/components/editor/widget.md)
- [Layout 组件参考](raw/ui-system/components/editor/layout.md)
- [Button 组件](raw/ui-system/components/editor/button.md)
- [Sprite 组件](raw/ui-system/components/editor/sprite.md)
- [Label 组件](raw/ui-system/components/editor/label.md)
- [Mask 组件](raw/ui-system/components/editor/mask.md)
- [Graphics 组件](raw/ui-system/components/editor/graphics.md)
- [RichText 组件](raw/ui-system/components/editor/richtext.md)
- [ScrollView 组件](raw/ui-system/components/editor/scrollview.md)
- [ScrollBar 组件](raw/ui-system/components/editor/scrollbar.md)
- [EditBox 组件](raw/ui-system/components/editor/editbox.md)
- [Toggle 组件](raw/ui-system/components/editor/toggle.md)
- [ToggleContainer 组件](raw/ui-system/components/editor/toggleContainer.md)
- [Slider 组件](raw/ui-system/components/editor/slider.md)
- [Progress 组件](raw/ui-system/components/editor/progress.md)
- [PageView 组件](raw/ui-system/components/editor/pageview.md)
- [PageViewIndicator 组件](raw/ui-system/components/editor/pageviewindicator.md)
- [WebView 组件](raw/ui-system/components/editor/webview.md)
- [VideoPlayer 组件](raw/ui-system/components/editor/videoplayer.md)
- [SafeArea 组件](raw/ui-system/components/editor/safearea.md)
- [BlockInputEvents 组件](raw/ui-system/components/editor/block-input-events.md)
- [UIStaticBatch 组件](raw/ui-system/components/editor/ui-static.md)
- [UIOpacity 组件](raw/ui-system/components/editor/ui-opacity.md)
- [UISkew 组件](raw/ui-system/components/editor/ui-skew.md)
- [UIModel 组件](raw/ui-system/components/editor/ui-model.md)
- [UICoordinateTracker 组件](raw/ui-system/components/editor/ui-coordinate-tracker.md)
- [RenderRoot2D 组件](raw/ui-system/components/editor/renderroot2d.md)
- [LabelShadow 组件](raw/ui-system/components/editor/label-shadow.md)
- [LabelOutline 组件](raw/ui-system/components/editor/label-outline.md)
