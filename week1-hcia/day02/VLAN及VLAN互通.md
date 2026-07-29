# HCIA - VLAN原理与配置 & VLAN间通信

## 一、核心概念速记
| 概念 | 一句话定义
|------|-----------|-------------|
| VLAN | 虚拟局域网，将一个物理广播域逻辑划分为多个广播域 | "VLAN通过802.1Q标签隔离广播域，同一VLAN内二层互通，不同VLAN二层隔离，需三层设备才能互通。" |
| 802.1Q Tag | 4字节VLAN标签，插入以太网帧中用于标识所属VLAN | "Tag包含TPID(0x8100)和12位VLAN ID，支持1-4094，交换机通过Tag识别帧属于哪个VLAN。" |
| PVID | 接口缺省VLAN，Port VLAN ID | "Access口的PVID就是所属VLAN；Trunk/Hybrid收到Untagged帧会打上PVID的Tag，默认PVID是1。" |
| Access口 | 仅允许一个VLAN通过，连接终端设备 | "Access口收Untagged帧打PVID标签，发帧时若VLAN ID等于PVID则剥标签；连接PC、服务器等终端。" |
| Trunk口 | 允许多个VLAN带标签通过，连接交换机或路由器 | "Trunk口收发Tagged帧，PVID相同的VLAN发出去剥标签，其他VLAN保留标签；用于交换机互联或接路由器子接口。" |
| Hybrid口 | 华为私有接口类型，收发灵活可控 | "Hybrid口可配置哪些VLAN以Tagged方式通过、哪些以Untagged方式通过，既能接交换机也能接终端，是华为特色。" |
| VLANIF | 三层逻辑接口，编号与VLAN ID对应，实现VLAN间路由 | "VLANIF是三层交换机上的逻辑接口，作为VLAN的网关，实现不同VLAN之间的三层转发。" |
| 子接口/单臂路由 | 在路由器一个物理接口上创建多个逻辑子接口，每个子接口终结一个VLAN | "单臂路由通过一个物理口接交换机Trunk，子接口终结不同VLAN Tag，为每个VLAN提供网关，节省路由器接口。" |
| 三层交换机 | 同时具备二层交换和三层路由功能的交换机 | "三层交换机内部有交换模块和路由模块，通过VLANIF实现VLAN间高速转发，转发性能远高于路由器。" |

## 二、华为命令配置模板
### 场景1：VLAN基础配置（基于接口划分）
```bash
# 创建VLAN
[SW1] vlan batch 10 20 100

# 配置Access口接PC
[SW1] interface GigabitEthernet 0/0/1
[SW1-GigabitEthernet0/0/1] port link-type access
[SW1-GigabitEthernet0/0/1] port default vlan 10

# 配置Trunk口互联交换机
[SW1] interface GigabitEthernet 0/0/24
[SW1-GigabitEthernet0/0/24] port link-type trunk
[SW1-GigabitEthernet0/0/24] port trunk pvid vlan 1
[SW1-GigabitEthernet0/0/24] port trunk allow-pass vlan 10 20 100
# 注意：华为Trunk默认只允许VLAN1通过，必须手动allow-pass

# 验证
[SW1] display vlan
[SW1] display port vlan

### 场景2：Hybrid接口典型配置（终端+服务器共享场景）

# 需求：VLAN10和VLAN20的PC都能访问VLAN100的服务器，但PC之间二层隔离
[SW1] vlan batch 10 20 100

# 接PC1（VLAN10）
[SW1] interface GigabitEthernet 0/0/1
[SW1-GigabitEthernet0/0/1] port link-type hybrid
[SW1-GigabitEthernet0/0/1] port hybrid pvid vlan 10
[SW1-GigabitEthernet0/0/1] port hybrid untagged vlan 10 100

# 接PC2（VLAN20）
[SW1] interface GigabitEthernet 0/0/2
[SW1-GigabitEthernet0/0/2] port link-type hybrid
[SW1-GigabitEthernet0/0/2] port hybrid pvid vlan 20
[SW1-GigabitEthernet0/0/2] port hybrid untagged vlan 20 100

# 接服务器（VLAN100）
[SW1] interface GigabitEthernet 0/0/3
[SW1-GigabitEthernet0/0/3] port link-type hybrid
[SW1-GigabitEthernet0/0/3] port hybrid pvid vlan 100
[SW1-GigabitEthernet0/0/3] port hybrid untagged vlan 10 20 100

# 交换机互联口（保留Tag）
[SW1] interface GigabitEthernet 0/0/24
[SW1-GigabitEthernet0/0/24] port link-type hybrid
[SW1-GigabitEthernet0/0/24] port hybrid tagged vlan 10 20 100

### 场景3：基于MAC地址划分VLAN
[SW1] vlan 10
[SW1-vlan10] mac-vlan mac-address 001e-10dd-dd01
[SW1-vlan10] mac-vlan mac-address 001e-10dd-dd02
[SW1-vlan10] quit

[SW1] interface GigabitEthernet 0/0/2
[SW1-GigabitEthernet0/0/2] mac-vlan enable
[SW1-GigabitEthernet0/0/2] port link-type hybrid
[SW1-GigabitEthernet0/0/2] port hybrid tagged vlan 10

# 验证
[SW1] display mac-vlan mac-address all

VLAN配置与面试要点
Markdown
复制
代码
预览
# HCIA - VLAN原理与配置 & VLAN间通信

## 一、核心概念速记
| 概念 | 一句话定义 |
|------|-----------|-------------|
| VLAN | 虚拟局域网，将一个物理广播域逻辑划分为多个广播域 | "VLAN通过802.1Q标签隔离广播域，同一VLAN内二层互通，不同VLAN二层隔离，需三层设备才能互通。" |
| 802.1Q Tag | 4字节VLAN标签，插入以太网帧中用于标识所属VLAN | "Tag包含TPID(0x8100)和12位VLAN ID，支持1-4094，交换机通过Tag识别帧属于哪个VLAN。" |
| PVID | 接口缺省VLAN，Port VLAN ID | "Access口的PVID就是所属VLAN；Trunk/Hybrid收到Untagged帧会打上PVID的Tag，默认PVID是1。" |
| Access口 | 仅允许一个VLAN通过，连接终端设备 | "Access口收Untagged帧打PVID标签，发帧时若VLAN ID等于PVID则剥标签；连接PC、服务器等终端。" |
| Trunk口 | 允许多个VLAN带标签通过，连接交换机或路由器 | "Trunk口收发Tagged帧，PVID相同的VLAN发出去剥标签，其他VLAN保留标签；用于交换机互联或接路由器子接口。" |
| Hybrid口 | 华为私有接口类型，收发灵活可控 | "Hybrid口可配置哪些VLAN以Tagged方式通过、哪些以Untagged方式通过，既能接交换机也能接终端，是华为特色。" |
| VLANIF | 三层逻辑接口，编号与VLAN ID对应，实现VLAN间路由 | "VLANIF是三层交换机上的逻辑接口，作为VLAN的网关，实现不同VLAN之间的三层转发。" |
| 子接口/单臂路由 | 在路由器一个物理接口上创建多个逻辑子接口，每个子接口终结一个VLAN | "单臂路由通过一个物理口接交换机Trunk，子接口终结不同VLAN Tag，为每个VLAN提供网关，节省路由器接口。" |
| 三层交换机 | 同时具备二层交换和三层路由功能的交换机 | "三层交换机内部有交换模块和路由模块，通过VLANIF实现VLAN间高速转发，转发性能远高于路由器。" |

## 二、华为命令配置模板
### 场景1：VLAN基础配置（基于接口划分）
```bash
# 创建VLAN
[SW1] vlan batch 10 20 100

# 配置Access口接PC
[SW1] interface GigabitEthernet 0/0/1
[SW1-GigabitEthernet0/0/1] port link-type access
[SW1-GigabitEthernet0/0/1] port default vlan 10

# 配置Trunk口互联交换机
[SW1] interface GigabitEthernet 0/0/24
[SW1-GigabitEthernet0/0/24] port link-type trunk
[SW1-GigabitEthernet0/0/24] port trunk pvid vlan 1
[SW1-GigabitEthernet0/0/24] port trunk allow-pass vlan 10 20 100
# 注意：华为Trunk默认只允许VLAN1通过，必须手动allow-pass

# 验证
[SW1] display vlan
[SW1] display port vlan
场景2：Hybrid接口典型配置（终端+服务器共享场景）
bash
# 需求：VLAN10和VLAN20的PC都能访问VLAN100的服务器，但PC之间二层隔离
[SW1] vlan batch 10 20 100

# 接PC1（VLAN10）
[SW1] interface GigabitEthernet 0/0/1
[SW1-GigabitEthernet0/0/1] port link-type hybrid
[SW1-GigabitEthernet0/0/1] port hybrid pvid vlan 10
[SW1-GigabitEthernet0/0/1] port hybrid untagged vlan 10 100

# 接PC2（VLAN20）
[SW1] interface GigabitEthernet 0/0/2
[SW1-GigabitEthernet0/0/2] port link-type hybrid
[SW1-GigabitEthernet0/0/2] port hybrid pvid vlan 20
[SW1-GigabitEthernet0/0/2] port hybrid untagged vlan 20 100

# 接服务器（VLAN100）
[SW1] interface GigabitEthernet 0/0/3
[SW1-GigabitEthernet0/0/3] port link-type hybrid
[SW1-GigabitEthernet0/0/3] port hybrid pvid vlan 100
[SW1-GigabitEthernet0/0/3] port hybrid untagged vlan 10 20 100

# 交换机互联口（保留Tag）
[SW1] interface GigabitEthernet 0/0/24
[SW1-GigabitEthernet0/0/24] port link-type hybrid
[SW1-GigabitEthernet0/0/24] port hybrid tagged vlan 10 20 100
场景3：基于MAC地址划分VLAN
bash
[SW1] vlan 10
[SW1-vlan10] mac-vlan mac-address 001e-10dd-dd01
[SW1-vlan10] mac-vlan mac-address 001e-10dd-dd02
[SW1-vlan10] quit

[SW1] interface GigabitEthernet 0/0/2
[SW1-GigabitEthernet0/0/2] mac-vlan enable
[SW1-GigabitEthernet0/0/2] port link-type hybrid
[SW1-GigabitEthernet0/0/2] port hybrid tagged vlan 10

# 验证
[SW1] display mac-vlan mac-address all

###场景4：单臂路由（路由器子接口实现VLAN间通信）

# 交换机侧：接路由器的口配Trunk
[SW1] interface GigabitEthernet 0/0/24
[SW1-GigabitEthernet0/0/24] port link-type trunk
[SW1-GigabitEthernet0/0/24] port trunk allow-pass vlan 10 20

# 路由器侧：创建子接口终结VLAN
[R1] interface GigabitEthernet 0/0/1.10
[R1-GigabitEthernet0/0/1.10] dot1q termination vid 10
[R1-GigabitEthernet0/0/1.10] ip address 192.168.10.254 24
[R1-GigabitEthernet0/0/1.10] arp broadcast enable

[R1] interface GigabitEthernet 0/0/1.20
[R1-GigabitEthernet0/0/1.20] dot1q termination vid 20
[R1-GigabitEthernet0/0/1.20] ip address 192.168.20.254 24
[R1-GigabitEthernet0/0/1.20] arp broadcast enable
# arp broadcast enable 必须配，否则子接口无法主动发ARP请求场景4：单臂路由（路由器子接口实现VLAN间通信）

### 景5：三层交换机VLANIF实现VLAN间通信（推荐方案）

# 创建VLAN并划分接口
[SW1] vlan batch 10 20
[SW1] interface GigabitEthernet 0/0/1
[SW1-GigabitEthernet0/0/1] port link-type access
[SW1-GigabitEthernet0/0/1] port default vlan 10
[SW1] interface GigabitEthernet 0/0/2
[SW1-GigabitEthernet0/0/2] port link-type access
[SW1-GigabitEthernet0/0/2] port default vlan 20

# 配置VLANIF作为网关
[SW1] interface Vlanif 10
[SW1-Vlanif10] ip address 192.168.10.254 24
[SW1] interface Vlanif 20
[SW1-Vlanif20] ip address 192.168.20.254 24

# 验证
[SW1] display ip routing-table
[SW1] display interface Vlanif 10
[SW1] ping -a 192.168.10.254 192.168.20.254

### 场景6：三层交换机物理口切换为三层口 [补充]

# 华为三层交换机物理口默认是二层口，如需当路由口使用：
[SW1] interface GigabitEthernet 0/0/24
[SW1-GigabitEthernet0/0/24] undo portswitch
[SW1-GigabitEthernet0/0/24] ip address 192.168.30.1 24

### 场景7：三层交换机+路由器出口（内网访问Internet）
# SW2作为三层核心，VLANIF作网关，上行口接路由器
[SW2] vlan batch 10 20 30
[SW2] interface Vlanif 10
[SW2-Vlanif10] ip address 192.168.10.254 24
[SW2] interface Vlanif 20
[SW2-Vlanif20] ip address 192.168.20.254 24
[SW2] interface Vlanif 30
[SW2-Vlanif30] ip address 192.168.30.1 24

[SW2] interface GigabitEthernet 0/0/2
[SW2-GigabitEthernet0/0/2] port link-type access
[SW2-GigabitEthernet0/0/2] port default vlan 30

# 配置默认路由指向出口路由器
[SW2] ip route-static 0.0.0.0 0 192.168.30.2

# 路由器R1侧
[R1] interface GigabitEthernet 0/0/0
[R1-GigabitEthernet0/0/0] ip address 192.168.30.2 24
[R1] ip route-static 192.168.10.0 24 192.168.30.1
[R1] ip route-static 192.168.20.0 24 192.168.30.1

## 三、思科-华为迁移对照表
| 思科命令                                        | 华为命令                                           | 注意点                                |
| ------------------------------------------- | ---------------------------------------------- | ---------------------------------- |
| vlan 10                                     | vlan 10                                        | 思科直接进入，华为需quit再进接口；批量创建用vlan batch |
| switchport mode access                      | port link-type access                          | 华为需显式指定链路类型                        |
| switchport access vlan 10                   | port default vlan 10                           | 关键字差异                              |
| switchport mode trunk                       | port link-type trunk                           | 功能一致                               |
| switchport trunk allowed vlan 10,20         | port trunk allow-pass vlan 10 20               | 华为用空格分隔，无逗号                        |
| switchport trunk native vlan 10             | port trunk pvid vlan 10                        | 思科叫native vlan，华为叫PVID             |
| interface fa0/0.10 + encapsulation dot1Q 10 | interface g0/0/1.10 + dot1q termination vid 10 | 华为需额外配arp broadcast enable         |
| interface vlan 10                           | interface Vlanif 10                            | 华为是Vlanif，思科是vlan                  |
| no shutdown                                 | undo shutdown                                  | 三层交换机物理口undo portswitch后才能配IP      |
| show vlan brief                             | display vlan                                   | 华为显示更详细，含Tag/Untagged标记            |

## 四、面试高频题

Q: VLAN的作用是什么？
A: 隔离广播域、增强安全性、灵活组网。同一VLAN内二层互通，不同VLAN二层隔离。
Q: Access、Trunk、Hybrid三种接口的区别？
A: Access只属于一个VLAN，收发剥标签，接终端；Trunk允许多个VLAN带标签通过，PVID的VLAN发出去剥标签，接交换机或路由器；Hybrid是华为特有，可灵活配置每个VLAN是带标签还是不带标签通过，既能接终端也能接交换机。
Q: 为什么Trunk口要配置allow-pass？华为和思科有什么区别？
A: 华为Trunk口默认只允许VLAN1通过，必须手动allow-pass其他VLAN；思科Trunk口默认允许所有VLAN通过，需手动prune排除。这是华为和思科最大的行为差异之一。
Q: 实现VLAN间通信有哪几种方式？各有什么优缺点？
A: 三种：①路由器物理接口——一个VLAN一个口，扩展性差；②单臂路由——子接口终结VLAN，节省接口但转发性能受限于物理口带宽，且是路由器转发性能瓶颈；③三层交换机VLANIF——转发性能高、配置简单，是现网最常用方案。
Q: VLANIF接口UP的条件是什么？
A: 两个条件：①对应VLAN已创建；②该VLAN下至少有一个物理接口处于UP状态（且属于该VLAN）。如果VLAN内所有物理口都DOWN，VLANIF也会DOWN。
Q: 单臂路由中子接口为什么要配arp broadcast enable？
A: 华为路由器子接口默认不转发广播ARP请求，导致无法主动学习对端MAC。开启arp broadcast enable后子接口才能正常处理ARP广播，实现三层互通。
Q: 三层交换机转发VLAN间流量的过程？
A: ①交换模块查MAC表，目的MAC是VLANIF的MAC，交给路由模块；②路由模块查路由表，匹配直连路由；③查ARP表获取目的IP的MAC；④交换模块重新封装帧，从目的VLAN的接口转发。
Q: Hybrid接口相比Trunk有什么优势？
A: Hybrid可以灵活控制每个VLAN发出去时是否带标签。典型场景：接服务器的口需要多个VLAN都不带标签访问服务器，用Hybrid的untagged配置比Trunk更方便。
## 五、易错点/坑
• 华为Trunk默认只允许VLAN1：这是最容易踩的坑，配完Trunk一定要port trunk allow-pass vlan xxx，否则VLAN间不通
• 子接口忘记arp broadcast enable：配完单臂路由发现ping不通，八成是这个没配
• VLANIF起不来：检查对应VLAN是否有UP的物理口，VLANIF依赖物理口状态
• 三层交换机物理口默认是二层口：想直接配IP需要先undo portswitch，否则提示错误
• Hybrid口PVID和untagged/tag列表要匹配：如果PVID不在untagged/tag列表中，该PVID的帧无法从该口发出
• 忘记回程路由：三层交换机VLANIF能通，但访问不了外网，检查是否有默认路由，以及路由器是否有到内网网段的回程静态路由
• Access口收到Tagged帧的处理：只有Tag的VLAN ID等于PVID时才接收，不等于PVID直接丢弃。不要把Trunk口能收Tagged帧的规则套到Access口上
• MAC地址划分VLAN需配hybrid或trunk上行：基于MAC的VLAN划分只在入接口生效，上行口仍需允许对应VLAN通过
