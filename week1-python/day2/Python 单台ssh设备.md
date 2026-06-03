#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import paramiko

# ================== 第7-10行改成你的实际信息 ==================
ip = '192.168.159.128'       # 第7行：改成你刚才查到的 ens33 IP
username = 'root'            # 第8行：登录 CentOS 的用户名（如果你用普通用户就写普通用户名）
password = '你的密码'         # 第9行：改成该用户的登录密码
# ==========================================================

# 1. 创建 SSH 客户端（就像打开 SecureCRT 新建会话）
ssh = paramiko.SSHClient()

# 2. 自动接受未知主机密钥（第一次连会问"是否保存"，这里自动点"是"）
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

# 3. 建立连接
print(f">>> 正在连接 {ip} ...")
ssh.connect(ip, username=username, password=password, timeout=10)

# 4. 执行命令（就像你在 bash 里敲命令）
stdin, stdout, stderr = ssh.exec_command('ip addr')

# 5. 打印回显
output = stdout.read().decode('utf-8')
print("\n=== 设备回显 ===")
print(output)

# 6. 关闭连接
ssh.close()
print("\n>>> 连接已关闭")
