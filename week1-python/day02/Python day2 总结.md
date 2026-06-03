# Day 02 · Python 网络自动化入门

## 环境信息
- OS: CentOS Linux 7 (Core) 1908
- Python: 3.6.8
- 位置: `~/python_net/`

## 今日目标
用 Paramiko 通过 SSH 登录本机，执行 `ip addr` 并打印回显。

## 踩坑记录（CentOS 7 安装 Paramiko 全链路）

| 步骤 | 命令 | 现象 | 解决 |
|:---|:---|:---|:---|
| 1 | `pip3 install paramiko` | `ModuleNotFoundError: No module named 'paramiko'` | 确认没装 |
| 2 | `pip3 install paramiko` | 编译失败 `gcc failed` | `yum install -y gcc python3-devel` |
| 3 | 重装 | `setuptools_rust` 报错 | 新版 `bcrypt` 需 Rust，CentOS 7 不支持 |
| 4 | `pip3 install paramiko==2.7.2` | 单等号 `=` 语法错误 | 改为双等号 `==` |
| 5 | 重装 2.7.2 | 成功，但运行报 `No module named 'six'` | `pip3 install six` |
| 6 | 运行脚本 | `AttributeError: 'AutoAddPolicy'` | 待修复（Day 03 解决） |

## 学到的 Linux 命令
- `cd`, `cd ..`, `cd -`, `pwd` —— 目录切换
- `mkdir -p`, `ls`, `cat` —— 文件操作
- `vi` + `:%s/旧/新/g` —— 查找替换
- `yum install`, `pip3 install` —— 包管理

## 学到的 Python 概念
- `import paramiko` —— 导入 SSH 库
- `SSHClient()` —— 创建 SSH 会话（类比 SecureCRT 新建连接）
- `exec_command()` —— 发送命令，返回 `(stdin, stdout, stderr)`
- `stdout.read().decode()` —— 二进制回显转人类可读字符串

## 脚本文件
- 单台设备 SSH 实验脚本
