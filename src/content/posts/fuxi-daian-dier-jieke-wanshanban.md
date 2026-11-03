---
title: "复习答案 - 第二节课（完善版）"
published: 2026-10-28
description: "华为 eNSP 期中复习拓扑第二节配置答案（完善版，共 16 题）：链路聚合、MSTP、VRRP、DHCP、多区域 OSPF、RIP 双向引入、高级 ACL 与验证命令。"
image: ""
tags:
  - eNSP
  - 华为
  - VRRP
  - MSTP
  - OSPF
  - 期中复习
category: 复习资料
series: 期中复习 · 第二节课答案
seriesOrder: 2
draft: false
---

> [!NOTE]
> 本文是《复习答案 - 第二节课（完善版）》，在[原版（第 1～6 题）](/posts/fuxi-daian-dier-jieke/)基础上补齐第 7～16 题：Eth-Trunk 链路聚合、MSTP、VRRP、DHCP、多区域 OSPF、RIP 与 OSPF 双向路由引入、高级 ACL 安全控制，以及最后的验证命令。
> 完善版同时修正了原版第 5 题编号重复的问题（端口模式划分修正为第 6 题）。

## 拓扑与地址规划速查

| 网段 / VLAN | 地址规划 | 说明 |
| --- | --- | --- |
| VLAN10 | 192.168.37.0/24，网关 .254 | SW1 .252 / SW2 .253，VRID 10 |
| VLAN20 | 192.168.42.0/24，网关 .254 | SW1 .252 / SW2 .253，VRID 20 |
| VLAN30 | 192.168.47.0/24，网关 .254 | SW1 .252 / SW2 .253，VRID 30 |
| VLAN40 | 192.168.52.0/24，网关 .254 | SW1 .252 / SW2 .253，VRID 40，DHCP |
| VLAN50 / VLAN60 | 192.168.50.0/24、192.168.60.0/24 | AR4 单臂路由接入 |
| SW1–AR6 互联 | Vlanif2 10.0.0.2/24 ↔ AR6 GE0/0/0 10.0.0.1 | RIP |
| SW2–AR6 互联 | Vlanif2 10.1.1.2/24 ↔ AR6 GE0/0/1 10.1.1.1 | RIP |
| AR6–AR2 互联 | 10.3.3.0/30（AR6 .1 / AR2 .2） | OSPF Area 0 |
| AR6–AR3 互联 | 10.2.2.0/30（AR6 .1 / AR3 .2） | OSPF Area 0 |
| AR2–AR3 互联 | 10.5.5.0/30（AR2 .1 / AR3 .2） | OSPF Area 0 |
| AR2–AR4 互联 | 10.4.4.0/30（AR2 .1 / AR4 .2） | OSPF Area 1 |
| AR3–AR4 互联 | 10.6.6.0/30（AR3 .1 / AR4 .2） | OSPF Area 1 |
| 服务器网段 | 172.18.25.0/24（AR6）、172.18.26.0/24（AR2）、172.18.27.0/24（AR4） | 25 段属 Area 0，26/27 段见题目 13 |

## 1. 基础配置：修改所有网络设备的主机名

根据图示要求修改所有网络设备的主机名字。

```shell
sysname SW1
```

## 2. 关闭所有网络设备的 debug 消息提示

```shell
undo info-center enable
```

## 3. 三层交换机创建 VLAN 并配置 IP 地址

在三层交换机上创建 VLAN10、20、30、40 并分别配置 IP 地址。

**在左边交换机 SW1 上：**

```shell
vlan batch 2 10 20 30 40
interface Vlanif2
 ip address 10.0.0.2 255.255.255.0
#
interface Vlanif10
 ip address 192.168.37.252 255.255.255.0
#
interface Vlanif20
 ip address 192.168.42.252 255.255.255.0
interface Vlanif30
 ip address 192.168.47.252 255.255.255.0
#
interface Vlanif40
 ip address 192.168.52.252 255.255.255.0
```

**在右边交换机 SW2 上：**

```shell
vlan batch 2 10 20 30 40
interface Vlanif2
 ip address 10.1.1.2 255.255.255.0
#
interface Vlanif10
 ip address 192.168.37.253 255.255.255.0
#
interface Vlanif20
 ip address 192.168.42.253 255.255.255.0
interface Vlanif30
 ip address 192.168.47.253 255.255.255.0
#
interface Vlanif40
 ip address 192.168.52.253 255.255.255.0
```

## 4. 三层设备物理接口配置 IP 地址

**在 AR6 上：**

```shell
interface GigabitEthernet0/0/0
 ip address 10.0.0.1 255.255.255.252
#
interface GigabitEthernet0/0/1
 ip address 10.1.1.1 255.255.255.252
#
interface GigabitEthernet0/0/2
 ip address 172.18.25.254 255.255.255.0
#
interface GigabitEthernet4/0/0
 ip address 10.3.3.1 255.255.255.252
#
interface GigabitEthernet4/0/1
 ip address 10.2.2.1 255.255.255.252
```

**在 AR2 上：**

```shell
interface GigabitEthernet0/0/0
 ip address 10.3.3.2 255.255.255.252
#
interface GigabitEthernet0/0/1
 ip address 10.5.5.1 255.255.255.252
#
interface GigabitEthernet0/0/2
 ip address 10.4.4.1 255.255.255.252
#
interface GigabitEthernet4/0/0
 ip address 172.18.26.254 255.255.255.0
```

**在 AR3 上：**

```shell
interface GigabitEthernet0/0/0
 ip address 10.2.2.2 255.255.255.252
#
interface GigabitEthernet0/0/1
 ip address 10.5.5.2 255.255.255.252
#
interface GigabitEthernet0/0/2
 ip address 10.6.6.1 255.255.255.252
```

**在 AR4 上：**

```shell
interface GigabitEthernet0/0/0
 ip address 10.4.4.2 255.255.255.252
#
interface GigabitEthernet0/0/1
 ip address 10.6.6.2 255.255.255.252
#
interface GigabitEthernet0/0/2
#
interface GigabitEthernet4/0/0
 ip address 172.18.27.254 255.255.255.0
```

## 5. AR4 配置单臂路由实现 VLAN50 和 VLAN60 的接入

```shell
interface GigabitEthernet0/0/2.1
 dot1q termination vid 50
 ip address 192.168.50.254 255.255.255.0
 arp broadcast enable
#
interface GigabitEthernet0/0/2.2
 dot1q termination vid 60
 ip address 192.168.60.254 255.255.255.0
 arp broadcast enable
```

> [!TIP]
> 子接口必须配置 `arp broadcast enable` 开启 ARP 广播，否则单臂路由无法转发 VLAN 间的 ARP 请求。

## 6. 交换机端口模式划分

将连接终端的设备接口和只需要通过一个 VLAN 的接口划分为 Access 模式，将连接交换机以及需要通过多个 VLAN 的接口划分为 Trunk 模式。

**在 SW1 和 SW2 上分别进行配置：**

```shell
interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/3
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/24
 port link-type access
 port default vlan 2
```

**在 SW3、SW4、SW5、SW7 二层交换机上——SW3：**

```shell
vlan batch 10 20 30 40
interface Ethernet0/0/1
 port link-type access
 port default vlan 10
interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
```

**SW4：**

```shell
vlan batch 10 20 30 40
interface Ethernet0/0/1
 port link-type access
 port default vlan 20
interface Ethernet0/0/2
 port link-type access
 port default vlan 30
interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
```

**SW5：**

```shell
vlan batch 10 20 30 40
interface Ethernet0/0/1
 port link-type access
 port default vlan 40
interface Ethernet0/0/2
 port link-type access
 port default vlan 40
interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
```

**SW7：**

```shell
vlan batch 50 60
interface Ethernet0/0/1
 port link-type access
 port default vlan 50
interface Ethernet0/0/2
 port link-type access
 port default vlan 60
#
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
```

## 7. 链路聚合配置（Eth-Trunk）

在 SW1 与 SW2 之间创建 Eth-Trunk 1，采用静态 LACP 模式，将两台交换机的 GE0/0/22 和 GE0/0/23 加入聚合组，并配置 SW1 的系统 LACP 优先级为 4096，使 SW1 成为 LACP 主动端。

**在 SW1 上：**

```shell
lacp priority 4096
interface Eth-Trunk1
 mode lacp-static
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/22
 eth-trunk 1
#
interface GigabitEthernet0/0/23
 eth-trunk 1
```

**在 SW2 上：**

```shell
interface Eth-Trunk1
 mode lacp-static
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/22
 eth-trunk 1
#
interface GigabitEthernet0/0/23
 eth-trunk 1
```

## 8. 开启 MSTP

在内网三层交换机 SW1、SW2 和二层交换机 SW3、SW4、SW5 上全部开启 MSTP。

**在 SW1、SW2、SW3、SW4、SW5 上分别配置：**

```shell
stp mode mstp
```

## 9. 配置 MSTP 域

在 SW1、SW2、SW3、SW4、SW5 上配置 MSTP 域：域名为 RG1，修订级别为 1，实例 1（Instance 1）映射 VLAN10、30，实例 2（Instance 2）映射 VLAN20、40，并激活域配置。

**在 SW1、SW2、SW3、SW4、SW5 上分别配置：**

```shell
stp region-configuration
 region-name RG1
 revision-level 1
 instance 1 vlan 10 30
 instance 2 vlan 20 40
 active region-configuration
```

> [!IMPORTANT]
> 同一域内所有交换机的域名、修订级别、VLAN 与实例映射关系必须完全一致，最后必须执行 `active region-configuration` 激活。

## 10. 配置各实例的根桥

SW1 为 Instance 1 的根桥、Instance 2 的备份根桥；SW2 为 Instance 2 的根桥、Instance 1 的备份根桥。

**在 SW1 上：**

```shell
stp instance 1 root primary
stp instance 2 root secondary
```

**在 SW2 上：**

```shell
stp instance 2 root primary
stp instance 1 root secondary
```

## 11. VRRP 配置

在 SW1、SW2 上为 VLAN10、20、30、40 配置 VRRP 备份组（VRID 与 VLAN ID 一致，虚拟网关 IP 为各网段的 .254 地址）。SW1 为 VLAN10、30 的 Master（优先级 120），为 VLAN20、40 的 Backup（默认优先级 100）；SW2 为 VLAN20、40 的 Master（优先级 120），为 VLAN10、30 的 Backup（默认优先级 100），并在 Master 上配置抢占延时和上行接口 GE0/0/24 联动。

**在 SW1 上：**

```shell
interface Vlanif10
 vrrp vrid 10 virtual-ip 192.168.37.254
 vrrp vrid 10 priority 120
 vrrp vrid 10 preempt-mode timer delay 20
 vrrp vrid 10 track interface GigabitEthernet0/0/24
#
interface Vlanif30
 vrrp vrid 30 virtual-ip 192.168.47.254
 vrrp vrid 30 priority 120
 vrrp vrid 30 preempt-mode timer delay 20
 vrrp vrid 30 track interface GigabitEthernet0/0/24
#
interface Vlanif20
 vrrp vrid 20 virtual-ip 192.168.42.254
#
interface Vlanif40
 vrrp vrid 40 virtual-ip 192.168.52.254
```

**在 SW2 上：**

```shell
interface Vlanif20
 vrrp vrid 20 virtual-ip 192.168.42.254
 vrrp vrid 20 priority 120
 vrrp vrid 20 preempt-mode timer delay 20
 vrrp vrid 20 track interface GigabitEthernet0/0/24
#
interface Vlanif40
 vrrp vrid 40 virtual-ip 192.168.52.254
 vrrp vrid 40 priority 120
 vrrp vrid 40 preempt-mode timer delay 20
 vrrp vrid 40 track interface GigabitEthernet0/0/24
#
interface Vlanif10
 vrrp vrid 10 virtual-ip 192.168.37.254
#
interface Vlanif30
 vrrp vrid 30 virtual-ip 192.168.47.254
```

> [!TIP]
> VRRP 的 Master 角色与 MSTP 根桥保持一致（VLAN10/30 走 SW1，VLAN20/40 走 SW2），避免次优路径；Backup 组只需配置虚拟 IP，优先级保持默认 100。

## 12. DHCP 全局地址池

在 SW1 和 SW2 上启用 DHCP，配置全局地址池 pool_vlan40：分配网段 192.168.52.0/24，网关为 192.168.52.254，DNS 服务器为 172.18.26.100，并在 Vlanif40 接口下应用全局地址池。（配置完成后将 VLAN40 下 PC4、Client1 的 IP 获取方式改为 DHCP 自动获取）

**在 SW1 和 SW2 上分别配置：**

```shell
dhcp enable
#
ip pool pool_vlan40
 gateway-list 192.168.52.254
 network 192.168.52.0 mask 255.255.255.0
 excluded-ip-address 192.168.52.252 192.168.52.254
 dns-list 172.18.26.100
#
interface Vlanif40
 dhcp select global
```

## 13. 多区域 OSPF

AR6、AR2、AR3 之间的互联链路（10.3.3.0/30、10.2.2.0/30、10.5.5.0/30）以及 AR6 所连的 172.18.25.0/24、AR2 所连的 172.18.26.0/24 属于 OSPF 进程 1 区域 0；AR2、AR3 与 AR4 之间的互联链路（10.4.4.0/30、10.6.6.0/30）以及 AR4 所连的 172.18.27.0/24、192.168.50.0/24、192.168.60.0/24 属于区域 1。同时为各路由器配置 Router ID。

**在 AR6 上：**

```shell
router id 6.6.6.6
ospf 1
 area 0.0.0.0
  network 10.3.3.0 0.0.0.3
  network 10.2.2.0 0.0.0.3
  network 172.18.25.0 0.0.0.255
```

**在 AR2 上：**

```shell
router id 2.2.2.2
ospf 1
 area 0.0.0.0
  network 10.3.3.0 0.0.0.3
  network 10.5.5.0 0.0.0.3
  network 172.18.26.0 0.0.0.255
 area 0.0.0.1
  network 10.4.4.0 0.0.0.3
```

**在 AR3 上：**

```shell
router id 3.3.3.3
ospf 1
 area 0.0.0.0
  network 10.2.2.0 0.0.0.3
  network 10.5.5.0 0.0.0.3
 area 0.0.0.1
  network 10.6.6.0 0.0.0.3
```

**在 AR4 上：**

```shell
router id 4.4.4.4
ospf 1
 area 0.0.0.1
  network 10.4.4.0 0.0.0.3
  network 10.6.6.0 0.0.0.3
  network 172.18.27.0 0.0.0.255
  network 192.168.50.0 0.0.0.255
  network 192.168.60.0 0.0.0.255
```

> [!IMPORTANT]
> 互联链路使用 /30 反掩码 `0.0.0.3`，网段使用 /24 反掩码 `0.0.0.255`。AR2 同时属于 Area 0 和 Area 1，是区域边界路由器（ABR）。

## 14. RIPv2 与双向路由引入

AR6 与内网三层交换机 SW1、SW2 之间运行 RIPv2（AR6 的 GE0/0/0、GE0/0/1 所连的 10.0.0.0/30、10.1.1.0/30 链路），SW1、SW2 将 VLAN10、20、30、40 各网段发布进 RIP。在 AR6 上进行路由重分布，将 RIP 路由注入 OSPF，同时将 OSPF 路由引入 RIP，实现内网与骨干区域全网互通。

**在 AR6 上：**

```shell
rip 1
 version 2
 undo summary
 network 10.0.0.0
 import-route ospf 1
#
ospf 1
 import-route rip 1
```

**在 SW1 上：**

```shell
rip 1
 version 2
 undo summary
 network 10.0.0.0
 network 192.168.37.0
 network 192.168.42.0
 network 192.168.47.0
 network 192.168.52.0
```

**在 SW2 上：**

```shell
rip 1
 version 2
 undo summary
 network 10.0.0.0
 network 192.168.37.0
 network 192.168.42.0
 network 192.168.47.0
 network 192.168.52.0
```

## 15. 高级 ACL 安全控制

在 AR4 上配置高级 ACL（编号 3000），拒绝 VLAN50 网段（192.168.50.0/24）中的 PC 访问 VLAN60 网段（192.168.60.0/24）的任何流量，但允许其他流量通行，并将该 ACL 应用在子接口 GE0/0/2.1 的入方向上。

**在 AR4 上：**

```shell
acl 3000
 rule 5 deny ip source 192.168.50.0 0.0.0.255 destination 192.168.60.0 0.0.0.255
 rule 10 permit ip
#
interface GigabitEthernet0/0/2.1
 traffic-filter inbound acl 3000
```

> [!WARNING]
> ACL 默认隐放全部，必须补上 `rule 10 permit ip` 放行其他流量；ACL 应用在 VLAN50 流量的入方向（GE0/0/2.1 inbound）。

## 16. 验证配置

**在 SW1 或 SW2 上查看聚合链路状态：**

```shell
display eth-trunk 1
```

**查看 MSTP 生成树状态：**

```shell
display stp brief
```

**查看 VRRP 备份组状态：**

```shell
display vrrp brief
```

**在 AR6、AR2、AR3、AR4 上查看 OSPF 邻居关系：**

```shell
display ospf peer brief
```

**在 SW1 或 SW2 上查看通过 RIP 学到的路由表：**

```shell
display ip routing-table protocol rip
```

**在 AR6 上查看双向引入后的完整路由表：**

```shell
display ip routing-table
```

最后在 PC 上 ping 虚拟网关、其他 VLAN 的 PC 以及 FTP/DNS/www 服务器，验证全网连通性。
