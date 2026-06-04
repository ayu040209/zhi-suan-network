! Cisco IOL Switch SSH 基础配置
! EVE-NG 实验: test-ssh
! 管理 IP: 192.168.253.201/24

enable
configure terminal

hostname SW1

! 配置域名（生成 RSA 密钥必须）
ip domain-name test.local

! 生成 2048 位 RSA 密钥
crypto key generate rsa modulus 2048

! 创建本地用户，privilege 15 直接进特权模式
username admin privilege 15 secret admin123

! 强制 SSHv2
ip ssh version 2

! VTY 线路只允许 SSH，使用本地认证
line vty 0 4
 transport input ssh
 login local
 exit

! 配置管理 IP（VLAN 1）
interface vlan 1
 ip address 192.168.253.201 255.255.255.0
 no shutdown
 exit

! 默认网关（同网段可选，建议加上）
ip default-gateway 192.168.253.2

! 保存配置
end
write memory

! === 验证命令 ===
show ip interface brief
show ssh
show running-config | include username
