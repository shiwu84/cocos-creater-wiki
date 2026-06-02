# 操作日志

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
