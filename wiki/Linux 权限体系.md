---
title: Linux 权限体系
date: 2026-06-02
tags:
  - layer/linux
  - type/concept
  - status/stable
aliases:
  - 权限模型
  - DAC
  - 自主访问控制
---

# Linux 权限体系

> [!abstract] 摘要
> Linux 权限体系是操作系统自主访问控制（DAC）的完整实现——每个文件归属于一个用户和一个组，通过 rwx 三元位对所有者、组、其他三类访问者分别控制读、写、执行权限。九位标准权限之上，SUID（以文件所有者身份运行）、SGID（以组身份运行；目录中继承组）、粘滞位（只有文件所有者可删除）三个特殊位提供了细粒度的提权和共享场景支持。进程继承了其执行者的权限身份，但通过 RUID/EUID/saved-UID 的三重 UID 机制，SUID 程序可以在需要时临时提权、完成任务后恢复原始身份。底层的 `/etc/passwd` + `/etc/shadow` + `/etc/group` 三文件构成了用户名、密码哈希、组成员关系的持久化基础。这套权限体系不是孤立的 OS 特性——它是所有现代访问控制（RBAC、capabilities）的原型，也是上层编程语言文件安全 API 的语义来源。

## 根本问题

多用户操作系统面临一个根本矛盾：**如何让多个用户在共享同一台机器的同时互不侵犯？**

如果 Alice 的所有文件对 Bob 完全可读可写，Alice 没有任何隐私可言。如果任何用户都能执行 `rm -rf /`，系统十分钟内就会崩溃。Linux 的答案是**自主访问控制（Discretionary Access Control, DAC）**——每个文件的所有者自主决定"谁"能以"什么方式"访问该文件。

DAC 的"自主"意味着：文件的访问权限由文件所有者自行设置，而非由系统管理员统一管控。这与 SELinux 的强制访问控制（MAC）形成对比——MAC 中系统策略强制执行，所有者也无权绕过。

## rwx 权限：三层 × 三位

### 权限字符串结构

`ls -l` 输出的第一个字段揭示了一切：

```
-rwxr-xr--  1 alice developers  4096 Jun 02 10:00 script.sh
 ↑↑↑↑↑↑↑↑↑↑
 │├─┬─┤├─┬─┤├─┬─┤
 ││ │ ││ │ ││ │ └── 其他用户：r--（只读）
 ││ │ ││ │ │└──── 其他用户：w（无写入）
 ││ │ ││ │ └───── 其他用户：x（无执行）
 ││ │ ││ └─────── 组：r（可读）
 ││ │ │└──────── 组：w（无写入）
 ││ │ └───────── 组：x（可执行）
 ││ └─────────── 所有者：r（可读）
 │└──────────── 所有者：w（可写）
 └───────────── 所有者：x（可执行）
- 表示普通文件（d=目录，l=符号链接，b=块设备，c=字符设备，p=管道，s=socket）
```

九个字符分三组，每组 `rwx` 三个位，缺失的权限用 `-` 表示。

### rwx 在文件上的语义

| 权限 | 符号 | 对文件的意义 |
|------|------|------------|
| **读 (Read)** | `r` | 可以读取文件内容（`cat`、`less`、打开读取） |
| **写 (Write)** | `w` | 可以修改文件内容（`vim`、`>` 重定向写入） |
| **执行 (Execute)** | `x` | 可以作为程序运行（`./script.sh`、`/usr/bin/python`） |

> [!warning] 注意
> 对文件有写权限不代表可以删除文件——删除文件需要其**所在目录**的写权限。对脚本文件有读权限但无执行权限，可以 `bash script.sh` 运行（因为执行的是 bash 本身），但不能 `./script.sh` 直接运行。

### rwx 在目录上的语义

> [!tip] 关键区别
> 目录上的 rwx 含义与文件上的完全不同——这是初学者最容易混淆的地方。

| 权限 | 符号 | 对目录的意义 |
|------|------|------------|
| **读 (Read)** | `r` | 可以列出目录内容（`ls`） |
| **写 (Write)** | `w` | 可以在目录中创建、删除、重命名文件 |
| **执行 (Execute)** | `x` | 可以进入目录（`cd`），访问目录内文件的 inode |

目录的 `x` 权限是最关键的——没有它，即使你有 `r` 权限也无法 `cd` 进入目录，也无法访问目录内任何文件。典型组合：

| 八进制 | 权限串 | 效果 |
|--------|--------|------|
| `7` | `rwx` | 完全控制：可列表、可增删、可进入 |
| `5` | `r-x` | 只读浏览：可列表、可进入，不可修改 |
| `1` | `--x` | 盲入：不可列表（不知道里面有什么），但如果知道确切的文件名可以直接访问 |
| `3` | `-wx` | 盲写：不可列表，但可以创建/删除文件（如果知道名字） |
| `0` | `---` | 完全拒绝 |

> [!note] 删除文件的权限归属
> 删除文件不受文件自身权限控制，而是受其**所在目录**的写+执行权限控制。这就是为什么 `/tmp` 目录中你有权删除自己创建的文件（粘滞位特例见下文）——因为你对 `/tmp` 有写权限。

## 八进制表示法与 chmod

### 数字对应

每个权限位映射到一个八进制数字：

| 权限 | 八进制值 | 含义 |
|------|---------|------|
| `r` | 4 | 读 |
| `w` | 2 | 写 |
| `x` | 1 | 执行 |
| `-` | 0 | 无权限 |

每组的三个位**相加**得到该组的权限值：

```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
-wx = 0+2+1 = 3
-w- = 0+2+0 = 2
--x = 0+0+1 = 1
--- = 0+0+0 = 0
```

### chmod 两种模式

**符号模式**（读友好，适合增量修改）：

```bash
chmod u+x file         # 给所有者加执行权限
chmod g-w file         # 移除组的写权限
chmod o= file          # 清空其他用户全部权限
chmod a+r file         # 给所有人加读权限
chmod ug+rwx file      # 给所有者和组加全部权限
chmod u=rwx,g=rx,o= file  # 精确设置三组权限
```

符号标记：`u`（用户/所有者）、`g`（组）、`o`（其他人）、`a`（所有人）；`+`（添加）、`-`（移除）、`=`（精确设置）。

**数字模式**（精确，适合一次性设置全部）：

```bash
chmod 755 script.sh    # rwxr-xr-x（所有者全部，组和他人读+执行）
chmod 644 config.ini   # rw-r--r--（所有者读写，组和他人只读）
chmod 600 id_rsa       # rw-------（只有所有者可读写，SSH 私钥标准权限）
chmod 777 public/      # rwxrwxrwx（所有人全部权限——危险！）
```

> [!warning] `chmod -R 777` 是常见反模式
> 递归设置 777 授予所有人对目录树的完全控制权，这几乎从不正确。遵循**最小权限原则**：只授予完成工作所需的权限。配置文件和脚本通常只需 `644`（不可执行）；敏感文件（私钥、密码文件）应为 `600`。

### 常见权限速查

| 场景 | 八进制 | 权限串 | 说明 |
|------|--------|--------|------|
| 可执行脚本 | `755` | `rwxr-xr-x` | 所有者可修改运行，其他人可运行 |
| 普通配置文件 | `644` | `rw-r--r--` | 所有者可修改，其他人只读 |
| SSH 私钥 | `600` | `rw-------` | 只有所有者可读写 |
| 共享目录 | `775` | `rwxrwxr-x` | 所有者+组可修改，他人可浏览 |
| Web 目录 | `755` | `rwxr-xr-x` | Apache/Nginx 需要 x 进入目录 |
| 系统配置 | `640` | `rw-r-----` | 所有者+组可读，他人禁止 |

## 文件所有权：chown 与 chgrp

每个文件有且仅有一个**所有者用户**和一个**所有者组**，由 inode 中的 UID 和 GID 字段记录。

### 查看所有权

```bash
ls -l
# -rw-r--r-- 1 alice developers 4096 Jun 02 10:00 note.txt
#            ↑     ↑
#         所有者  组
```

### 修改所有权

```bash
# 只改所有者
sudo chown bob note.txt          # 所有者改为 bob

# 只改组
sudo chgrp admins note.txt       # 组改为 admins

# 同时改所有者和组
sudo chown bob:admins note.txt   # 所有者→bob, 组→admins

# 只改组（chown 的 :语法）
sudo chown :admins note.txt      # 只改组（冒号前留空）

# 递归修改
sudo chown -R alice:developers /home/alice/
```

> [!tip] chown 需要 root 权限
> 只有 root 可以更改文件的所有者。这是因为一旦你把文件所有权交给别人，你就失去了对该文件权限的控制——系统要求变更所有权必须是特权操作。普通用户可以更改文件所属组为他们所在的组。

## umask：默认权限掩码

每个新创建的文件都有一个默认权限集合。`umask` 不是"设置默认权限"，而是**从最大权限中减去某些位**。

```
文件最大权限: rw-rw-rw- (666)  ← 文件默认不含 x（安全考虑）
目录最大权限: rwxrwxrwx (777)  ← 目录默认需要 x 进入
```

umask 值从最大权限中**减去**对应的位：

```bash
umask 022
# 新文件: 666 - 022 = 644 (rw-r--r--)  ← 组和其他无写权限
# 新目录: 777 - 022 = 755 (rwxr-xr-x)
```

常见 umask 值：

| umask | 新文件权限 | 新目录权限 | 场景 |
|-------|----------|----------|------|
| `022` | `644` (rw-r--r--) | `755` (rwxr-xr-x) | 大多数发行版默认，组和其他不可写 |
| `002` | `664` (rw-rw-r--) | `775` (rwxrwxr-x) | 协作环境，组成员可写 |
| `077` | `600` (rw-------) | `700` (rwx------) | 个人隐私，只有所有者可访问 |
| `027` | `640` (rw-r-----) | `750` (rwxr-x---) | 组只读，其他人完全禁止 |

```bash
# 查看当前 umask
umask

# 临时设置
umask 027

# 持久化：写入 shell 启动文件（~/.bashrc、~/.profile）
echo "umask 027" >> ~/.bashrc
```

## 特殊权限位：SUID、SGID、粘滞位

标准九位权限之上还有三个特殊位，分别用八进制的前缀数字表示：4=SUID, 2=SGID, 1=粘滞位。

### SUID (Set User ID) — 以文件所有者身份运行

SUID 解决了**普通用户需要临时提升权限执行特定操作**的矛盾。当一个可执行文件设置了 SUID 位，任何用户运行它时，进程的**有效 UID** 被设为文件所有者的 UID，而非运行者的 UID。

**典型例子：`passwd` 命令**

```bash
$ ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 47032 Dec 1 11:45 /usr/bin/passwd
#    ↑
#  s = SUID + x（所有者位出现 s 替代 x）
```

`/usr/bin/passwd` 归 root 所有，设置了 SUID。当你运行 `passwd`：进程以 **root**（UID 0）的有效 UID 运行，所以它能写入只有 root 可写的 `/etc/shadow` 文件来更新你的密码。但程序内部仍知道你的真实 UID，从而限制你只能修改自己的密码。

```bash
# 设置 SUID
chmod u+s myprogram        # 符号模式
chmod 4755 myprogram       # 数字模式（前缀 4 = SUID）

# 移除 SUID
chmod u-s myprogram
chmod 0755 myprogram       # 前缀 0 清除特殊位
```

> [!warning] SUID 安全风险
> SUID root 程序是系统安全的重点攻击面。如果 SUID root 程序存在漏洞（如缓冲区溢出），攻击者可以获取 root shell。现代系统上 SUID root 程序数量被严格控制在最低限度。编写 SUID 程序时务必遵守安全编码规范。

**SUID 的权限显示变化**：

- `rwsr-xr-x`：SUID + 所有者有执行权限（`s` 小写）
- `rwSr-xr-x`：SUID + 所有者无执行权限（`S` 大写），表示 SUID 存在但实际可能无法生效

### SGID (Set Group ID) — 双面用途

SGID 有两种不同的含义，取决于它设置在**文件**还是**目录**上。

**1. 设置在可执行文件上**：与 SUID 类似，运行时进程的有效 GID 变为文件的组所有者 GID。

```bash
$ ls -l /usr/bin/wall
-rwxr-sr-x 1 root tty 19024 Dec 14 11:45 /usr/bin/wall
#       ↑
#     s = SGID + x（组权限位出现 s）
```

**2. 设置在目录上**：该目录中新建的文件自动继承目录的**组**，而非创建者的主组。这对共享工作目录非常有用。

```bash
# 创建共享项目目录
mkdir /shared/project
chgrp developers /shared/project
chmod g+s /shared/project         # 设置 SGID
chmod 2775 /shared/project        # 数字模式（前缀 2 = SGID）
```

设置后，无论谁在该目录中创建文件，文件的组都自动设为 `developers`，确保团队成员都能访问。

```bash
# SGID 设置
chmod g+s myfile          # 符号模式
chmod 2755 myfile         # 数字模式

# SGID 移除
chmod g-s myfile
```

### 粘滞位 (Sticky Bit) — `/tmp` 的守护者

粘滞位**只对目录有意义**。设置了粘滞位的目录中，文件只能被以下用户删除或重命名：

- 文件的所有者
- 目录的所有者
- root

**典型例子：`/tmp`**

```bash
$ ls -ld /tmp
drwxrwxrwt 17 root root 4096 Dec 15 11:45 /tmp
#           ↑
#         t = 粘滞位 + x（其他用户位出现 t）
```

`/tmp` 对所有用户可写（`rwx`），任何人都可以在其中创建临时文件。粘滞位确保：Alice 可以在 `/tmp` 中创建文件，但 Bob 不能删除 Alice 的文件——即使 Bob 对 `/tmp` 目录有写权限。这一机制防止了一个用户破坏其他用户的临时文件。

```bash
# 设置粘滞位
chmod +t shared_dir        # 符号模式
chmod 1775 shared_dir      # 数字模式（前缀 1 = 粘滞位）

# 移除
chmod -t shared_dir
```

**粘滞位的权限显示变化**：

- `rwxrwxrwt`：粘滞位 + 其他人有执行权限（`t` 小写）
- `rwxrwxrwT`：粘滞位 + 其他人无执行权限（`T` 大写）

### 特殊权限位速查

| 位 | 八进制前缀 | 符号 | 适用对象 | 效果 |
|----|-----------|------|---------|------|
| SUID | 4 | `u+s` | 可执行文件 | 进程以文件所有者身份运行 |
| SGID | 2 | `g+s` | 文件 | 进程以文件组身份运行 |
| SGID | 2 | `g+s` | 目录 | 新建文件继承目录的组 |
| 粘滞位 | 1 | `+t` | 目录 | 只有所有者/root 可删除文件 |

综合示例：

```bash
chmod 4755 program    # SUID + rwxr-xr-x
chmod 2755 shared/    # SGID + rwxr-xr-x（目录：新文件继承组）
chmod 1777 /mytmp/    # 粘滞位 + rwxrwxrwx（仿 /tmp 行为）
```

## 进程权限：RUID、EUID、saved-UID

进程的权限身份不是单一的，而是由三个 UID 共同决定：

| UID | 全称 | 含义 | 决定什么 |
|-----|------|------|---------|
| **RUID** | Real User ID | 启动进程的用户的 UID | 标识"谁真正启动了这个进程" |
| **EUID** | Effective User ID | 进程当前用于权限检查的 UID | 决定进程能访问哪些文件 |
| **saved-UID** | Saved Set-User-ID | SUID 程序的原始有效 UID 备份 | 允许进程在 EUID 和 RUID 之间切换 |

### 普通进程：RUID = EUID

当 Alice（UID 1000）运行 `touch` 命令时：

```
RUID = 1000 (Alice)
EUID = 1000 (Alice)     ← 权限检查用，创建的文件归 Alice
saved-UID = 1000
```

### SUID 进程：EUID ≠ RUID

当 Alice 运行 `passwd`（SUID root 程序）时：

```
RUID = 1000 (Alice)     ← 系统知道是 Alice 在操作
EUID = 0 (root)         ← 进程实际以 root 权限运行，可以写 /etc/shadow
saved-UID = 0           ← 备份了 root 的 UID
```

`passwd` 程序利用这个双重身份：
- **EUID 0 用于写 `/etc/shadow`**（只有 root 可写）
- **RUID 1000 用于限制只能修改自己的密码**（不是 UID 1000 的用户密码不可改）

### 权限切换机制

进程可以通过 `seteuid()`/`setreuid()` 系统调用在 EUID 和 saved-UID/RUID 之间切换：

```
EUID = 0 (root) → seteuid(RUID) → EUID = 1000 (降权)
EUID = 1000    → seteuid(saved-UID) → EUID = 0 (恢复 root)
```

> [!tip] 最小特权原则的 OS 级实现
> SUID 程序的最佳实践：在最开始用 root 权限做必要操作，然后立即降权到 RUID，之后只凭普通用户权限运行。这样即使程序有漏洞，攻击者可利用的时间窗口也极小。

## 用户与组基础设施

权限体系依赖三份系统文件来将用户名/组名映射到数字 ID：

### /etc/passwd — 用户列表

```bash
root:x:0:0:root:/root:/bin/bash
# ⑦  ⑦ ⑦ ⑦ ⑦  ⑦   ⑦
# │   │ │ │ │  │    └── 默认 Shell
# │   │ │ │ │  └─────── 主目录
# │   │ │ │ └────────── GECOS（全名/注释）
# │   │ │ └──────────── GID（主组 ID）
# │   │ └────────────── UID（用户 ID，root=0）
# │   └──────────────── 密码占位符（x=密码在 /etc/shadow）
# └──────────────────── 用户名
```

文件对所有用户可读（只有通过它，系统才能将数字 UID 翻译为人名）。密码字段为 `x` 表示实际密码哈希在 `/etc/shadow`。

### /etc/shadow — 密码与策略

```bash
$ sudo cat /etc/shadow
root:$6$...加密哈希...:15000:0:99999:7:::
#   ⑦               ⑦  ⑦  ⑦   ⑦⑦⑦
#   │               │  │  │   │││└── 保留
#   │               │  │  │   ││└─── 账户过期日期（自 epoch 天数）
#   │               │  │  │   │└──── 密码过期后禁用天数
#   │               │  │  │   └───── 密码过期前警告天数
#   │               │  │  └──────── 密码最大有效天数
#   │               │  └─────────── 密码最小更改间隔天数
#   │               └────────────── 上次密码更改日期（自 epoch 天数）
#   └────────────────────────────── 加密密码哈希
```

> [!warning] /etc/shadow 是安全核心
> 该文件只有 root 可读（`-rw-r----- 1 root shadow`）。如果非特权用户能读取密码哈希，就能离线暴力破解密码。`passwd` 命令能修改它是因为它是 SUID root 程序。

### /etc/group — 组定义

```bash
root:*:0:pete,alice
#   ⑦ ⑦ ⑦
#   │ │ └── 组成员列表（逗号分隔）
#   │ └─── GID（组 ID）
#   └──── 组密码占位符（遗留，极少使用）
```

## 权限检查优先级

当进程试图访问文件时，内核按以下顺序检查权限：

1. **进程的 EUID == 文件的 UID？** → 使用所有者权限位（第一组 rwx）
2. **进程的 EUID 属于文件的组成员？** → 使用组权限位（第二组 rwx）
3. **以上都不匹配** → 使用其他用户权限位（第三组 rwx）

> [!warning] 注意
> 权限检查一旦匹配就**立即停止**。如果 Alice 是文件所有者（匹配第 1 步），即使她的组有更宽松的权限，也只使用所有者的权限。这意味着：如果所有者权限是 `---`（不可读写执行），即使组权限是 `rwx`，所有者本人也无法访问该文件。

root 用户（UID 0）**绕过所有权限检查**——这是一个有意设计的例外，而非漏洞。这也是 `sudo` 工作的基础。

## 跨领域连接

### → Rust：权限的安全封装

Rust 的标准库将 Linux 权限模型封装为跨平台类型：

- **`std::fs::Permissions`** — 封装 rwx 位。在 Unix 上内部是 `mode_t`（16 位权限），提供 `readonly()` 和 `set_readonly()` 方法。
- **`std::fs::set_permissions()`** — 底层调用 `chmod(2)` 系统调用。
- **`std::os::unix::fs::PermissionsExt`** — Unix 扩展 trait，暴露 `mode()` / `set_mode()` 来直接操作原始八进制权限位，包括 SUID/SGID/粘滞位。

```rust
use std::os::unix::fs::PermissionsExt;
let mut perms = std::fs::metadata("script.sh")?.permissions();
perms.set_mode(0o755);   // rwxr-xr-x
std::fs::set_permissions("script.sh", perms)?;
```

更深的层面上，Linux 的 **DAC（自主访问控制）**和 Rust 的**类型系统**在哲学上有共鸣——两者都在"赋能的同时约束"：
- DAC：文件所有者自主决定谁可以访问，但不能越权访问别人的文件。
- Rust 所有权：代码自由使用自己拥有的数据，但不能在未经生命周期/借用检查的前提下访问别人的数据。
- **Capability-based security**（现代安全模型的趋势）——将最小权限封装为不可伪造的 token——在理念上与 Rust 的 `&T`（不可变的借用就是"读能力"）和 `&mut T`（可变借用就是"写能力"）完全同构。两者都可以追溯到同一个根本思想：**权限应当是显式的、可传递的、最小化的**。

### → 软件工程：DAC 是 RBAC 的原型

Linux DAC 模型在软件工程中对应的是**访问控制**的设计模式：

| DAC 概念 | Web 应用 RBAC 对应 | 说明 |
|---------|-------------------|------|
| 文件所有者 | 资源 Owner | 谁创建了资源，谁有权设置其权限 |
| 组 | 角色 (Role) | 将多个用户归为一个权限组 |
| rwx 三位 | CRUD 操作 | 读=GET，写=POST/PUT/PATCH，执行=调用/触发 |
| SUID | 权限提升模式 (sudo, setuid) | 临时以更高权限执行受限操作 |
| 粘滞位 | 多用户共享空间的攻击防护 | 防止用户 A 删除用户 B 的数据 |
| umask | 默认权限策略 | 新资源的默认 ACL 配置 |
| root 绕过检查 | 超级管理员 (Super Admin) | 系统管理账户忽略常规权限限制 |

SUID 的"进程以文件所有者身份运行"是一种**特权分离模式**——将需要高权限的操作集中到少数经过审查的程序中，普通用户通过调用这些程序获得临时高权限，而不是直接持有高权限。这种模式在微服务架构中被广泛继承：限流服务、认证服务等基础设施以更高的服务间权限运行，普通业务服务通过网络调用它们获取授权。

### → Cocos Creator：资源访问控制模式

Cocos Creator 的资源管理系统虽然没有文件级权限，但借鉴了类似的访问控制思想：

- **Bundle 级别隔离** ≈ 目录+组权限：每个 bundle 是一个独立的资源命名空间，不同 bundle 的资源互不可见（类似不同组的目录）
- **远程资源加载** ≈ 外部用户权限：远程 bundle 的资源需要通过网络请求获取，类似于"其他用户"需要正确的 API 密钥（类比读权限）才能访问
- **编辑器中的资源引用** ≈ 文件所有者权限：在编辑器中拖拽建立的引用关系通过 UUID 绑定，相当于建立了一条"所有者→资源"的访问链
- **预制体中的嵌套引用** ≈ 目录的 x 权限：预制体内部可以引用其子资源（就像进入目录需要 x），但外部不能直接访问预制体内嵌套的资源

## 相关页面

- [[Linux 概述]] — Linux 知识体系的入口枢纽，本页面属于其「权限与用户」域
- [[Linux 文件系统]] — inode 中的 UID/GID/权限位字段，权限元数据的存储位置
- [[Linux 进程模型]] — 进程的 RUID/EUID 凭证如何从 fork/exec 继承
- [[Linux 概述]] — Linux 知识体系的入口枢纽，本页面属于其「权限与用户」域
- [[软件工程概述]] — DAC → RBAC 的访问控制演进、SUID 的特权提升模式在架构中的映射
- [[Rust 概述]] — Rust 所有权模型与 DAC 在"能力封装"层面的概念共鸣
- [[Cocos Creator 概述]] — 资源系统的访问控制模式借鉴

## 原始来源

- [文件权限](raw/linuxjourney/lessons/zh/permissions/file-permissions.md) — rwx 三层三位结构、`ls -l` 权限字符串解读
- [修改权限](raw/linuxjourney/lessons/zh/permissions/modifying-permissions.md) — chmod 符号模式与数字模式
- [所有权权限](raw/linuxjourney/lessons/zh/permissions/ownership-permissions.md) — chown、chgrp 修改文件所有者与组
- [umask](raw/linuxjourney/lessons/zh/permissions/umask.md) — 默认权限掩码机制
- [SUID](raw/linuxjourney/lessons/zh/permissions/setuid-set-user-id.md) — Set User ID 特殊权限位
- [SGID](raw/linuxjourney/lessons/zh/permissions/setgid-set-group-id.md) — Set Group ID 特殊权限位
- [粘滞位](raw/linuxjourney/lessons/zh/permissions/sticky-bit.md) — Sticky Bit 与 /tmp 的保护
- [进程权限](raw/linuxjourney/lessons/zh/permissions/process-permissions.md) — RUID、EUID、saved-UID
- [/etc/passwd](raw/linuxjourney/lessons/zh/user-management/etc-passwd-file.md) — 用户账户文件
- [/etc/shadow](raw/linuxjourney/lessons/zh/user-management/etc-shadow-file.md) — 加密密码与策略文件
- [/etc/group](raw/linuxjourney/lessons/zh/user-management/etc-group-file.md) — 组定义文件
- [用户和组](raw/linuxjourney/lessons/zh/user-management/users-and-groups.md) — UID/GID 基本概念与 sudo 提权
