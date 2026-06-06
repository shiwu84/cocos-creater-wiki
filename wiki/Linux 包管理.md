---
title: Linux 包管理
date: 2026-06-10
tags:
  - layer/linux
  - type/concept
  - status/stable
aliases: []
---

# Linux 包管理

> [!abstract] 摘要
> 包管理器解决的根本问题是**依赖地狱**：软件 A 需要库 X v1.2，软件 B 需要库 X v2.0——如果手动安装，要么覆盖要么冲突。包管理器通过集中的仓库、冲突检测和自动依赖解析，将软件安装从"下载→配置→编译→祈祷"简化为一条命令。Linux 包管理有两层：底层工具（dpkg/rpm）管理单个包文件，上层工具（apt/yum/dnf/pacman）管理仓库和依赖——后者是用户日常交互的接口。这一模型启发了所有现代语言包管理器（Cargo、npm、pip），它们的核心架构（注册中心 + 依赖图解析）本质上与 apt 同构。

## 根本问题：软件的分发与依赖协调

在没有包管理器的时代，Linux 上安装软件需要：
1. 下载源码 tarball
2. 安装编译工具链和依赖库（自己找、自己编译）
3. `./configure && make && make install`（处理各种编译错误）
4. 发现缺少依赖 → 回到第 2 步
5. 更新软件 → 重复以上过程

包管理器将这个过程自动化，同时解决了版本冲突问题。

## 包格式与工具

### 两大生态

| | Debian 系 (apt/dpkg) | Red Hat 系 (yum/dnf/rpm) |
|---|---|---|
| **发行版** | Debian, Ubuntu, Mint | RHEL, CentOS, Fedora |
| **包格式** | `.deb` | `.rpm` |
| **底层工具** | `dpkg` | `rpm` |
| **上层工具** | `apt`/`apt-get` | `yum`/`dnf` |
| **安装** | `apt install pkg` | `dnf install pkg` |
| **删除** | `apt remove pkg` | `dnf erase pkg` |
| **搜索** | `apt search keyword` | `dnf search keyword` |
| **更新仓库** | `apt update` | `dnf check-update` |

> **注意**：Fedora/RHEL 8+ 中 `yum` 已被 `dnf` 取代——`dnf` 使用更现代的实现但命令兼容。

### 其他包管理器

| 工具 | 特点 | 代表发行版 |
|------|------|-----------|
| `pacman` | 简洁高效，滚动更新 | Arch Linux |
| `zypper` | SAT 求解器依赖解析 | openSUSE |
| `apk` | 极简，为容器设计 | Alpine Linux |
| `snap`/`flatpak` | 沙盒化，跨发行版 | 通用 |

## 底层 vs 上层

- **底层**：只管安装单个 `.deb`/`.rpm` 文件，不处理依赖。`dpkg -i package.deb` 如果缺少依赖会失败。
- **上层**：知道仓库地址，自动下载依赖树，按顺序安装。`apt install nginx` 可能先安装/更新十几个依赖包。

## 仓库

包管理器从**仓库**（repository）下载软件。仓库是 HTTP/FTP 服务器上按特定布局组织的包集合。每个发行版维护自己的官方仓库（安全审计、签名验证）。

`/etc/apt/sources.list`（Debian）或 `/etc/yum.repos.d/`（RPM）定义仓库地址。添加第三方仓库（如 Docker、Chrome）时需特别注意安全性——它们可能没有官方仓库的审计级别。

## 与语言包管理器的概念共鸣

| 概念 | Linux 包管理 | Cargo | npm | pip |
|------|------------|-------|-----|-----|
| 注册中心 | apt repository | crates.io | npm registry | PyPI |
| 包文件 | `.deb`/`.rpm` | `.crate` | `.tgz` | `.whl` |
| 锁文件 | 隐式（包管理器自己追踪） | `Cargo.lock` | `package-lock.json` | `requirements.txt` |
| 依赖解析 | 发行版负责 | 语义化版本 | 语义化版本 | 最少（手动管理） |
| 安全模型 | 签名包 + 官方仓库 | 无签名验证 | `npm audit` | 无 |

Linux 包管理的依赖解析是最复杂的——因为系统级库（如 libc、OpenSSL）会被数十个包共享，任何一个包的版本需要与所有依赖方兼容。语言包管理器通常在自己的生态内做解析，不存在跨生态冲突（npm 包不会要求特定版本的 libc）。

## 编译安装 vs 包安装

包安装是推荐方式：自动依赖、自动更新、安全审计。编译安装只在以下情况用：
- 软件不在仓库中
- 需要启用/禁用特定编译选项
- 需要最新版本（仓库版本落后）

Arch Linux 的 AUR（Arch User Repository）和 `makepkg` 提供了第三种路径：用户上传编译脚本，其他人可以一键下载编译。

## 注意事项

- **不要混用不同发行版的包**：把 Ubuntu 的 `.deb` 装在 Debian 上可能因为依赖版本冲突而损坏系统
- **`apt update` 不等于 `apt upgrade`**：前者刷新包列表（不改变安装的包），后者实际更新安装的包
- **删除不需要的依赖**：`apt autoremove` 清除不再被任何包需要的孤立依赖
- **包管理器本身是软件**：如果仓库中的 pip 版本太旧，不要 `apt install python3-pip` 然后 `pip install --upgrade pip`——这可能导致系统工具（依赖特定 pip 版本）出问题

## 相关页面

- [[Linux 概述]] — Linux 知识体系总览
- [[Linux 文件系统]] — 包安装文件的存放位置（FSH 标准）
- [[Linux 权限体系]] — 安装包的权限要求（root 安装系统包）
- [[Cargo 与 crate 生态]] — Rust 包管理，同构的概念对比
- [[软件工程概述]] — 依赖管理作为 SWE 通用概念
- [[systemd 服务管理]] — 包安装的服务由 systemd 管理

## 原始来源

- [包管理系统](raw/linuxjourney/lessons/zh/packages/package-management-systems.md)
- [软件分发](raw/linuxjourney/lessons/zh/packages/software-distribution.md)
- [包依赖](raw/linuxjourney/lessons/zh/packages/package-dependencies.md)
- [编译安装](raw/linuxjourney/lessons/zh/packages/compile-source-code.md)
