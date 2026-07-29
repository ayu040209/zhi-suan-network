# HCIA - IPv6基础

## 一、核心概念速记

| 概念 | 一句话定义 | 面试口述话术 |
|------|-----------|-------------|
| IPv6 | 互联网协议第6版，使用128bit地址替代IPv4的32bit | "面试官你好，IPv6是下一代互联网协议，地址长度从IPv4的32位扩展到128位，地址空间近乎无限，还简化了报文头部、支持即插即用、内置安全特性。" |
| GUA | 全球单播地址，相当于IPv4的公网地址 | "全球单播地址是全球唯一可路由的IPv6地址，前缀2000::/3，相当于IPv4的公网地址。" |
| ULA | 唯一本地地址，相当于IPv4的私网地址 | "唯一本地地址是IPv6的私网地址，前缀FC00::/7，目前使用FD00::/8，仅在本地网络内路由。" |
| LLA | 链路本地地址，仅在同一链路上有效 | "链路本地地址前缀FE80::/10，自动配置在每个IPv6接口上，用于邻居发现、链路层通信，不会跨路由器转发。" |
| SLAAC | 无状态地址自动配置，无需DHCP服务器 | "SLAAC是IPv6的亮点功能，主机通过路由器通告RA获取前缀，结合EUI-64生成的接口标识自动配置地址，即插即用。" |
| NDP | 邻居发现协议，替代IPv4的ARP | "NDP是IPv6的邻居发现协议，使用ICMPv6报文实现地址解析、重复地址检测、路由器发现等功能，替代了IPv4的ARP。" |
| EUI-64 | 将48bit MAC地址扩展为64bit接口标识的方法 | "EUI-64规范将48位MAC地址中间插入FFFE并翻转第7位，生成64位接口标识，用于自动配置IPv6地址。" |

## 二、华为命令配置模板
### 场景1：IPv6基本配置

```bash
# 步骤1：全局使能IPv6
[R1] ipv6

# 步骤2：配置接口全球单播地址（手工配置）
[R1] interface GigabitEthernet 0/0/0
[R1-GigabitEthernet0/0/0] ipv6 enable
[R1-GigabitEthernet0/0/0] ipv6 address 2001::1 64

# 步骤3：配置链路本地地址
[R1-GigabitEthernet0/0/0] ipv6 address auto link-local
# 或手工配置
[R1-GigabitEthernet0/0/0] ipv6 address fe80::1 link-local

# 验证命令
[R1] display ipv6 interface GigabitEthernet 0/0/0
[R1] display ipv6 neighbors
```

### 场景2：配置IPv6静态路由

```bash
# 配置IPv6静态路由
[R1] ipv6 route-static 2002:: 64 2001::2

# 配置默认路由
[R1] ipv6 route-static :: 0 2001::2

# 验证命令
[R1] display ipv6 routing-table
[R1] display ipv6 route-static
```

### 场景3：DHCPv6服务器配置

```bash
# 步骤1：使能DHCP
[R2] dhcp enable

# 步骤2：创建DHCPv6地址池
[R2] dhcpv6 pool pool1
[R2-dhcpv6-pool-pool1] address prefix 2002::/64
[R2-dhcpv6-pool-pool1] quit

# 步骤3：接口应用地址池
[R2] interface GigabitEthernet 0/0/0
[R2-GigabitEthernet0/0/0] dhcpv6 server pool1

# 验证命令
[R2] display dhcpv6 pool
```

### 场景4：无状态自动配置（SLAAC）

```bash
# 路由器侧：开启RA通告
[R2] interface GigabitEthernet 0/0/1
[R2-GigabitEthernet0/0/1] ipv6 address 2003::1 64
[R2-GigabitEthernet0/0/1] undo ipv6 nd ra halt  # 华为默认不发RA，必须手动开启

# 主机侧：配置自动获取
[R4] interface GigabitEthernet 0/0/0
[R4-GigabitEthernet0/0/0] ipv6 address auto global  # 无状态自动配置全球地址
[R4-GigabitEthernet0/0/0] ipv6 address auto link-local

# 验证命令
[R4] display ipv6 interface GigabitEthernet 0/0/0
```

### 场景5：完整小型IPv6网络配置

```bash
# R1配置（R1-R2互联）
[R1] ipv6
[R1] interface GigabitEthernet 0/0/0
[R1-GigabitEthernet0/0/0] ipv6 enable
[R1-GigabitEthernet0/0/0] ipv6 address auto link-local
[R1-GigabitEthernet0/0/0] ipv6 address 2001::1 64
[R1] ipv6 route-static 2002:: 15 2001::2  # 聚合路由

# R2配置（中心路由器）
[R2] ipv6
[R2] dhcp enable
[R2] dhcpv6 pool pool1
[R2-dhcpv6-pool-pool1] address prefix 2002::/64

[R2] interface GigabitEthernet 1/0/0
[R2-GigabitEthernet1/0/0] ipv6 address 2001::2 64

[R2] interface GigabitEthernet 0/0/0
[R2-GigabitEthernet0/0/0] ipv6 address 2002::1 64
[R2-GigabitEthernet0/0/0] dhcpv6 server pool1

[R2] interface GigabitEthernet 0/0/1
[R2-GigabitEthernet0/0/1] ipv6 address 2003::1 64
[R2-GigabitEthernet0/0/1] undo ipv6 nd ra halt

# R3配置（DHCPv6客户端）
[R3] ipv6
[R3] dhcp enable
[R3] interface GigabitEthernet 0/0/0
[R3-GigabitEthernet0/0/0] ipv6 address auto dhcp

# R4配置（SLAAC）
[R4] ipv6
[R4] interface GigabitEthernet 0/0/0
[R4-GigabitEthernet0/0/0] ipv6 address auto global
[R4] ipv6 route-static 2001:: 64 2003::1
[R4] ipv6 route-static 2002:: 64 2003::1

# R3配置默认路由
[R3] ipv6 route-static :: 0 2002::1
```

## 三、思科-华为迁移对照表

| 思科命令 | 华为命令 | 注意点 |
|----------|----------|--------|
| ipv6 unicast-routing | ipv6 | 华为全局使能ipv6即可 |
| interface f0/0; ipv6 address 2001::1/64 | interface g0/0/0; ipv6 address 2001::1 64 | 华为用空格不用斜杠 |
| ipv6 address FE80::1 link-local | ipv6 address fe80::1 link-local | 语法相同 |
| ipv6 address autoconfig | ipv6 address auto global | 华为分global和link-local |
| ipv6 route 2001::/64 2002::1 | ipv6 route-static 2001:: 64 2002::1 | 华为用route-static |
| ipv6 nd ra suppress | ipv6 nd ra halt | 华为默认抑制RA，用undo ipv6 nd ra halt开启 |
| show ipv6 interface brief | display ipv6 interface brief | 华为display前缀 |
| show ipv6 route | display ipv6 routing-table | 华为display前缀 |
| show ipv6 neighbors | display ipv6 neighbors | 语法类似 |

## 四、面试高频题

**Q: IPv6地址长度是多少？有多少个地址？**

A: IPv6地址长度128位，地址数量为2^128个（约3.4*10^38），号称可以为地球上的每一粒沙子分配一个地址。

**Q: IPv6为什么没有广播地址？用什么替代？**

A: IPv6取消了广播地址，所有广播场景用组播替代。比如ARP被NDP（邻居发现协议）替代，使用被请求节点组播地址FF02::1:FFxx:xxxx。

**Q: EUI-64如何生成？**

A: 将48位MAC地址从中间分成两部分，中间插入FFFE，然后将第一个字节的第7位（U/L位）取反，就得到了64位的接口标识。

**Q: SLAAC和DHCPv6的区别？**

A: SLAAC是无状态自动配置，主机从路由器RA获取前缀，自行生成地址，服务器不记录分配状态。DHCPv6是有状态配置，服务器分配完整地址并记录状态，还可下发DNS等其他参数。

**Q: IPv6报文头部相比IPv4有什么变化？**

A: 固定40字节基本头部（IPv4可变20-60字节）；删除了校验和、分片、可选字段；增加了流标签Flow Label；通过扩展头部实现可选功能，提高转发效率。

## 五、易错点/坑

- **坑1：华为默认不发RA** -- 必须undo ipv6 nd ra halt才能发送RA，否则下游设备无法SLAAC
- **坑2：ipv6 enable忘了配** -- 系统视图和接口视图都需要使能ipv6
- **坑3：链路本地地址和全球地址混淆** -- link-local地址fe80::/10只在本地链路有效，不能用于跨网段通信
- **坑4：DHCPv6和IPv4 DHCP命令不同** -- 华为DHCPv6使用dhcpv6 pool，和ip pool是两套命令
- **坑5：IPv6静态路由写法** -- 华为ipv6 route-static后面用空格分隔前缀和长度，不是斜杠
