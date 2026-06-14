# HCIA - ACL原理与配置

## 一、核心概念速记（面试口述用）

| 概念 | 一句话定义 | 面试口述话术 |
|------|-----------|-------------|
| ACL | 访问控制列表，由一系列permit/deny规则组成的有序列表，用于匹配和区分报文 | "面试官你好，ACL是一种访问控制列表技术，通过permit或deny规则对报文的源IP、目的IP、端口等字段进行匹配，实现流量过滤、QoS、NAT等功能的访问控制。" |
| 基本ACL | 编号2000~2999，仅匹配源IP地址 | "基本ACL只能根据报文的源IP地址进行匹配，编号范围2000到2999，配置简单但粒度粗。" |
| 高级ACL | 编号3000~3999，可匹配源/目的IP、协议类型、端口号等 | "高级ACL可以根据源IP、目的IP、协议类型、TCP/UDP端口号等多个字段进行匹配，编号3000到3999，粒度更精细。" |
| 通配符 | 32bit数值，0表示必须匹配，1表示忽略 | "通配符用于ACL中指定IP地址哪些位需要严格匹配，0表示必须匹配该位，1表示忽略，和子网掩码的逻辑正好相反。" |
| 步长 | 规则编号之间的间隔，默认值为5 | "步长是ACL规则编号的增量间隔，默认是5，目的是方便后续在规则之间插入新规则。" |

## 二、华为命令配置模板

### 场景1：基本ACL过滤流量（禁止某网段访问服务器）

```bash
# 步骤1：创建基本ACL
[Router] acl 2000
[Router-acl-basic-2000] rule deny source 192.168.1.0 0.0.0.255
[Router-acl-basic-2000] rule permit source any

# 步骤2：在接口入方向应用ACL
[Router] interface GigabitEthernet 0/0/1
[Router-GigabitEthernet0/0/1] traffic-filter inbound acl 2000

# 验证命令
[Router] display acl 2000
[Router] display traffic-filter applied-record
```

### 场景2：高级ACL限制部门互访

```bash
# 步骤1：创建高级ACL拒绝研发部访问市场部
[Router] acl 3001
[Router-acl-adv-3001] rule deny ip source 10.1.1.0 0.0.0.255 destination 10.1.2.0 0.0.0.255

# 步骤2：创建高级ACL拒绝市场部访问研发部
[Router] acl 3002
[Router-acl-adv-3002] rule deny ip source 10.1.2.0 0.0.0.255 destination 10.1.1.0 0.0.0.255

# 步骤3：在接口入方向应用ACL
[Router] interface GigabitEthernet 0/0/1
[Router-GigabitEthernet0/0/1] traffic-filter inbound acl 3001
[Router] interface GigabitEthernet 0/0/2
[Router-GigabitEthernet0/0/2] traffic-filter inbound acl 3002

# 验证命令
[Router] display acl all
[Router] display traffic-filter applied-record
```

### 场景3：命名型ACL配置 [补充]

```bash
# 创建命名型基本ACL
[Router] acl name deny_r&d basic
[Router-acl-basic-deny_r&d] rule deny source 10.1.1.0 0.0.0.255

# 创建命名型高级ACL
[Router] acl name adv_filter advance
[Router-acl-adv-adv_filter] rule permit tcp source 10.1.1.0 0.0.0.255 destination 10.1.3.0 0.0.0.255 destination-port eq 80

# 验证命令
[Router] display acl name deny_r&d
```

## 三、思科-华为迁移对照表

| 思科命令 | 华为命令 | 注意点 |
|----------|----------|--------|
| access-list 1 deny 192.168.1.0 0.0.0.255 | acl 2000; rule deny source 192.168.1.0 0.0.0.255 | 思科用反向掩码，华为用通配符（逻辑相同） |
| ip access-group 1 in | traffic-filter inbound acl 2000 | 华为用traffic-filter，思科用access-group |
| show access-list | display acl all | 华为display前缀 |
| show ip interface (查看ACL应用) | display traffic-filter applied-record | 华为有专门命令查看ACL应用记录 |
| ip access-list extended | acl 3000 (高级ACL) | 思科命名ACL分standard/extended，华为分basic/advance |
| permit tcp any any eq 80 | rule permit tcp source any destination any destination-port eq 80 | 语法结构略有不同 |
| no access-list 1 | undo acl 2000 | 删除方式相同 |

## 四、面试高频题

**Q: ACL的匹配原则是什么？为什么规则顺序很重要？**

A: ACL匹配遵循"一旦命中即停止匹配"的原则，从编号最小的规则开始逐条匹配。规则顺序至关重要，因为如果把permit any放在前面，后面的deny规则永远不会被匹配到。

**Q: 基本ACL和高级ACL的区别是什么？应该在什么位置应用？**

A: 基本ACL（2000-2999）只能匹配源IP地址，建议应用在靠近目的地的接口；高级ACL（3000-3999）可匹配源/目的IP、协议、端口等，建议应用在靠近源的接口，减少不必要的流量传输。

**Q: 通配符0.0.0.255和子网掩码255.255.255.0有什么区别？**

A: 通配符中0表示"必须匹配"，1表示"忽略"；子网掩码中1表示网络位，0表示主机位。两者逻辑相反但表示的匹配范围相同，0.0.0.255对应的就是/24网段。

**Q: 默认步长是5，如果我需要插入一条规则怎么办？**

A: 可以利用步长留下的编号间隙直接插入，比如rule 7插入到rule 5和rule 10之间。如果间隙不够，可以用step命令修改步长后重新分配编号。

## 五、易错点/坑

- **坑1：忘记最后加permit any** -- ACL末尾隐含deny all，如果没有显式permit，所有未匹配流量都会被拒绝
- **坑2：方向搞反** -- inbound是进入接口的方向，outbound是离开接口的方向，搞反会导致ACL不生效
- **坑3：通配符和掩码混淆** -- 华为通配符0=匹配、1=忽略，和掩码逻辑相反，配置时容易搞混
- **坑4：修改已应用的ACL** -- 在线修改已应用的ACL可能导致业务中断，建议在维护窗口操作或使用ACL复制功能
- **坑5：高级ACL协议关键字** -- 匹配TCP/UDP时必须显式写tcp/udp，不能直接写ip然后指望匹配端口
