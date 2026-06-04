#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Day 04 - EVE-NG + Netmiko SSH 测试脚本
环境: CentOS 7.6 + Python 3.6 + Netmiko 3.4.0
目标: 通过 SSH 连接 EVE-NG 中的 Cisco IOL Switch 并执行命令
"""

from netmiko import ConnectHandler

# 设备连接信息
cisco_switch = {
    'device_type': 'cisco_ios',      # Cisco IOS 设备类型
    'ip': '192.168.253.201',         # Switch 管理 IP
    'username': 'admin',             # 本地用户名
    'password': 'admin123',          # 密码
    'secret': 'admin123',            # enable 密码（privilege 15 时可选）
    'port': 22,                      # SSH 端口
    'timeout': 10,
}

def main():
    try:
        print("正在连接 192.168.253.201 ...")
        net_connect = ConnectHandler(**cisco_switch)
        
        # 发送 show 命令
        output = net_connect.send_command('show ip interface brief')
        print("\n=== 命令回显 ===")
        print(output)
        
        # 优雅断开
        net_connect.disconnect()
        print("\n✅ 连接成功，已断开")
        
    except Exception as e:
        print(f"\n❌ 连接失败：{e}")

if __name__ == '__main__':
    main()
