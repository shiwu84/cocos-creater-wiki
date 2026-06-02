# AGENTS.md — LLM Wiki Schema

本文件是 LLM Wiki 的 Schema 配置。`llm-wiki.md` 是概念文档（不修改），本文件是具体约定。

## 架构

```
llm-wiki.md          # 概念参考（不修改）
AGENTS.md            # Schema 约定（LLM + 用户共同维护）
raw/                 # 原始源文档 — 只读，绝不修改
wiki/                # LLM 生成的 Wiki 页面 — LLM 全权拥有
output/              # 生成物（幻灯片、图表、canvas 等）
index.md             # Wiki 内容目录（每次 Ingest 后更新）
log.md               # 操作日志（追加写入，格式: ## [日期] 类型 | 描述）
```

- **禁止**修改 `raw/` 下任何文件。只读取、引用、摘录。
- **禁止**主动修改 `llm-wiki.md`，除非用户明确要求更新 Schema 设计。
- **使用中文回复用户**。
- 对笔记（.md 文件）的读写操作，优先使用 `obsidian-markdown` skill；涉及 `raw/` 源文档的具体文件路径确认则使用 Bash。
- 涉及多个文件的操作时，优先使用多智能体并行处理。
- 根据任务需要，优先选择对应的 Obsidian skill：
  - `obsidian-markdown` — .md 文件的读写编辑
  - `obsidian-cli` — 文件搜索、批量操作、属性管理
  - `json-canvas` — .canvas 图谱文件
  - `obsidian-bases` — .base 数据库视图

## 命名规范

- Wiki 页面用**中文**文件名，简洁描述性标题
- 实体页：`场景.md`、`Camera 组件.md`
- 概念页：`渲染管线.md`、`ECS 架构.md`
- 对比页：`Cocos Creator vs Unity.md`
- 概述页：以 `XXX 概述` 命名
- 特殊文件：`index.md`、`log.md` 固定小写英文

## 页面模板

每个 Wiki 页面必须包含以下结构：

```markdown
---
title: 页面标题
date: YYYY-MM-DD
tags:
  - tag1
  - tag2
aliases: []
---

# 页面标题

> [!abstract] 摘要
> 一段 2-3 句话的核心摘要。

## 核心概念
## 关键细节
## 与其他系统的关系
## 注意事项
## 相关页面
- [[相关页面1]]
- [[相关页面2]]

## 原始来源
- [display](raw/relative/path.md)
```

## 交叉引用规则

| 目标类型 | 语法 | 示例 |
|---------|------|------|
| Wiki 页面 | `[[页面名]]` | `[[物理系统]]` |
| raw/ 源文档 | `[描述](raw/path.md)` | `[节点组件](raw/concepts/scene/node-component.md)` |
| 块引用 | `[[页面名#^block-id]]` | `[[引擎架构#更新循环]]` |
| 外部 URL | 标准 Markdown | `[Cocos](https://...)` |

## 标签

| 标签 | 用途 |
|------|------|
| `#overview` | 概述/合成页面 |
| `#entity` | 实体（场景、组件、资源类型） |
| `#concept` | 概念页面 |
| `#comparison` | 对比页面 |
| `#todo` | 待后续补充 |
| `#v3.8` | 3.8 版本相关特性 |

## Ingest 工作流

处理 raw/ 中的源文档时：

1. 读源文档全文
2. 与用户讨论关键发现和重点
3. 创建/更新该主题的 Wiki 页面
4. 跨 wiki 找到所有相关页面并更新它们的交叉引用
5. 更新 `index.md`（添加到对应分类表格中）
6. 在 `log.md` 末尾追加操作记录
7. **验证 raw/ 链接**：创建页面后，运行以下命令确认所有 raw 引用指向实际存在的文件：

```bash
grep -hPo '\(raw/[^)]+\)' <新页面> \
  | sed 's/[()]//g' | while read f; do \
    [ -f "$f" ] && echo "OK: $f" || echo "DEAD: $f"; \
  done
```

> [!caution] 部分 raw/ 子目录没有 `index.md`（如 `raw/physics-2d/` 的主文件是 `physics-2d.md`）。引用前先 `ls` 确认实际文件名。

一个源文档通常触及 10–15 个 wiki 页面，每次 Ingest 后都应运行链路验证。

## Query 工作流

1. 先读 `index.md` 定位相关页面
2. 读 wiki 页面和 raw/ 源文档
3. 合成回答，附引用
4. 有价值的分析、比较、发现应归档回 wiki（新建或更新页面）

## Lint 检查

定期对 wiki 做健康检查。关注：
- 页面间矛盾声明
- 过时内容（与新版源文档对照）
- 孤儿页面（零入站链接）
- 遗漏的交叉引用
- 死链接

全量死链检查命令：

```bash
grep -rhoP '\(raw/[^)]+\)' wiki/*.md \
  | sed 's/[()]//g' | sort -u | while read f; do \
    [ -f "$f" ] || echo "DEAD: $f"; \
  done
```

## 领域上下文

- 领域：**Cocos Creator 3.8 游戏引擎**
- 源文档：Cocos Creator 3.8 LTS 中文用户手册（~583 个 .md 文件）
- 关键主题：场景/节点系统、资源管理、TypeScript 脚本、图形渲染、动画、3D/2D 物理、UI 系统、粒子/音频/缓动、跨平台发布、编辑器扩展

## 版本

- 更新日期：2026-06-02
- Schema 版本：v1.1
