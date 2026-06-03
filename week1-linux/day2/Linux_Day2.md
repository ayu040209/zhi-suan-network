# 🐧 Linux 学习日志 · Day 2

> **学习者**：ayu（网工转Linux）  
> **日期**：2026-06-02  
> **教材**：《鸟哥的Linux私房菜》第四版 Ch5（文件权限）  
> **环境**：VMware + CentOS 7.6 (1908)，4G内存，2核CPU  
> **角色**：一位老牌运维师傅带徒弟学Linux，坚信"命令行是网工的第二把扳手"

---

## 📌 今日学习目标

- [x] 搞定 `man` / `--help` / `cman` 帮助系统（含中文乱码踩坑）
- [x] 看懂 `ls -l` 输出，理解 `rwx` 本质
- [x] 掌握 `chmod` 数字法与字母法
- [x] 理解**目录的 `x` 权限**与**文件的 `x` 权限**完全不同
- [x] 掌握 `chown` 改归属
- [x] SUID 混脸熟（`passwd` 命令）

---

## 🛠️ 环境信息

```bash
# 系统版本
cat /etc/redhat-release
# CentOS Linux release 7.6.1810 (Core)

# 当前用户（普通用户 & root 切换练习）
whoami
# ayu / root

# 家目录
pwd
# /home/ayu
```

---

## 📖 Part 1：帮助系统踩坑全记录

### 1.1 `man` 页结构（网工类比版）

| 段落 | 网工翻译 | 含义 |
|------|----------|------|
| `NAME` | 命令名称 | 一句话介绍 |
| `SYNOPSIS` | 命令模板 | 语法格式，`[]` 可选，`<>` 必填，`|` 二选一 |
| `DESCRIPTION` | 功能描述 | 这个命令干嘛的 |
| `OPTIONS` | 可选参数 | `-a`、`-l` 这些开关 |
| `EXAMPLES` | 配置举例 | **不一定有！** 没有就换 `--help` 或鸟哥的书 |
| `SEE ALSO` | 相关命令 | 看完这个还可以看别的 |

### 1.2 `man` 页按键操作

```bash
man ls          # 进入手册
# 上下键 / Enter   → 一行一行翻
# Space           → 一页一页翻（最常用）
# b               → 往回翻一页
# /keyword        → 搜索，如 /SYNOPSIS
# n               → 下一个匹配
# N               → 上一个匹配
# q               → 退出（quit）
```

### 1.3 `--help` vs `man` 选择策略

| 场景 | 推荐 | 原因 |
|------|------|------|
| 快速回忆参数 | `--help` | 短，一页完事 |
| 深入学习/查返回值 | `man` | 完整，像字典 |
| 中文环境 | `cman` | 需要额外安装 |

### 1.4 中文 `man` 安装与配置（踩坑重灾区）

```bash
# 安装中文man包（需要root）
su -
yum install -y man-pages-zh-CN
# 输出：Package man-pages-zh-CN-1.5.2-4.el7.noarch already installed...

# 给普通用户配置 alias
echo "alias cman='man -M /usr/share/man/zh_CN'" >> ~/.bashrc
source ~/.bashrc

# 给root也配一份（切到root后家目录变了！）
su -
echo "alias cman='man -M /usr/share/man/zh_CN'" >> ~/.bashrc
source ~/.bashrc
```

**⚠️ 坑点 1：引号陷阱**
```bash
# 错误！双引号把 ~ 当成字面量
source "~/.bashrc"
# bash: /home/ayu/.bashrc: No such file or directory

# 正确写法
source ~/.bashrc
# 或写绝对路径
source /home/ayu/.bashrc
```

**⚠️ 坑点 2：用户隔离**
```bash
# alias 写在 /home/ayu/.bashrc 里
# 切到 root 后，root 读的是 /root/.bashrc，不认识 cman！
# 每个用户有自己的家目录和配置文件，像每台交换机有自己的配置
```

**⚠️ 坑点 3：中文乱码（VMware终端）**
```bash
# 尝试修复
export LANG=zh_CN.UTF-8
export LC_ALL=zh_CN.UTF-8
cman ls

# 如果 VMware 虚拟终端还是白色方块 → 是终端字体渲染问题
# 解决方案：用 Xshell/PuTTY/MobaXterm SSH 连接虚拟机
# 或：暂时放弃 cman，用 man + 鸟哥实体书（中文）对照
```

---

## 📂 Part 2：Ch5 文件权限 —— 五个通关实验

### 网工类比（核心思想）

> Linux 文件权限就像交换机 ACL：
> - `Owner` = 管理 VLAN（你自己）
> - `Group` = 业务 VLAN（同组同事）
> - `Other` = 访客 VLAN（其他人）
> - `rwx` = permit/deny 的细化版

### 实验 1：`ls -l` 拆解训练

```bash
# 命令
ls -l /etc/passwd
ls -l /bin/ls
ls -ld /tmp

# 参数
#   -l  → 长格式（权限、归属、大小、时间）
#   -d  → 只看目录本身，不看里面内容

# 输出样例
# -rw-r--r-- 1 root root 2875 Jun 2 10:00 /etc/passwd
#  ↑ 文件类型  ↑Owner  ↑Group  ↑Other
#  - = 普通文件，d = 目录，l = 链接
```

**验证标准**：看到 `-rw-r--r--` 能立刻说出：普通文件，owner 读写，group 只读，other 只读。

---

### 实验 2：`chmod` 数字法与字母法

```bash
# ========== 准备 ==========
cd ~
mkdir ch5_lab && cd ch5_lab
echo -e '#!/bin/bash
echo "Interface UP"' > ifcheck.sh
ls -l ifcheck.sh
# 预期：-rw-rw-r-- （664，没有执行权）

# ========== 数字法 ==========
chmod 755 ifcheck.sh
ls -l ifcheck.sh
# 预期：-rwxr-xr-x

# 数字拆解
# 7 = 4+2+1 = rwx  → Owner：读+写+执行
# 5 = 4+0+1 = r-x  → Group：读+执行（不能写，防误删）
# 5 = 4+0+1 = r-x  → Other：读+执行

# ========== 字母法 ==========
chmod u+x,g-w,o=r ifcheck.sh
# u+x → 给 Owner 加执行
# g-w → 给 Group 去掉写
# o=r → 给 Other 只保留读

# 常见报错
# chmod: changing permissions of 'xxx': Operation not permitted
# 原因：你不是 Owner，也不是 root
# 解决：sudo chmod 755 xxx  或  su - 切 root
```

---

### 实验 3：目录的 `x` 权限（网工最容易掉的坑！）

```bash
# ========== root 端 ==========
su -
mkdir /test_room
chmod 644 /test_room      # 去掉 x，保留 r
ls -ld /test_room
# 预期：drw-r--r-- （注意！owner 位没有 x！）
exit

# ========== 普通用户端 ==========
cd /test_room
# 预期报错：-bash: cd: /test_room: Permission denied

# 网工类比
# 目录的 r = 你有楼道里的房间清单（能 ls 看到文件名）
# 目录的 x = 你有楼道的钥匙（能 cd 进去）
# 有清单没钥匙 = 站在门外干瞪眼
```

**⚠️ 坑点：相对路径 vs 绝对路径**
```bash
# 错误：在当前目录找 test_room，但它在根目录
ls -l test_room
# ls: cannot access test_room: No such file or directory

# 正确：绝对路径，从根开始
ls -ld /test_room

# 网工类比
# test_room     → 相对路径，像 "去 test_room"，从当前位置出发
# /test_room    → 绝对路径，像 "去 /test_room"，从根目录出发，不管你在哪
```

---

### 实验 4：`chown` 改归属（像改接口 VLAN 归属）

```bash
# root 创建系统文件
su -
touch /tmp/netconfig.txt
ls -l /tmp/netconfig.txt
# 预期：owner=root, group=root

# 把文件划归给 ayu
chown ayu:ayu /tmp/netconfig.txt
ls -l /tmp/netconfig.txt
# 预期：owner=ayu, group=ayu

# 命令拆解
# chown 新Owner:新Group 文件
# chown ayu file      → 只改 Owner
# chown :ayu file     → 只改 Group
# chgrp ayu file      → 只改 Group（另一种写法）
exit

# 常见报错
# chown: changing ownership of 'xxx': Operation not permitted
# 原因：普通用户不能随意把文件丢给别人，只有 root 能自由改归属
```

---

### 实验 5：SUID 初体验（预习，混脸熟）

```bash
# 查看 passwd 命令的权限
ls -l /usr/bin/passwd
# 预期：-rwsr-xr-x （注意 owner 的 x 变成了 s！）

# 网工类比
# SUID 就像交换机上的 "enable" 提权
# /etc/shadow 是 root 专属文件（像 running-config）
# 普通用户能改自己密码，就是因为 passwd 命令有 SUID
# 执行时临时获得 root 权限去修改 shadow

# 鸟哥 Ch5 后半段会细讲，今晚只混脸熟
```

---

## 🐛 今日踩坑合集（按时间顺序）

| 序号 | 踩的坑 | 原因 | 解决 |
|------|--------|------|------|
| 1 | `source "~/.bashrc"` 报错 | 双引号把 `~` 当成字面量 | 去掉引号：`source ~/.bashrc` |
| 2 | root 下 `cman` 不认识 | alias 配在 ayu 用户，root 读的是 `/root/.bashrc` | root 单独配一份 alias |
| 3 | 中文 man 白色方块 | VMware 虚拟终端对 UTF-8 支持不好 | 用 SSH 客户端，或暂时用英文 man |
| 4 | `ls -l test_room` 报错 | 相对路径 vs 绝对路径混淆 | 系统目录用绝对路径 `/test_room` |
| 5 | `ls -l /test_room` 显示 `total 0` | `ls -l` 看的是目录**里面**的文件 | 看目录本身用 `ls -ld` |
| 6 | `mkdir test_room` vs `mkdir /test_room` | 不加 `/` 在当前目录建，加了在根目录建 | 系统级实验用绝对路径 |

---

## 📝 师傅的今日命令卡片（示例）

```text
【命令】chmod 755 file.sh
【作用】修改文件权限
【参数】
  755：数字权限
    7 = 4+2+1 = rwx（Owner：读+写+执行）
    5 = 4+0+1 = r-x（Group：读+执行，不能写）
    5 = 4+0+1 = r-x（Other：读+执行，不能写）
【场景】给脚本加执行权限，同时防止被误修改
【常见报错】
  Operation not permitted → 不是 Owner，用 sudo 或切 root
【验证】ls -l file.sh 看是否变成 -rwxr-xr-x
【网工类比】像给不同 VLAN 配 ACL，Owner 是管理 VLAN，Other 是访客 VLAN
```

---

## ✅ Day 2 通关标准（师傅定的）

- [x] `ls -l` 看到 `-rwxr-xr-x` 能立刻说出每个字母代表什么
- [x] `chmod 755` 和 `chmod u+x` 能熟练切换使用
- [x] 理解**目录的 x 权限**和**文件的 x 权限**是两回事
- [x] 遇到 `Permission denied` 知道先 `whoami`、`pwd`、`ls -l` 三板斧
- [x] `man` 进去能按 `q` 退出，会 `/` 搜索
- [x] 写好了第一张命令卡片

---

## 🌅 明日预告（Day 3）

- 鸟哥 Ch6：文件与目录管理（`cp`、`mv`、`rm`、`cat`、`more`、`less`）
- 鸟哥 Ch7：磁盘管理基础（`df`、`du`、`fdisk` 初体验）
- 继续巩固 `man` / `--help` 使用
- 尝试用 SSH 客户端连接虚拟机，解决中文乱码

---

## 🧔 师傅寄语

> 徒弟，你今天从 `man` 页的白色方块，到 `chmod` 的数字字母双修法，再到目录 `x` 权限的"有清单没钥匙"——每一个坑都是以后排障时的肌肉记忆。
>
> 网工学 Linux，不要背命令大全，要像调交换机一样：**理解它在网络里干什么，然后敲10遍。**
>
> 明天继续，不赶进度，地基打牢。
>
> **徒弟还有哪里卡住的，随时发命令截图或报错信息。师傅在。**

---

*Generated by 运维师傅 · 2026-06-02*
