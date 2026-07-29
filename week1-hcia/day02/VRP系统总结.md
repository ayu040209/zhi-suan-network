# HCIA - VRP系统基础

## 一、核心概念速记
| 概念 | 一句话定义 | 
|------|-----------|-------------|
| VRP | 华为数据通信设备的通用操作系统平台 | "VRP是华为网络设备的操作系统，类似思科IOS，提供统一命令行管理界面。" |
| 用户/系统视图 | VRP命令行的分层权限架构 | "&lt;&gt;是用户视图只能查看，[]是系统视图才能配置，进接口或协议视图前必须先system-view。" |
| 用户级别(0-15) | VRP内置的权限控制机制 | "0参观级只能ping，1监控级能display，2配置级能配业务，3-15管理级才能debug和文件操作。" |
| Console/VTY | 本地与远程登录的两种用户界面 | "Console是本地带外管理，不依赖网络；VTY是虚拟终端用于Telnet/SSH远程访问。" |

## 二、华为命令配置模板
### 场景1：系统基础配置（改名+接口IP）
```bash
&lt;Huawei&gt; system-view
[Huawei] sysname AR1
[AR1] interface GigabitEthernet 0/0/1
[AR1-GigabitEthernet0/0/1] ip address 192.168.1.1 24
[AR1-GigabitEthernet0/0/1] quit

### 场景2：VTY远程登录（密码认证+只读权限）
[AR1] user-interface vty 0 4
[AR1-ui-vty0-4] authentication-mode password
Please configure the login password: huawei123
[AR1-ui-vty0-4] user privilege level 1
[AR1-ui-vty0-4] quit

### 场景3：保存配置并指定下次启动文件
<AR1> save huawei.zip
# 输入 y 确认
<AR1> startup saved-configuration huawei.zip
# 验证
<AR1> display startup

### 场景4：文件系统常用操作
<Huawei> pwd
<Huawei> dir
<Huawei> mkdir test
<Huawei> rmdir test
<Huawei> rename a.txt b.txt
<Huawei> copy a.txt b.txt
<Huawei> move a.txt flash:/dhcp/
<Huawei> delete a.txt
<Huawei> undelete a.txt
<Huawei> reset recycle-bin    # [补充] 彻底清空回收站

## 三、思科-华为迁移对照表
| 思科命令                   | 华为命令                            | 注意点                         |
| ---------------------- | ------------------------------- | --------------------------- |
| enable                 | system-view                     | 思科进特权模式，华为进系统视图；华为无enable密码 |
| configure terminal     | system-view                     | 华为直接system-view，无需单独conf t  |
| show                   | display                         | 所有查看命令前缀，必须牢记               |
| hostname               | sysname                         | 功能相同，关键字不同                  |
| no                     | undo                            | 删除/恢复默认配置的前缀                |
| write / copy run start | save                            | **save在用户视图执行**，系统视图敲会报错    |
| interface f0/0         | interface GigabitEthernet 0/0/1 | 华为命名格式：槽位/子卡/端口             |
| end                    | return / Ctrl+Z                 | 直接返回用户视图                    |
| line vty 0 4           | user-interface vty 0 4          | 关键字不同                       |
| ip address x x         | ip address x x                  | 格式相同，华为支持掩码长度写法如24          |

## 四、面试高频题
Q: VRP是什么？和思科IOS有什么主要区别？
A: VRP是华为网络设备的操作系统。与IOS相比，VRP采用分层视图（用户/系统/接口/协议），查看命令用display，删除配置用undo。
Q: VRP用户级别有哪些？面试常考哪两级？
A: 0参观、1监控、2配置、3-15管理。常考2级（能配业务）和3级（能debug和文件操作）的区别。
Q: 为什么配完要save？startup saved-configuration是干嘛的？
A: 运行配置在内存中掉电丢失，save写入Flash持久化。startup用于指定下次重启加载哪个配置文件，默认是vrpcfg.zip。
Q: VRP5和VRP8在配置生效机制上有什么区别？
A: VRP5配置直接生效，无需提交；VRP8有候选配置库，配完后需要commit才能生效到运行配置库。
Q: 生产环境远程登录应该用什么方式？为什么？
A: 必须用SSH。Telnet明文传输密码和配置，存在严重安全隐患；SSH加密传输，是运维规范要求。

## 五、易错点/坑
• save必须在用户视图执行：系统视图下敲save会提示错误，先quit或按Ctrl+Z退回<>再save
• undo不等于no的简单替换：虽然功能类似，但部分华为命令的undo形式有细微差异，不能无脑套
• 接口命名格式：华为固定为槽位/子卡/端口（如0/0/1），不要写成思科的f0/0习惯
• VTY密码配置因版本而异：不同VRP版本命令可能略有差别，实验以V200R007C00为准
• VRP8需要commit：如果以后接触NE9000等高端设备，记住配完要commit，否则重启后配置丢失

