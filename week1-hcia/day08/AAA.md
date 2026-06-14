# HCIA - AAA原理与配置

## 一、核心概念速记（面试口述用）

| 概念 | 一句话定义 | 面试口述话术 |
|------|-----------|-------------|
| AAA | Authentication认证、Authorization授权、Accounting计费的网络安全管理框架 | "面试官你好，AAA是认证、授权、计费三个单词的缩写，是一种网络安全管理机制。认证验证用户身份，授权决定用户能访问哪些资源，计费记录用户使用网络资源的情况。" |
| 认证(Authentication) | 验证用户身份是否合法 | "认证是确认用户身份的过程，常见方式有不认证、本地认证和远端认证三种。" |
| 授权(Authorization) | 确定通过认证的用户可以访问哪些服务 | "授权决定认证通过的用户能使用哪些服务，比如分配VLAN、下发ACL等权限。" |
| 计费(Accounting) | 记录用户使用网络资源的情况 | "计费功能用于监控和记录用户的网络行为，包括上网时长、流量等。" |
| RADIUS | 远程认证拨号用户服务，最常用的AAA实现协议 | "RADIUS是AAA最常用的实现协议，使用UDP端口1812认证、1813计费，具有可扩展性强、易于集中管理的优点。" |
| NAS | 网络接入服务器，负责收集和管理用户访问请求 | "NAS是网络接入服务器，作为AAA的客户端，负责把用户的认证请求转发给AAA服务器。" |

## 二、华为命令配置模板

### 场景1：本地AAA认证（Telnet登录）

```bash
# 步骤1：进入AAA视图
[R1] aaa

# 步骤2：创建认证方案（缺省为local）
[R1-aaa] authentication-scheme default
[R1-aaa-authentication-default] authentication-mode local

# 步骤3：创建本地用户
[R1-aaa] local-user huawei password cipher Huawei@123
[R1-aaa] local-user huawei service-type telnet
[R1-aaa] local-user huawei privilege level 15

# 步骤4：配置VTY用户界面使用AAA认证
[R1] user-interface vty 0 4
[R1-ui-vty0-4] authentication-mode aaa

# 验证命令
[R1] display aaa configuration
[R1] display local-user
```

### 场景2：配置domain并绑定认证方案 [补充]

```bash
# 步骤1：创建domain
[R1-aaa] domain huawei.com

# 步骤2：在domain下绑定认证方案
[R1-aaa-domain-huawei.com] authentication-scheme default

# 步骤3：创建带域名的用户
[R1-aaa] local-user admin@huawei.com password cipher Admin@123
[R1-aaa] local-user admin@huawei.com service-type telnet
[R1-aaa] local-user admin@huawei.com privilege level 15

# 验证命令
[R1] display domain name huawei.com
```

## 三、思科-华为迁移对照表

| 思科命令 | 华为命令 | 注意点 |
|----------|----------|--------|
| aaa new-model | aaa | 华为直接进入aaa视图即可 |
| aaa authentication login default local | authentication-scheme default; authentication-mode local | 华为分两步：先创建scheme再指定mode |
| username admin password Admin@123 | local-user admin password cipher Admin@123 | 华为必须指定cipher或simple |
| username admin privilege 15 | local-user admin privilege level 15 | 华为用level，思科用privilege |
| line vty 0 4; login authentication default | user-interface vty 0 4; authentication-mode aaa | 华为authentication-mode直接指定aaa |
| aaa accounting network default start-stop group radius | accounting-scheme default; accounting-mode radius | 华为accounting需单独配置scheme |
| radius-server host 10.1.1.1 key abc | radius-server template template1; radius-server shared-key cipher abc | 华为先创建模板再配置参数 |

## 四、面试高频题

**Q: AAA的三个A分别是什么？各自的作用？**

A: Authentication认证（验证身份）、Authorization授权（确定访问权限）、Accounting计费（记录资源使用）。三者协同工作，构成完整的网络安全管理框架。

**Q: 本地认证和远端认证的区别？各自适用场景？**

A: 本地认证将用户信息配置在NAS设备上，处理速度快但存储量受限，适合小型网络设备管理。远端认证使用RADIUS/TACACS+服务器集中存储用户信息，适合大规模用户接入场景如企业园区网。

**Q: RADIUS使用什么传输协议？端口是多少？**

A: RADIUS使用UDP传输，认证端口1812，计费端口1813（早期版本使用1645/1646）。UDP具有良好的实时性，配合重传机制保证可靠性。

**Q: 用户没有指定域名时属于哪个域？**

A: 不带@的用户名属于系统缺省域default（普通用户）或default_admin（管理用户）。

## 五、易错点/坑

- **坑1：service-type没配** -- 创建用户时必须指定service-type（telnet/ssh/ftp等），否则用户无法登录
- **坑2：privilege level低于3** -- 对于FTP用户，级别必须>=3，否则FTP连接失败
- **坑3：VTY下authentication-mode未改为aaa** -- 默认可能是password或none，必须显式改为aaa才会走AAA流程
- **坑4：domain绑定的scheme不存在** -- 创建domain时引用的authentication-scheme必须先创建好
- **坑5：RADIUS服务器不通** -- 配置远端认证时务必测试NAS到RADIUS服务器的网络连通性
