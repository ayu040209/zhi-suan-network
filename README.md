# 🚀 智算网络30天冲刺计划

&gt; 目标：智算中心 / AI数据中心高性能网络（RoCE/RDMA/无损网络/Spine-Leaf）  
&gt; 学习者：阿鱼（铁道机械→网络工程师）  
&gt; 时间：2026-06-01 至 2026-06-30

---

## 📋 学习者档案

- **背景**：专科，铁道机械专业转网络工程师，已离职高密度学习
- **基础**：思科模拟器经验，会VLAN/OSPF/静态路由，HCIA-Datacom 学习中
- **设备**：i5-11260 + 16G + RTX3050，VMware 17.5，CentOS 7.6（1908）
- **模拟器**： Packet Tracer（Week 3 计划装 EVE-NG）
- **认证路线**：HCIA-Datacom（Week 1）→ HCIP（第2个月）

---

## 📅 30天计划框架

| 阶段 | 天数 | 核心内容 |
|:---:|:---:|:---|
| **Week 1** | Day 1-7 | HCIA 7天模块通关 + Linux + Python 脚本 |
| **Week 2** | Day 8-14 | 智算网络概念（RDMA/RoCE/PFC/ECN/Spine-Leaf） |
| **Week 3** | Day 15-21 | Ansible + Docker + EVE-NG 智算拓扑实验 |
| **Week 4** | Day 22-30 | 综合项目 + 简历升级 + 面试弹药 |

---

## 📚 学习资料

- **HCIA**：B站「网络达达老师」《零基础学网络工程师 HCIA Datacom 2025》94集
- **Linux**：《鸟哥的Linux私房菜》第四版（Ch4/5/6/9/10/12/15/16）
- **Python**：知乎《网络工程师的Python之路》+ Netmiko/Paramiko/Ansible
- **智算网络**：华为无损以太网白皮书、RoCE技术详解

---

## ✅ 学习日志

### Day 1（2026-06-01）
- [x] HCIA 第01-06集：网络基础 / OSI / TCP / 三次握手
- [x] Linux 基础命令：`whoami` `date` `cal` `bc` `nano`
- [x] Python 环境安装 + Hello World
- [x] GitHub 仓库初始化

### Day 2（2026-06-02）
- [x] HCIA 第18-24集：路由基础 / 静态 / 浮动 / CIDR
- [x] HCIA 第34-39集：VLAN / Trunk / Hybrid 概念
- [x] 子网划分补完（5道题，第1/3题做错已纠正）
- [x] Linux Ch5：文件权限（`chmod` `chown` `umask`）
- [x] Python Netmiko + Paramiko 安装成功
- [x] GitHub push day02（待补）

### Day 3（2026-06-03）
- [x] HCIA 第40-46集：生成树 STP / RSTP
- [x] HCIA 第47-53集：VLAN间路由 + 链路聚合/堆叠
- [x] Linux Ch6：文件与目录管理
- [x] Python 脚本1：Netmiko SSH 单台交换机执行 show 命令
- [x] GitHub 补推 day02 + day03

### Day 4（2026-06-04）
- [x] 安装EVE-NG，配置思科交换机（模拟）
- [x] Python 配合EVE-NG学习
- [x] Linux CH7

---

## 📁 目录结构
zhi-suan-network-30days/
├── week1-hcia/
│   ├── day02/
│   │   └── 路由器相关.md
│   └── day03/
│       └── STP-RSTP-VLAN间路由-链路聚合-笔记.md
├── week1-linux/
│   ├── day02/
│   │   └── Ch5-文件权限与目录配置.md
│   └── day03/
│       └── Ch6-文件与目录管理.md
├── week1-python/
│   ├── day02/
│   │   └── 环境搭建-Netmiko-Paramiko安装.md
│   └── day03/
│       └── Netmiko-SSH单台交换机-show命令.md
└── README.md
