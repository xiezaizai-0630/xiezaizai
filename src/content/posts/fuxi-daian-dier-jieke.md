---
title: "复习答案 - 第二节课"
published: 2026-10-28
description: "华为 eNSP 期中复习拓扑第二节配置答案（原版）：主机名、关闭 debug、VLAN 与 Vlanif 地址、三层设备接口 IP、单臂路由、端口模式划分。"
image: ""
tags:
  - eNSP
  - 华为
  - 期中复习
  - 网络运维
category: 复习资料
series: 期中复习 · 第二节课答案
seriesOrder: 1
draft: false
---

> [!NOTE]
> 本文是《复习答案 - 第二节课》**原版**答案归档，包含第 1～6 题。
> 完整版在第 1～6 题基础上补齐了第 7～16 题（链路聚合、MSTP、VRRP、DHCP、多区域 OSPF、RIP 双向引入、ACL 安全控制与验证命令），见[《复习答案 - 第二节课（完善版）》](/posts/fuxi-daian-dier-jieke-wanshanban/)。

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

## 6. 交换机端口模式划分

> [!TIP]
> 将连接终端的设备接口和只需要通过一个 VLAN 的接口划分为 **Access** 模式；将连接交换机以及需要通过多个 VLAN 的接口划分为 **Trunk** 模式。
> 原版文档此题编号为"5"，与上一题重复，完善版已修正为第 6 题。

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
