# HCIA - SDN与NFV概述

## 一、核心概念速记（面试口述用）

| 概念 | 一句话定义 | 面试口述话术 |
|------|-----------|-------------|
| SDN | 软件定义网络，将控制平面与数据平面分离，实现集中控制 | "面试官你好，SDN是软件定义网络，核心理念是转控分离、集中控制、开放可编程。通过将网络设备的控制平面集中到控制器，实现网络的全局视图和统一编排。" |
| OpenFlow | 控制器与交换机之间的南向接口协议 | "OpenFlow是SDN最著名的南向接口协议，控制器通过OpenFlow下发流表到交换机，指导数据包转发。" |
| 流表 | OpenFlow交换机中的转发规则表 | "流表是OpenFlow交换机的转发依据，由匹配字段和动作组成，支持多表流水线处理，比传统路由表更灵活。" |
| NFV | 网络功能虚拟化，将网络功能从专用硬件解放到通用服务器 | "NFV是网络功能虚拟化，将防火墙、路由器等传统专用网络设备的功能，以软件形式运行在通用x86服务器上，降低硬件成本。" |
| 北向接口 | 控制器向上层应用提供的开放API | "北向接口是控制器面向应用的开放接口，通常基于RESTful，使上层应用可以调用网络能力。" |
| 南向接口 | 控制器向下管理设备的接口 | "南向接口是控制器管理网络设备的协议，包括OpenFlow、NETCONF、OVSDB等。" |

## 二、华为命令配置模板（直接复制粘贴）

> **注意：** SDN/NFV属于概念性模块，无传统CLI配置命令。以下为华为iMaster NCE相关基础配置 [补充]

### 场景1：设备配置NETCONF（对接SDN控制器） [补充]

```bash
# 步骤1：使能SSH（NETCONF基于SSH传输）
[Router] stelnet server enable
[Router] rsa local-key-pair create

# 步骤2：配置NETCONF
[Router] netconf ssh server enable
[Router] snetconf server enable

# 步骤3：配置AAA用户（供控制器使用）
[Router] aaa
[Router-aaa] local-user sdn-admin password irreversible-cipher Huawei@123
[Router-aaa] local-user sdn-admin service-type ssh
[Router-aaa] local-user sdn-admin level 15

# 验证命令
[Router] display netconf status
[Router] display ssh server status
```

## 三、思科-华为迁移对照表

| 思科概念/产品 | 华为概念/产品 | 注意点 |
|-------------|-------------|--------|
| Cisco ACI | CloudFabric (iMaster NCE-Fabric) | 都是数据中心SDN方案 |
| Cisco DNA Center | iMaster NCE-Campus | 都是园区SDN方案 |
| Cisco SD-WAN (Viptela) | SD-WAN (iMaster NCE-WAN) | 都是SD-WAN方案 |
| OpenFlow | OpenFlow | 都支持，是行业标准 |
| APIC-EM | iMaster NCE | 华为平台功能更全 |
| IOS XE/NX-OS的NETCONF/YANG | VRP的NETCONF/YANG | 都支持标准化接口 |

## 四、面试高频题

**Q: SDN的三个核心特征是什么？**

A: 转控分离（控制平面与数据平面分离）、集中控制（控制器统一管理网络）、开放可编程（通过API开放网络能力给上层应用）。

**Q: SDN和传统网络最大的区别？**

A: 传统网络是分布式控制，每台设备独立运行路由协议计算转发表。SDN是集中式控制，控制器统一计算并下发流表，设备只负责转发。

**Q: OpenFlow和传统路由协议的区别？**

A: 传统路由基于路由表最长匹配转发，由设备间路由协议计算生成。OpenFlow基于流表匹配转发，流表由控制器统一下发，匹配字段更丰富（可匹配MAC、IP、端口、VLAN等），动作更灵活。

**Q: NFV和SDN有什么关系？**

A: NFV和SDN是互补关系。NFV关注网络功能的软件化和通用硬件承载，SDN关注网络架构的集中控制。两者结合可以实现软件定义的网络功能，如vFW、vRouter等。

**Q: 华为iMaster NCE的定位是什么？**

A: iMaster NCE是华为的智能驾驶网络管理与控制系统，集管理、控制、分析于一体，支持数据中心（Fabric）、园区（Campus）、广域（WAN/SD-WAN）等场景，实现网络的自动化和智能化运维。

## 五、易错点/坑

- **坑1：SDN不等于OpenFlow** -- OpenFlow只是SDN的一种南向协议实现，SDN是更广泛的架构理念
- **坑2：SDN控制器不是网管** -- SDN控制器负责实时控制平面，网管（如eSight/iMaster NCE的Manager）侧重监控和运维
- **坑3：NFV不等于虚拟化** -- NFV是网络功能的虚拟化，不仅包括计算虚拟化，还包括网络虚拟化（如OVS）和存储虚拟化
- **坑4：SDN不会取代所有传统网络** -- 传统分布式控制在故障收敛等方面仍有优势，SDN更适合数据中心和园区等场景
- **坑5：南北向接口搞混** -- 北向对应用（RESTful API），南向对设备（OpenFlow/NETCONF），不要搞反
