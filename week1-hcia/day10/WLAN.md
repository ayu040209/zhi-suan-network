# HCIA - WLAN概述

## 一、核心概念速记（面试口述用）

| 概念 | 一句话定义 | 面试口述话术 |
|------|-----------|-------------|
| WLAN | 无线局域网，通过无线技术构建的局域网络 | "面试官你好，WLAN是无线局域网技术，广义上包括Wi-Fi、蓝牙、红外等，狭义上特指基于IEEE 802.11标准的Wi-Fi技术。" |
| Wi-Fi | 基于IEEE 802.11标准的无线局域网技术 | "Wi-Fi是Wi-Fi联盟的商标，基于IEEE 802.11标准，使用2.4GHz和5GHz频段进行无线通信。" |
| FAT AP | 胖AP，能够独立自治、自我管理的AP | "FAT AP是独立工作的接入点，自带完整功能，适合家庭或小型场景，但大规模部署时管理困难。" |
| FIT AP | 瘦AP，需要配合AC使用的AP | "FIT AP功能精简，需要和AC配合工作，由AC统一管理配置，适合大中型企业园区部署。" |
| AC | 接入控制器，统一管理FIT AP的设备 | "AC是接入控制器，负责FIT AP的集中管理、配置下发、用户认证、漫游控制等功能。" |
| CAPWAP | 无线接入点控制和配置协议 | "CAPWAP是AC和AP之间的控制协议，基于UDP，端口5246控制隧道、5247数据隧道，实现AP的集中管理。" |
| BSS/SSID/BSSID | BSS是一个AP的覆盖范围，SSID是无线网络名称，BSSID是AP的MAC地址 | "BSS是基本服务集即一个AP的覆盖范围，SSID是给用户看的无线网络名称，BSSID是用AP的MAC地址标识的。" |
| ESS | 扩展服务集，多个BSS组成的大覆盖区域 | "ESS通过多个使用相同SSID的BSS组成，实现大范围覆盖和无缝漫游。" |

## 二、华为命令配置模板（直接复制粘贴）

### 场景1：FAT AP基本配置 [补充]

```bash
# 步骤1：配置AP管理IP
<Huawei> system-view
[Huawei] interface vlanif 1
[Huawei-Vlanif1] ip address 192.168.1.1 24

# 步骤2：配置WLAN业务（FAT AP自治模式）
[Huawei] wlan
[Huawei-wlan-view] ap-id 0 type-id 69 ap-mac 00e0-fc12-3456
[Huawei-wlan-ap-0] quit

# 步骤3：配置SSID和安全策略
[Huawei-wlan-view] ssid-profile name ssid1
[Huawei-wlan-ssid-prof-ssid1] ssid HUAWEI-WLAN
[Huawei-wlan-ssid-prof-ssid1] quit

[Huawei-wlan-view] security-profile name sec1
[Huawei-wlan-sec-prof-sec1] security-policy wpa2
[Huawei-wlan-sec-prof-sec1] wpa2 authentication-method psk pass-phrase Huawei@123 encryption-method ccmp

# 验证命令
[Huawei] display wlan ssid-profile
[Huawei] display wlan security-profile
```

### 场景2：AC+FIT AP基本配置（简化版） [补充]

```bash
# AC侧配置
# 步骤1：配置AC接口和VLAN
[AC] vlan 100
[AC-vlan100] quit
[AC] interface GigabitEthernet 0/0/1
[AC-GigabitEthernet0/0/1] port link-type trunk
[AC-GigabitEthernet0/0/1] port trunk allow-pass vlan 100

# 步骤2：配置DHCP为AP分配IP
[AC] dhcp enable
[AC] interface vlanif 100
[AC-Vlanif100] ip address 10.1.1.1 24
[AC-Vlanif100] dhcp select interface

# 步骤3：配置AP上线
[AC] wlan
[AC-wlan-view] ap-group name default
[AC-wlan-ap-group-default] quit

# 步骤4：配置SSID模板并下发
[AC-wlan-view] ssid-profile name office
[AC-wlan-ssid-prof-office] ssid Office-WLAN
[AC-wlan-ssid-prof-office] quit
[AC-wlan-view] vap-profile name vap1
[AC-wlan-vap-prof-vap1] ssid-profile office
[AC-wlan-vap-prof-vap1] security-profile office-sec
[AC-wlan-vap-prof-vap1] service-vlan vlan-id 101
[AC-wlan-vap-prof-vap1] quit

# 验证命令
[AC] display ap all
[AC] display vap all
```

## 三、思科-华为迁移对照表

| 思科概念/命令 | 华为概念/命令 | 注意点 |
|-------------|-------------|--------|
| Autonomous AP | FAT AP | 名称不同，功能相同 |
| Lightweight AP + WLC | FIT AP + AC | 思科WLC=华为AC |
| LWAPP/CAPWAP | CAPWAP | 华为只支持CAPWAP |
| SSID | SSID | 概念相同 |
| BSSID | BSSID | 概念相同 |
| show wlan summary | display vap all | 华为display前缀 |
| show ap summary | display ap all | 查看AP状态 |
| config wlan create | ssid-profile name xxx | 华为使用profile模板化配置 |

## 四、面试高频题

**Q: FAT AP和FIT AP的区别？各自适用场景？**

A: FAT AP独立工作，自带完整功能，适合家庭或小型场景但管理困难。FIT AP功能精简需配合AC，由AC统一管理，适合大中型企业园区，支持集中配置、统一升级、无缝漫游。

**Q: CAPWAP协议的作用？使用什么传输层协议和端口？**

A: CAPWAP实现AC对AP的集中管理和控制，使用UDP传输。控制隧道端口5246（管理报文），数据隧道端口5247（业务报文）。支持DTLS加密保证安全。

**Q: 2.4GHz和5GHz频段各有什么优缺点？**

A: 2.4GHz覆盖范围大、穿墙能力强，但信道少（仅3个非重叠信道）、干扰多、速率低。5GHz信道多、干扰少、速率高，但覆盖范围小、穿墙能力弱。

**Q: 什么是PoE？有什么优点？**

A: PoE是以太网供电技术，通过网线同时传输数据和电力。优点包括简化布线、降低部署成本、支持远程供电管理，适合AP、IP电话、摄像头等设备。

**Q: WLAN漫游是什么？同频漫游和异频漫游的区别？**

A: 漫游是STA在ESS内不同AP间移动保持业务不中断。同频漫游在同一信道内切换，速度快；异频漫游需要切换信道，可能有短暂中断。

## 五、易错点/坑

- **坑1：CAPWAP隧道不通** -- AP获取IP后需要能访问AC的IP地址，检查中间网络是否放行了UDP 5246/5247
- **坑2：AP和AC版本不匹配** -- FIT AP的版本的AC版本需兼容，上线前确认版本配套关系
- **坑3：PoE供电不足** -- 大功率AP（如支持Wi-Fi 6的）可能需要PoE+或PoE++供电，普通PoE可能功率不够
- **坑4：2.4GHz信道重叠** -- 相邻AP如果使用1/5/9信道会互相干扰，应使用1/6/11三个非重叠信道
- **坑5：FAT AP和FIT AP模式切换** -- 部分AP支持模式切换，但切换后配置会清空，操作前备份配置
