# HCIA - 网络地址转换(NAT)

## 一、核心概念速记

| 概念 | 一句话定义 | 
|------|-----------|-------------|
| NAT | 网络地址转换，将IP数据报文中的IP地址进行转换的技术 | "面试官你好，NAT是网络地址转换技术，主要用于将私有IP地址转换为公有IP地址，缓解IPv4地址短缺问题，同时增强内网安全性。" |
| 静态NAT | 私有地址和公有地址一对一固定映射 | "静态NAT实现私有地址和公有地址的一对一固定映射，支持双向互访，适合需要从外部访问内部服务器的场景。" |
| 动态NAT | 从地址池中动态分配公有地址，不用时回收 | "动态NAT维护一个公有地址池，内网主机访问外网时临时分配地址，不用时回收，提高了地址利用率。" |
| NAPT/PAT | 网络地址端口转换，多对一映射 | "NAPT也叫PAT，不仅转换IP地址还转换端口号，实现多个私有地址共享一个公有地址，是最常用的NAT方式。" |
| Easy IP | 直接使用接口IP作为NAT转换地址的NAPT | "Easy IP是NAPT的特殊形式，直接使用出口接口的IP地址做NAT转换，适合拨号或DHCP获取动态IP的场景。" |
| NAT Server | 将内网服务器映射到公网指定地址和端口 | "NAT Server用于将内部服务器发布到公网，实现外部用户主动访问内网服务，也叫端口映射或目的NAT。" |

## 二、华为命令配置模板
### 场景1：静态NAT配置

```bash
# 步骤1：进入公网接口
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] ip address 122.1.2.1 24

# 步骤2：配置静态NAT映射（接口视图下）
[R1-GigabitEthernet0/0/1] nat static global 122.1.2.1 inside 192.168.1.1
[R1-GigabitEthernet0/0/1] nat static global 122.1.2.2 inside 192.168.1.2

# 或者系统视图下配置+接口使能
[R1] nat static global 122.1.2.1 inside 192.168.1.1
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] nat static enable

# 验证命令
[R1] display nat static
```

### 场景2：动态NAT（No-PAT）

```bash
# 步骤1：配置地址池
[R1] nat address-group 1 122.1.2.1 122.1.2.3

# 步骤2：配置ACL匹配需要转换的内网地址
[R1] acl 2000
[R1-acl-basic-2000] rule 5 permit source 192.168.1.0 0.0.0.255

# 步骤3：接口下应用动态NAT（带no-pat表示不做端口转换）
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] nat outbound 2000 address-group 1 no-pat

# 验证命令
[R1] display nat address-group
[R1] display nat outbound
```

### 场景3：NAPT配置

```bash
# 步骤1：配置地址池（可只有一个地址）
[R1] nat address-group 1 122.1.2.1 122.1.2.1

# 步骤2：配置ACL
[R1] acl 2000
[R1-acl-basic-2000] rule 5 permit source 192.168.1.0 0.0.0.255

# 步骤3：接口下应用NAPT（不加no-pat）
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] nat outbound 2000 address-group 1

# 验证命令
[R1] display nat session all
```

### 场景4：Easy IP配置

```bash
# 步骤1：配置ACL
[R1] acl 2000
[R1-acl-basic-2000] rule 5 permit source 192.168.1.0 0.0.0.255

# 步骤2：接口下应用Easy IP（不指定地址池，直接使用接口IP）
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] nat outbound 2000

# 验证命令
[R1] display nat outbound
[R1] display nat session all
```

### 场景5：NAT Server配置

```bash
# 进入公网接口配置NAT Server
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] nat server protocol tcp global 122.1.2.20 8080 inside 192.168.1.10 80

# 验证命令
[R1] display nat server
```

## 三、思科-华为迁移对照表

| 思科命令 | 华为命令 | 注意点 |
|----------|----------|--------|
| ip nat inside source static 192.168.1.1 122.1.2.1 | nat static global 122.1.2.1 inside 192.168.1.1 | 参数顺序相反 |
| ip nat pool POOL1 122.1.2.1 122.1.2.3 netmask 255.255.255.0 | nat address-group 1 122.1.2.1 122.1.2.3 | 华为不需要指定掩码 |
| ip nat inside source list 1 pool POOL1 overload | nat outbound 2000 address-group 1 | overload对应华为不加no-pat |
| ip nat inside source list 1 interface g0/0/0 overload | nat outbound 2000 | 华为Easy IP直接省略地址池 |
| ip nat inside/outside | 华为不需要指定inside/outside | 华为NAT直接在接口配置，无需标记内外 |
| show ip nat translations | display nat session all | 华为display前缀 |

## 四、面试高频题

**Q: NAT有什么优缺点？**

A: 优点：缓解IPv4地址短缺、隐藏内网拓扑增强安全性。缺点：破坏端到端通信、某些协议（如FTP、SIP）需要ALG辅助、增加网络延迟和复杂度。

**Q: 静态NAT和动态NAT的区别？什么时候用哪种？**

A: 静态NAT是一对一固定映射，支持双向访问，适合服务器发布。动态NAT从地址池临时分配，不支持外部主动访问内网，适合普通用户上网。

**Q: NAPT和动态NAT的区别？**

A: NAPT不仅转换IP还转换端口号，实现多对一映射；动态NAT只转换IP地址，是一对一映射。NAPT大幅提高了公有地址利用率，是目前最主流的NAT方式。

**Q: Easy IP和NAPT有什么区别？**

A: Easy IP是NAPT的特殊形式，直接使用接口IP作为转换地址，不需要配置地址池。适合PPPoE拨号或DHCP获取动态IP的场景。

**Q: NAT Server和静态NAT有什么区别？**

A: NAT Server可以只映射特定端口（如只映射80端口），而静态NAT是整个IP地址的映射。NAT Server更灵活安全，只需暴露必要的服务端口。

## 五、易错点/坑

- **坑1：ACL里deny和permit写反** -- NAT调用的ACL应该用permit来指定"需要转换"的流量，deny的流量不会走NAT
- **坑2：no-pat参数忘记加** -- 动态NAT不加no-pat会变成NAPT，如果需要纯动态NAT必须显式加no-pat
- **坑3：地址池和接口IP冲突** -- 地址池中的地址不能和接口实际IP冲突，否则会导致NAT失败
- **坑4：NAT Server的global地址和接口IP不在同一网段** -- 需要运营商配合路由指向，否则外部流量到不了设备
- **坑5：忘记配置默认路由** -- 内网用户要能访问公网，除了NAT还需要配置默认路由指向ISP
