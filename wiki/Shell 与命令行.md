---
title: Shell 与命令行
date: 2026-06-10
tags:
  - layer/linux
  - type/concept
  - status/stable
aliases: []
---

# Shell 与命令行

> [!abstract] 摘要
> Shell 是用户与操作系统内核之间的命令解释器——它接收人类可读的命令文本，翻译为系统调用，将结果返回给用户。Shell 的核心威力不在于单个命令，而在于**组合能力**：管道（`|`）让一个命令的输出成为另一个命令的输入，重定向（`>`/`<`）让文件参与数据流，脚本让这些组合自动化。这种"小工具 + 管道 = 大能力"的哲学就是 Unix 哲学的核心——每个工具只做一件事并做好它，复杂功能由组合产生。这一哲学在 JS 的模块系统（小模块 → 大应用）和 Rust 的迭代器链（小适配器 → 大转换）中都能看到共鸣。

## 根本问题：人类如何向 OS 内核下达指令

操作系统的内核只理解系统调用（syscall）——机器层面的操作指令。用户不可能每次操作都直接写系统调用。Shell 提供了三个层次的抽象：

1. **命令解释**：将 `ls -la` 翻译为 `open`/`read`/`write` 等系统调用
2. **脚本编程**：变量、条件、循环——让命令序列成为程序
3. **环境管理**：环境变量、工作目录、权限上下文

### Bash

Bash（Bourne Again Shell）是大多数 Linux 发行版的默认 shell。提示符格式 `user@host:path$`——`$` 后的光标等待输入。其他 shell（zsh、fish）提供更好的交互体验（自动补全、语法高亮、主题），但核心概念是通用的。

## 核心命令结构

```
command [options] [arguments]
```

- **command**：可执行程序名（在 `$PATH` 枚举的目录中查找）
- **options**：以 `-` 或 `--` 开头，修改命令行为
- **arguments**：命令操作的对象（文件、字符串等）

## 管道与重定向

### 管道

```
command1 | command2
```

`command1` 的标准输出（stdout）直接成为 `command2` 的标准输入（stdin）。这是 Unix 哲学的核心操作符——不需要临时文件，数据在命令间像水流一样传递。

### 重定向

| 操作符 | 含义 |
|--------|------|
| `>` | stdout 写入文件（覆盖） |
| `>>` | stdout 追加到文件 |
| `<` | 文件内容作为 stdin |
| `2>` | stderr 重定向 |
| `&>` | stdout 和 stderr 合并重定向 |
| `2>&1` | stderr 合并到 stdout |

### `/dev/null`

特殊文件——写入它的数据被丢弃，从它读取返回 EOF。用于有意忽略输出：`command 2>/dev/null`（忽略错误输出）。

## 文本处理命令

| 命令 | 作用 |
|------|------|
| `grep` | 搜索文本模式 |
| `sed` | 流式文本编辑（替换、删除） |
| `awk` | 文本处理语言（列提取、计算、报表） |
| `sort` | 排序 |
| `uniq` | 去重（通常与 sort 联用） |
| `wc` | 统计行数/词数/字节数 |
| `head`/`tail` | 查看文件开头/结尾 |
| `cut` | 按列提取 |
| `tr` | 字符转换 |

典型组合：`cat log.txt | grep ERROR | sort | uniq -c | sort -rn | head -10`（找出出现最多的 10 种错误）。

## 环境变量

```
echo $PATH         # 查看 PATH
export MYVAR=hello # 设置环境变量（对子进程可见）
```

`$PATH` 是最关键的环境变量——shell 在这些目录中查找命令。

## Shell 脚本基础

```bash
#!/bin/bash
# 第一行是 shebang——告诉 OS 用哪个解释器执行

for file in *.txt; do
    echo "Processing $file"
    wc -l "$file"
done
```

## 跨系统对比

| 概念 | Linux Shell | JavaScript |
|------|-----------|------------|
| 组合方式 | 管道 `\|` | 方法链 `.filter().map()` |
| 错误处理 | 退出码 + stderr | try-catch + Promise rejection |
| 抽象单元 | 命令 | 函数 |
| 领域 | 系统管理 | 应用开发 |

Shell 的管道和 JS 数组方法的链式调用是同构的——都是"数据流经一系列转换"的思想。Rust 的迭代器适配器链也是如此。

## 注意事项

- **管道中的命令是并行执行的**：不是串行——`cmd1 | cmd2` 中两者同时运行，数据边产生边传递
- **退出码**：`$?` 获取上一个命令的退出状态（0 = 成功，非 0 = 失败）。脚本中用 `set -e` 让任何命令失败时脚本立即退出
- **空格是有意义的**：`var = value`（错误，shell 会认为 `var` 是命令名）→ `var=value`（正确）

## 相关页面

- [[Linux 概述]] — Linux 知识体系总览
- [[Linux 文件系统]] — 命令操作的对象（目录结构、文件类型）
- [[Linux 权限体系]] — 命令执行的权限限制
- [[Linux 进程模型]] — 命令作为进程被 fork+exec
- [[Jujutsu VCS]] — 命令行工具的日常使用
- [[Git 与版本控制]] — 命令行 Git 操作
- [[Cargo 与 crate 生态]] — `cargo` 作为语言级"命令集合"与 shell 管道的概念共鸣

## 原始来源

- [Shell](raw/linuxjourney/lessons/zh/command-line/the-shell.md)
- `raw/linuxjourney/lessons/zh/command-line/` 下 19 个命令课程
- `raw/linuxjourney/lessons/zh/text-fu/` 下 17 个文本处理课程
