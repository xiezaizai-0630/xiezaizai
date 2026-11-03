---
title: "常用命令（华为 VRP 配置命令手册）"
published: 2026-11-03
description: "依据复习样题 4 个任务知识点整理的华为 VRP 命令手册：链路聚合（静态 LACP）、MSTP、VRRP、全局 DHCP、OSPF、RIPv2 双向引入、单臂路由、高级 ACL，每条命令按「命令 → 意思 → 怎么用」讲解。"
image: ""
tags:
  - 华为
  - VRP
  - eNSP
  - VRRP
  - MSTP
  - OSPF
  - 期中复习
category: 复习资料
series: 期中复习 · 第二节课答案
seriesOrder: 3
draft: false
---

> [!NOTE]
> 本手册依据《复习样题》中的 4 个任务知识点整理，覆盖链路聚合（静态 LACP）、MSTP、VRRP、全局 DHCP、OSPF、RIPv2 双向引入、单臂路由、高级 ACL 所需的全部配置命令。每条命令按「命令 → 意思 → 怎么用」三段讲解，命令基于华为 VRP（S 系列交换机 / AR 系列路由器）语法，不同型号个别命令可能略有差异。
> 配套完整拓扑配置见[《复习答案 - 第二节课（完善版）》](/posts/fuxi-daian-dier-jieke-wanshanban/)。

## 〇、先看懂：视图、提示符与命令约定

华为设备采用分层视图结构，命令必须在正确的视图下输入，视图不对会报 `Error: Unrecognized command`。

| 视图 | 进入方式 | 提示符 | 主要用途 |
| --- | --- | --- | --- |
| 用户视图 | 登录后默认 | `<Huawei>` | 查看、保存、测试 |
| 系统视图 | `system-view` | `[Huawei]` | 全局配置：VLAN、进程、ACL、地址池 |
| 接口视图 | `interface <接口>` | `[Huawei-GigabitEthernet0/0/1]` | IP、链路类型、VRRP、ACL 应用 |
| 协议视图 | `ospf 1` / `rip 1` | `[Huawei-ospf-1]` | 宣告网段、路由引入 |
| ACL 视图 | `acl 3000` | `[Huawei-acl-adv-3000]` | 写规则 rule |
| 地址池视图 | `ip pool <名字>` | `[Huawei-ip-pool-xxx]` | 地址池参数 |
| MST 域视图 | `stp region-configuration` | `[Huawei-mst-region]` | 域名、VLAN 与实例映射 |

**三条铁律**

- `quit` 退回上一级视图；`return`（或 Ctrl+Z）直接回用户视图。
- `undo <原命令>` 删除配置或恢复缺省值，例如 `undo shutdown` 表示开启接口。
- 配置只存在于内存中，重启会丢失，必须在用户视图执行 `save` 写入闪存。

**帮助与简写**

- 输入 `?` 列出当前视图可用命令；命令后加空格再输 `?` 可看参数，例如 `ip address ?`。
- 命令支持不冲突的简写，例如 `sys` 等于 `system-view`、`dis` 等于 `display`；Tab 键自动补全。
- 命令关键字不区分大小写，但 VLAN 名、MST 域名、地址池名区分大小写。

**反掩码（通配符掩码）**

OSPF 的 `network`、ACL 的 `source` 与 `destination` 都用反掩码：0 表示必须匹配，255 表示忽略。常用对照：`255.255.255.0`（/24）对应 `0.0.0.255`；`255.255.255.252`（/30）对应 `0.0.0.3`。

## 一、通用基础命令

### 1.1 本章命令一览

| 命令 | 作用 | 使用视图 |
| --- | --- | --- |
| `system-view` | 进入系统视图 | 用户视图 |
| `sysname <名称>` | 设置设备名 | 系统视图 |
| `interface <接口>` | 进入（或创建）接口视图 | 系统视图 |
| `vlan batch <列表>` | 批量创建 VLAN | 系统视图 |
| `port link-type <类型>` | 设置二层口链路类型 | 交换机接口视图 |
| `port trunk allow-pass vlan <列表>` | trunk 口允许通过的 VLAN | trunk 接口视图 |
| `port default vlan <ID>` | access 口划入 VLAN | access 接口视图 |
| `ip address <IP> <掩码>` | 配置三层接口 IP | 三层接口视图 |
| `undo shutdown` | 激活接口 | 接口视图 |
| `quit` / `return` | 退出视图 | 任意视图 |
| `undo <命令>` | 删除配置 | 对应视图 |
| `save` | 保存配置到闪存 | 用户视图 |
| `display current-configuration` | 查看全部生效配置 | 用户 / 系统视图 |
| `display this` | 查看当前视图下的配置 | 任意子视图 |
| `?` | 在线帮助 | 任意视图 |
| `ping` / `tracert` | 连通性 / 路径测试 | 用户视图 |

### 1.2 逐条详解

**1. system-view**

- 意思：从「只能看」的用户视图切换到「可以改」的系统视图，是全部配置的起点。
- 怎么用：登录后第一条配置命令；提示符由 `<Huawei>` 变为 `[Huawei]`。简写 `sys`。

**2. sysname \<名称>**

- 意思：给设备命名，便于区分 SW1、SW2、R1、R3 等多台设备。
- 怎么用：系统视图下执行，改完提示符立刻变成 `[SW1]`。名称不含空格、区分大小写。
- 举例：`sysname SW1`

**3. interface \<接口名>**

- 意思：进入接口视图；若接口是逻辑接口且尚不存在（Eth-Trunk、Vlanif、子接口），会顺带创建它。
- 怎么用：系统视图下执行。常见写法：`GigabitEthernet 0/0/23`、`Eth-Trunk 1`、`Vlanif 10`、`GigabitEthernet 0/0/1.50`。接口编号必须与设备实际槽位一致。
- 举例：`interface Vlanif 10`

**4. vlan batch \<VLAN 列表>**

- 意思：一次批量创建多个 VLAN，省去逐个敲 `vlan 10`。
- 怎么用：系统视图；VLAN 号用空格分隔，连续段可写 `10 to 20`。创建后用 `display vlan` 核对。
- 举例：`vlan batch 10 20 30 40`

**5. port link-type \<access / trunk / hybrid>**

- 意思：设置交换机二层口的链路类型。access 只承载一个 VLAN（接 PC、打印机）；trunk 承载多个 VLAN 且带 802.1Q 标签（接交换机或路由器子接口）；hybrid 为混合模式。
- 怎么用：交换机接口视图下执行；trunk 口必须配合下一条命令放行 VLAN，否则默认只放行 VLAN 1。
- 举例：`port link-type trunk`

**6. port trunk allow-pass vlan \<VLAN 列表>**

- 意思：指定 trunk 口允许哪些 VLAN 的帧通过，列表之外的 VLAN 帧会被丢弃。
- 怎么用：trunk 接口视图；可写 `vlan all` 放行全部。链路两端放行的 VLAN 必须一致，否则该 VLAN 不通。
- 举例：`port trunk allow-pass vlan 10 20 30 40`

**7. port default vlan \<ID>**

- 意思：把 access 口划入指定 VLAN，PC 发出的无标签帧进入端口时被打上该 VLAN 标签。
- 怎么用：access 接口视图；VLAN 必须已创建。
- 举例：`port default vlan 50`

**8. ip address \<IP 地址> \<掩码>**

- 意思：给三层接口配置 IP，使其成为该网段的网关或互联地址。
- 怎么用：Vlanif、三层物理口、子接口视图下执行；掩码可写点分十进制，也可写位数（如 `24`）。同一设备不同三层接口不能配置重叠网段。
- 举例：`ip address 192.168.10.1 255.255.255.0`、`ip address 10.1.1.1 30`

**9. shutdown / undo shutdown**

- 意思：管理性关闭 / 开启接口。华为部分接口默认关闭，或改动后需要激活。
- 怎么用：接口视图；`undo shutdown` 是最常用的「启用」命令。排障时先看 `display ip interface brief` 中的 Status 是否为 up。

**10. quit / return**

- 意思：`quit` 退回上一级视图，`return` 从任意视图一步回到用户视图。
- 怎么用：任意视图；`return` 的快捷键是 Ctrl+Z。配置完一层要退出才能进另一层，新手常忘。

**11. undo \<原命令>**

- 意思：删除某条配置或恢复缺省值，是改错、清理配置的标准做法。
- 怎么用：在原命令前加 `undo`，且必须在同一视图下执行。例如 `undo vrrp vrid 10 priority 120` 把优先级改回默认 100。
- 举例：`undo port trunk allow-pass vlan 10`

**12. save**

- 意思：把内存中的当前配置写入闪存，设备重启后不丢失。
- 怎么用：必须在用户视图执行（系统视图下会提示不识别）；会询问 `Are you sure to continue?[Y/N]`，输入 `y`。这是最容易漏的一步。
- 举例：`return` 后执行 `save`

**13. display current-configuration**

- 意思：查看当前生效的全部配置，用来核对命令有没有敲错、敲漏、敲错视图。
- 怎么用：用户视图或系统视图；可配合管道过滤，例如 `display current-configuration | include vrrp` 只显示含 vrrp 的行。

**14. display this**

- 意思：只显示当前视图下已配置的内容，排查某个接口 / 协议时最省事。
- 怎么用：进入接口视图、协议视图后直接执行，不需要回系统视图。
- 举例：先 `interface Vlanif 10`，再 `display this`

**15. ?（在线帮助）**

- 意思：列出当前视图可用命令，或显示某条命令后续可填的参数与取值范围。
- 怎么用：直接输 `?`，或在命令后空一格再输 `?`。例如 `vrrp vrid 10 ?` 能看到该备份组可配的所有参数。
- 举例：`ip address ?`

**16. ping / tracert**

- 意思：测试连通性 / 跟踪报文经过的三层路径。
- 怎么用：用户视图；`ping -a <源 IP> <目的 IP>` 可指定源地址，验证 ACL 或策略路由时非常有用；`tracert <目的 IP>` 看逐跳。
- 举例：`ping -a 192.168.50.1 192.168.60.1`

## 二、任务 1：链路聚合与多实例生成树（LACP + MSTP）

### 2.1 知识点回顾

在 SW1 与 SW2 之间创建 Eth-Trunk 1，采用静态 LACP 模式，把两台交换机的 GE 0/0/23、GE 0/0/24 加入聚合口；配置 SW1 的系统优先级为 4096 使其成为 LACP 主动端；所有交换机配置 MSTP，域名 RG1、修订级别 1，实例 1 映射 VLAN 10、30，实例 2 映射 VLAN 20、40；SW1 为实例 1 主根桥、实例 2 备份根桥，SW2 反之。

### 2.2 链路聚合命令一览

| 命令 | 作用 | 使用视图 |
| --- | --- | --- |
| `interface Eth-Trunk 1` | 创建并进入聚合逻辑口 | 系统视图 |
| `mode lacp-static` | 设为静态 LACP 模式 | Eth-Trunk 接口视图 |
| `lacp priority 4096` | 设置 LACP 系统优先级，决定主动端 | 系统视图 |
| `eth-trunk 1` | 把物理口加入聚合组 | 物理接口视图 |
| `port link-type trunk` | 聚合口设为 trunk | Eth-Trunk 接口视图 |
| `port trunk allow-pass vlan <列表>` | 放行 VLAN | Eth-Trunk 接口视图 |
| `load-balance <方式>` | 设置负载分担依据（可选） | Eth-Trunk 接口视图 |
| `max active-linknumber 2` | 限制活动链路数（可选） | Eth-Trunk 接口视图 |
| `display eth-trunk 1` | 查看聚合口成员与状态 | 用户 / 系统视图 |

### 2.3 链路聚合逐条详解

**1. interface Eth-Trunk 1**

- 意思：创建一个编号为 1 的 Eth-Trunk 逻辑接口（编号范围视型号，通常为 0～63）并进入其视图。它把多条物理链路绑成一条逻辑链路，带宽叠加、互为备份。
- 怎么用：系统视图下执行。关键顺序：先建 Eth-Trunk 并配好模式，再把物理口加进来；后续所有二层配置（trunk、放行 VLAN）只在这个逻辑口上做一次，不要在成员物理口上做。

**2. mode lacp-static**

- 意思：把聚合模式改为「静态 LACP」。默认模式是 manual（手工负载分担，不跑协议）；静态 LACP 会交互 LACPDU 报文协商活动链路，能检测链路错连、单纤故障。
- 怎么用：Eth-Trunk 接口视图下执行，且必须在成员口加入之前配置，否则部分平台会拒绝。两端设备必须配置相同的模式，一端 LACP 一端手工无法协商。

**3. lacp priority 4096**

- 意思：配置本设备的 LACP 系统优先级，取值范围 0～65535，缺省 32768，数值越小越优先。系统优先级小的一端成为「主动端」，由它决定哪些成员链路被选为活动链路。
- 怎么用：系统视图下执行（注意不是接口视图）。题目把 SW1 设为 4096，SW2 保持缺省 32768，因此 SW1 是主动端。接口下另有 `lacp priority` 用于设置「接口 LACP 优先级」，两者不要混淆。

**4. eth-trunk 1**

- 意思：把当前物理接口加入 Eth-Trunk 1，成为成员口。
- 怎么用：物理接口视图下执行，GE 0/0/23 和 GE 0/0/24 各执行一次。加入前该物理口必须是「干净」的二层口：不能有 IP、不能配过链路类型 / VLAN / 其他业务配置，否则会报错，需要先用 undo 清掉（或 `clear configuration interface` 清空）。两端端口的速率、双工、类型必须一致。
- 举例：`interface GigabitEthernet 0/0/23` → `eth-trunk 1`

**5. port link-type trunk + port trunk allow-pass vlan 10 20 30 40**

- 意思：把聚合逻辑口配成 trunk，允许题目中的 4 个 VLAN 通过。
- 怎么用：在 Eth-Trunk 接口视图下配置，成员物理口不需要重复配置。

**6. load-balance \<方式>（可选）**

- 意思：指定聚合组内流量按什么哈希分担，常用 `src-dst-mac`（源+目的 MAC）、`src-dst-ip`、`src-ip`、`dst-ip`。
- 怎么用：Eth-Trunk 接口视图；两端建议一致。缺省一般按源+目的 MAC/IP 分担。

**7. max active-linknumber 2（可选）**

- 意思：限制同时处于转发状态的成员链路数量，超出的链路做备份（配合 LACP 抢占实现冗余）。
- 怎么用：Eth-Trunk 接口视图；题目只有 2 条成员链路，配 2 即全部活动。

**8. display eth-trunk 1**

- 意思：查看聚合口的工作模式、成员口、选中（Selected/Up）状态与活动链路数，是验证聚合是否成功的首选命令。
- 怎么用：用户视图或系统视图；还可看 `display lacp statistics eth-trunk 1` 检查 LACP 报文收发。

### 2.4 MSTP 命令一览

| 命令 | 作用 | 使用视图 |
| --- | --- | --- |
| `stp mode mstp` | 设置生成树模式为 MSTP | 系统视图 |
| `stp enable` | 全局启用生成树 | 系统视图 |
| `stp region-configuration` | 进入 MST 域视图 | 系统视图 |
| `region-name RG1` | 设置 MST 域名 | MST 域视图 |
| `revision-level 1` | 设置修订级别 | MST 域视图 |
| `instance 1 vlan 10 30` | VLAN 到实例的映射 | MST 域视图 |
| `active region-configuration` | 提交 / 激活域配置 | MST 域视图 |
| `stp instance 1 root primary` | 设为实例 1 主根桥 | 系统视图 |
| `stp instance 2 root secondary` | 设为实例 2 备份根桥 | 系统视图 |
| `display stp region-configuration` | 查看生效的域配置 | 用户 / 系统视图 |

### 2.5 MSTP 逐条详解

**1. stp mode mstp / stp enable**

- 意思：把生成树工作模式设为 MSTP（多实例生成树），并全局启用 STP。MSTP 可让不同 VLAN 组走不同实例、不同根桥，从而实现链路负载分担，避免 STP 把所有冗余链路都阻塞。
- 怎么用：系统视图下执行；华为交换机缺省即 MSTP 且默认开启，显式写出更稳妥，且所有交换机必须一致。

**2. stp region-configuration**

- 意思：进入 MST 域配置视图，域内的三要素（域名、修订级别、VLAN-实例映射）都在这里配。
- 怎么用：系统视图下执行，提示符变为 `[Huawei-mst-region]`。只有「域三要素完全相同」的交换机才属于同一个 MST 域，否则各自按 CIST 计算，负载分担失效。

**3. region-name RG1**

- 意思：设置 MST 域名，题目要求 RG1。
- 怎么用：MST 域视图；区分大小写，所有交换机必须逐字一致。

**4. revision-level 1**

- 意思：设置 MST 域修订级别，题目要求 1。取值 0～65535，缺省 0，也必须与域内其他设备一致。

**5. instance 1 vlan 10 30 / instance 2 vlan 20 40**

- 意思：把 VLAN 映射到生成树实例。同一实例内的 VLAN 共用一棵生成树；实例 0 是缺省实例（CIST/IST），所有未显式映射的 VLAN 自动归实例 0。
- 怎么用：MST 域视图；多个 VLAN 用空格分隔，连续段可写 `vlan 10 to 30`。题目：实例 1 管 VLAN 10、30，实例 2 管 VLAN 20、40。所有交换机映射表必须完全相同。

**6. active region-configuration**

- 意思：提交并激活前面配置的域三要素，使其生效。
- 怎么用：MST 域视图下执行，这是最容易漏的一条。不执行的话，前面配的域名 / 修订级别 / 映射都不会生效；修改域配置后也必须重新执行一次。可用 `check region-configuration` 查看尚未激活的配置，用 `display stp region-configuration` 查看已生效配置。

**7. stp instance 1 root primary / stp instance 2 root secondary**

- 意思：把本设备指定为某实例的主根桥 / 备份根桥。`root primary` 等价于把该实例优先级设为 0，`root secondary` 等价于 4096；主根桥故障时备份根桥接管。
- 怎么用：系统视图下执行，按实例分别配置。题目：SW1 为实例 1 主根、实例 2 备份根；SW2 为实例 2 主根、实例 1 备份根，这样 VLAN 10/30 与 VLAN 20/40 的流量分别走不同上行链路。
- 等价手写方式：`stp instance 1 priority 0`、`stp instance 2 priority 4096`。优先级取值范围 0～61440，必须是 4096 的倍数，数值越小越优先。手动指定时两台设备的优先级要拉开差距。

**8. display stp instance 1 brief**

- 意思：查看实例 1 各端口的角色（ROOT/DESI/ALTE）与状态（FORWARDING/DISCARDING），确认阻塞端口是否符合预期。
- 怎么用：用户 / 系统视图；`display stp` 看全局，`display stp region-configuration` 看域配置。

### 2.6 完整配置示例

**SW1（LACP 主动端、实例 1 主根桥）：**

```shell
system-view
sysname SW1
vlan batch 10 20 30 40
lacp priority 4096
interface Eth-Trunk 1
 mode lacp-static
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40
quit
interface GigabitEthernet 0/0/23
 eth-trunk 1
quit
interface GigabitEthernet 0/0/24
 eth-trunk 1
quit
stp mode mstp
stp enable
stp region-configuration
 region-name RG1
 revision-level 1
 instance 1 vlan 10 30
 instance 2 vlan 20 40
 active region-configuration
quit
stp instance 1 root primary
stp instance 2 root secondary
return
save
```

**SW2 与 SW1 的差异：**

- 不配 `lacp priority 4096`（保持缺省 32768，让 SW1 成为主动端）；Eth-Trunk 1 的 `mode lacp-static` 与 trunk 配置相同。
- 根桥角色对调：`stp instance 2 root primary`、`stp instance 1 root secondary`。
- MST 域三要素（域名 RG1、修订级别 1、实例映射）必须与 SW1 完全一致，同样要执行 `active region-configuration`。

## 三、任务 2：VRRP 双机热备与全局 DHCP

### 3.1 知识点回顾

SW1 与 SW2 作为各 VLAN 网关：VLAN 10 网段 192.168.10.0/24、虚拟网关 192.168.10.254；VLAN 20 网段 192.168.20.0/24、虚拟网关 192.168.20.254。VRRP 备份组的 VRID 与 VLAN ID 一致；SW1 为 VLAN 10 的 Master（优先级 120）、VLAN 20 的 Backup（缺省 100）；SW2 反之。两台交换机启用 DHCP，全局地址池 pool_vlan10 分配 192.168.10.0/24，网关 192.168.10.254，DNS 172.16.1.100。

### 3.2 VRRP 命令一览

| 命令 | 作用 | 使用视图 |
| --- | --- | --- |
| `interface Vlanif 10` | 创建 VLAN 三层网关接口 | 系统视图 |
| `ip address <IP> <掩码>` | 配置网关真实 IP | Vlanif 接口视图 |
| `vrrp vrid 10 virtual-ip 192.168.10.254` | 创建备份组并指定虚拟网关 IP | Vlanif 接口视图 |
| `vrrp vrid 10 priority 120` | 设置本端优先级，决定 Master | Vlanif 接口视图 |
| `vrrp vrid 10 preempt-mode timer delay 5` | 抢占延时（可选） | Vlanif 接口视图 |
| `vrrp vrid 10 track interface <接口> reduced 30` | 监视上行口并降优先级（可选） | Vlanif 接口视图 |
| `display vrrp` | 查看备份组状态 | 用户 / 系统视图 |

### 3.3 VRRP 逐条详解

**1. interface Vlanif 10 + ip address 192.168.10.1 255.255.255.0**

- 意思：Vlanif 是 VLAN 的三层虚接口，承担该 VLAN 内 PC 的网关功能；给它配真实 IP。
- 怎么用：系统视图下进入 Vlanif（VLAN 必须已创建）；SW1 用 192.168.10.1，SW2 用 192.168.10.2，两端同网段但不能相同。VRRP 虚拟 IP 单独由下一条命令指定。

**2. vrrp vrid 10 virtual-ip 192.168.10.254**

- 意思：创建 VRRP 备份组 10 并指定虚拟 IP。两台设备对外呈现同一个网关 IP，Master 实际转发流量，Master 故障时 Backup 自动接管，PC 无需改网关。
- 怎么用：Vlanif 接口视图下执行。vrid 范围 1～255，题目要求与 VLAN ID 一致（VLAN 10 用 vrid 10，VLAN 20 用 vrid 20）——这只是题目约定，协议本身不要求。虚拟 IP 必须与接口 IP 同网段，且同一备份组在两台设备上的 vrid 和虚拟 IP 必须完全一致。

**3. vrrp vrid 10 priority 120**

- 意思：设置本设备在备份组 10 中的优先级，范围 1～254，缺省 100，越大越优先，优先级高者成为 Master。0 与 255 为保留值（当虚拟 IP 与接口 IP 相同时，该设备自动成为优先级 255 的「IP 地址拥有者」）。
- 怎么用：Vlanif 接口视图；只在希望当 Master 的那台设备上配。题目：SW1 在 Vlanif 10 配 120（Master），SW2 在 Vlanif 20 配 120（Master）；其余备份组保持缺省 100 即为 Backup。

**4. vrrp vrid 10 preempt-mode timer delay 5（可选）**

- 意思：设置抢占延时。优先级更高的设备恢复后，等待若干秒再抢回 Master，避免链路震荡时频繁切换；VRP 缺省开启抢占。
- 怎么用：Vlanif 接口视图，单位秒。

**5. vrrp vrid 10 track interface \<接口> reduced 30（可选）**

- 意思：监视上行接口，一旦该接口 Down，就把本端优先级降低 30，让对端接管，避免「上行断了但网关仍是自己」造成黑洞。
- 怎么用：Vlanif 接口视图；`<接口>` 换成实际的上行口，如 `GigabitEthernet 0/0/1`。也可 `track bfd-session`。
- 举例：`vrrp vrid 10 track interface GigabitEthernet 0/0/1 reduced 30`

**6. display vrrp / display vrrp brief**

- 意思：查看每个备份组的 State（Master / Backup）、虚拟 IP、优先级、Master IP。
- 怎么用：用户 / 系统视图；配完后应在两台设备上都确认一个 Master 一个 Backup。

### 3.4 DHCP 命令一览

| 命令 | 作用 | 使用视图 |
| --- | --- | --- |
| `dhcp enable` | 全局启用 DHCP 服务 | 系统视图 |
| `ip pool pool_vlan10` | 创建 / 进入全局地址池 | 系统视图 |
| `network 192.168.10.0 mask 255.255.255.0` | 指定可分配网段 | 地址池视图 |
| `gateway-list 192.168.10.254` | 下发网关地址 | 地址池视图 |
| `dns-list 172.16.1.100` | 下发 DNS 服务器 | 地址池视图 |
| `excluded-ip-address <起> <止>` | 排除不分配的地址（可选） | 地址池视图 |
| `lease day 1` | 设置租期（可选） | 地址池视图 |
| `dhcp select global` | 接口采用全局地址池分配 | Vlanif 接口视图 |
| `display ip pool name pool_vlan10` | 查看地址池使用情况 | 用户 / 系统视图 |

### 3.5 DHCP 逐条详解

**1. dhcp enable**

- 意思：全局开启 DHCP 服务（DHCP Server 功能）。
- 怎么用：系统视图下执行，且必须最先配。不开这条，后面的地址池、`dhcp select global` 都不会生效，PC 拿不到地址。

**2. ip pool pool_vlan10**

- 意思：创建一个名为 pool_vlan10 的全局地址池并进入其视图。全局地址池可被多个接口引用。
- 怎么用：系统视图；名称区分大小写，题目要求 pool_vlan10。

**3. network 192.168.10.0 mask 255.255.255.0**

- 意思：指定该地址池可分配的网段与掩码，DHCP 只会从这个范围里挑地址发给 PC。
- 怎么用：地址池视图；掩码也可写位数 `24`。一个地址池只能配一个网段。

**4. gateway-list 192.168.10.254**

- 意思：向 PC 下发默认网关地址。
- 怎么用：地址池视图；必须写 VRRP 虚拟 IP 192.168.10.254，不能写 SW1 的 192.168.10.1 或 SW2 的 192.168.10.2，否则主备切换后 PC 的网关失效。可配多个网关。

**5. dns-list 172.16.1.100**

- 意思：向 PC 下发 DNS 服务器地址，题目要求 172.16.1.100。
- 怎么用：地址池视图；可写多个，用空格分隔。

**6. excluded-ip-address \<起始 IP> \<结束 IP>（可选）**

- 意思：把某些地址从可分配范围中排除，避免与静态配置的网关、服务器地址冲突。
- 怎么用：地址池视图；VRRP 双机热备场景建议排除 192.168.10.1 和 192.168.10.2。

**7. lease day 1 hour 0 minute 0（可选）**

- 意思：设置地址租期，缺省一般为 1 天。
- 怎么用：地址池视图；`lease unlimited` 表示永久。

**8. dhcp select global**

- 意思：让该接口使用「全局地址池」方式为下挂 PC 分配地址。
- 怎么用：Vlanif 接口视图下执行，与地址池配套。另两种模式：`dhcp select interface`（用接口地址池，地址从接口所在网段取）、`dhcp select relay`（DHCP 中继，转发给外部服务器）。

**9. display ip pool name pool_vlan10**

- 意思：查看地址池总地址数、已用 / 空闲 / 冲突数量，以及每个已分配 IP 对应的 MAC，用来确认 PC 是否拿到地址。
- 怎么用：用户 / 系统视图；也可 `display ip pool` 看所有池、`display dhcp server statistics` 看报文统计。

### 3.6 完整配置示例（SW1 的 VLAN 10 部分）

```shell
system-view
dhcp enable
vlan batch 10 20
interface Vlanif 10
 ip address 192.168.10.1 255.255.255.0
 vrrp vrid 10 virtual-ip 192.168.10.254
 vrrp vrid 10 priority 120
 dhcp select global
quit
interface Vlanif 20
 ip address 192.168.20.1 255.255.255.0
 vrrp vrid 20 virtual-ip 192.168.20.254
 dhcp select global
quit
ip pool pool_vlan10
 network 192.168.10.0 mask 255.255.255.0
 gateway-list 192.168.10.254
 dns-list 172.16.1.100
 excluded-ip-address 192.168.10.1 192.168.10.2
quit
return
save
```

**SW2 与 SW1 的差异：**

- Vlanif 10：IP 改为 192.168.10.2，只配 `vrrp vrid 10 virtual-ip 192.168.10.254`，不配 priority 120（保持 100，成为 Backup）。
- Vlanif 20：IP 改为 192.168.20.2，在 `vrrp vrid 20 virtual-ip 192.168.20.254` 后追加 `vrrp vrid 20 priority 120`，成为 Master。
- 建议同样配置 pool_vlan10 地址池（或把地址范围切分），保证主备切换后 DHCP 服务不中断；PC 的网关务必始终写虚拟 IP。

## 四、任务 3：多区域 OSPF 与 RIPv2 双向引入

### 4.1 知识点回顾

R1 与 R2 的互联链路 10.1.1.0/30（R1 为 .1、R2 为 .2）属于 OSPF 进程 1、区域 0；R1 与边缘路由器 R0 运行 RIPv2，互联网段 10.2.2.0/24；在 R1 上做双向路由重分布，把 RIP 路由注入 OSPF、把 OSPF 路由引入 RIP；最后写出在 R0 上查看学到的路由表的命令。

### 4.2 OSPF 命令一览

| 命令 | 作用 | 使用视图 |
| --- | --- | --- |
| `ip address 10.1.1.1 255.255.255.252` | 配置互联接口地址 | 三层接口视图 |
| `ospf 1 router-id 1.1.1.1` | 启动 OSPF 进程并指定 Router ID | 系统视图 |
| `area 0` | 进入骨干区域视图 | OSPF 进程视图 |
| `network 10.1.1.0 0.0.0.3` | 在区域中宣告网段 | 区域视图 |
| `ospf enable 1 area 0` | 接口下启用 OSPF（等价写法） | 接口视图 |
| `display ospf peer` | 查看邻居状态 | 用户 / 系统视图 |
| `display ip routing-table protocol ospf` | 只看 OSPF 路由 | 用户 / 系统视图 |

### 4.3 OSPF 逐条详解

**1. ip address 10.1.1.1 255.255.255.252**

- 意思：给两台路由器互联的三层口配地址。/30 掩码即 255.255.255.252，一个网段只有 2 个可用地址，专用于点对点链路，最省地址。
- 怎么用：物理接口视图；R1 用 10.1.1.1，R2 用 10.1.1.2，掩码两端一致。也可写 `ip address 10.1.1.1 30`。

**2. ospf 1 router-id 1.1.1.1**

- 意思：启动 OSPF 进程 1，并手工指定 Router ID。进程号只在本设备有效（两端进程号可以不同），Router ID 必须全网唯一，否则邻居建不起来；通常取 Loopback 口地址。
- 怎么用：系统视图；不手工指定时设备自动选举（优先 Loopback 最大 IP，其次物理口最大 IP），但自动选举在地址变化后会变，建议手工写死。修改 Router ID 后需 `reset ospf process` 才生效。

**3. area 0**

- 意思：进入区域 0（骨干区域）的配置视图，可写成 `area 0.0.0.0`。OSPF 要求所有非骨干区域必须与骨干区域相连，多区域时非骨干写 `area 1`、`area 2`。
- 怎么用：OSPF 进程视图下执行，下一层才是宣告网段的命令。

**4. network 10.1.1.0 0.0.0.3**

- 意思：在区域内宣告网段。只有 IP 落在宣告范围内的接口才会运行 OSPF、收发 Hello 报文并把该网段洪泛出去。
- 怎么用：区域视图下执行，第二段是反掩码（不是正掩码）：/30 对应 `0.0.0.3`。这是最常见的错误点——写成 `255.255.255.252` 会匹配不到任何接口。也可用 `network 10.1.1.1 0.0.0.0` 精确宣告单个接口地址。

**5. ospf enable 1 area 0（等价写法）**

- 意思：直接在接口下使能 OSPF 并划入区域，效果与 network 宣告相同，新版本 VRP 更推荐，配置更直观、不易算错反掩码。
- 怎么用：物理接口视图下执行，进程号与区域号要和本设备 OSPF 进程一致。二选一即可，不要两种混用造成混淆。
- 举例：`interface GigabitEthernet 0/0/0` → `ospf enable 1 area 0`

**6. display ospf peer / display ospf peer brief**

- 意思：查看 OSPF 邻居，重点看 State 是否为 Full（完全邻接）。停在 ExStart / 2-Way 通常是 MTU、Router ID 冲突、区域号或认证不一致。
- 怎么用：用户 / 系统视图；`display ospf routing` 看 OSPF 进程算出的路由。

### 4.4 RIPv2 命令一览

| 命令 | 作用 | 使用视图 |
| --- | --- | --- |
| `rip 1` | 启动 RIP 进程 | 系统视图 |
| `version 2` | 切换到 RIPv2 | RIP 进程视图 |
| `network 10.0.0.0` | 宣告主类网络 | RIP 进程视图 |
| `undo summary` | 关闭自动汇总 | RIP 进程视图 |
| `rip version 2 multicast` | 接口以组播发送 RIPv2（可选） | 接口视图 |
| `display rip 1 route` | 查看 RIP 学到的路由 | 用户 / 系统视图 |

### 4.5 RIPv2 逐条详解

**1. rip 1**

- 意思：启动 RIP 进程 1 并进入 RIP 视图。RIP 是距离矢量协议，以跳数为度量，最大 15 跳，16 跳表示不可达。
- 怎么用：系统视图；进程号只在本设备有意义，R1 与 R0 可以都用 1 便于记忆。

**2. version 2**

- 意思：把 RIP 版本切到 RIPv2。v2 报文中携带子网掩码、支持 VLSM/CIDR、用组播 224.0.0.9 发送、支持认证；v1 只按主类网络发布，无法传递 /30 这类子网。
- 怎么用：RIP 进程视图下执行，全局生效；两端必须一致，否则收不到或解析错路由。也可在接口视图用 `rip version 2 multicast` 单独指定。

**3. network 10.0.0.0**

- 意思：宣告参与 RIP 的网段，被命中的接口才会发送 / 接收 RIP 更新。
- 怎么用：RIP 进程视图。与 OSPF 不同，RIP 的 network 只能写主类网络号、不能带掩码：10.2.2.0/24 属于 A 类 10.0.0.0/8，因此写 `network 10.0.0.0`。写 `network 10.2.2.0 0.0.0.255` 会报错。副作用是所有 10.x.x.x 接口都会启用 RIP（本题 R1 的 10.1.1.1 也在 10.0.0.0/8 内，要注意不要把 OSPF 侧网段意外引入，可配合 `undo rip input/output` 或策略过滤）。

**4. undo summary**

- 意思：关闭 RIPv2 的自动汇总（缺省开启时按主类网络边界汇总）。关掉后路由更新才会携带真实子网掩码。
- 怎么用：RIP 进程视图；在存在不连续子网或使用 VLSM 的网络中必须关闭，否则学到的路由会变成 10.0.0.0/8 这种大段，导致选路错误。

**5. display rip 1 route / display rip 1 database**

- 意思：查看 RIP 进程学到的路由及度量（跳数）。
- 怎么用：用户 / 系统视图；`display rip 1` 看进程全局参数（版本、是否汇总、定时器）。

### 4.6 双向路由引入（重分布）命令详解

**1. import-route rip 1**

- 意思：在 OSPF 进程中把 RIP 进程 1 学到的路由「重分布」进来，OSPF 以外部路由（ASE，Type-5 LSA）形式在区域内洪泛，区域 0 内的 R2 因此能学到 10.2.2.0/24。
- 怎么用：OSPF 进程视图（`ospf 1` 之后）执行。可附加参数控制开销与类型：`import-route rip 1 cost 10 type 1 tag 100`。缺省外部路由类型为 Type-2（只算引入时的 cost，不累加内部链路开销），缺省 cost 为 1。
- 举例：`ospf 1` → `import-route rip 1 cost 10 type 1`

**2. import-route ospf 1 cost 1**

- 意思：在 RIP 进程中把 OSPF 进程 1 的路由引入 RIP，RIP 以跳数形式发布给 R0，于是 R0 能学到 10.1.1.0/30 等 OSPF 侧网段。
- 怎么用：RIP 进程视图（`rip 1` 之后）执行。cost 指定引入后的初始跳数（1～15，16 不可达），必须保证不超过 15，否则 R0 认为不可达。
- 举例：`rip 1` → `import-route ospf 1 cost 1`

**3. 两条命令必须成对出现，且只在边界路由器 R1 上配**

- R1 是同时运行 OSPF 和 RIP 的「ASBR / 边界路由器」，双向引入只能在它上面做。只做 RIP→OSPF，则 R2 能学到 RIP 网段但 R0 学不到 OSPF 网段；题目明确要求双向，缺一不可。
- 引入时不会把对端协议的路由自动反灌回去，但多点双向引入可能产生环路与次优路径，实际工程中用 filter-policy、route-policy、tag 做过滤和防环。
- R0 自身也要运行 RIP：`rip 1` → `version 2` → `network 10.0.0.0`，接口配 10.2.2.x/24，否则收不到路由。

**4. 题目要求：在 R0 上查看学到的路由表的命令**

- 标准答案：`display ip routing-table`
- 只看 RIP 学到的：`display ip routing-table protocol rip`
- 判定成功的标志：路由表中出现带 R 标记、下一跳指向 R1（10.2.2.1）的 OSPF 侧网段，例如 10.1.1.0/30；这些条目在引入前是不存在的。
- 辅助命令：`display rip 1 route`、`ping 10.1.1.1`。

### 4.7 完整配置示例

**R1（边界路由器，双向引入）：**

```shell
system-view
sysname R1
interface GigabitEthernet 0/0/0
 ip address 10.1.1.1 30
quit
interface GigabitEthernet 0/0/1
 ip address 10.2.2.1 24
quit
ospf 1 router-id 1.1.1.1
 area 0
  network 10.1.1.0 0.0.0.3
 quit
 import-route rip 1 cost 10 type 1
quit
rip 1
 version 2
 undo summary
 network 10.0.0.0
 import-route ospf 1 cost 1
quit
return
save
```

**R2（OSPF 侧）：**

```shell
interface GigabitEthernet 0/0/0
 ip address 10.1.1.2 30
quit
ospf 1 router-id 2.2.2.2
 area 0
  network 10.1.1.0 0.0.0.3
```

**R0（RIP 侧，题目验证点）：**

```shell
interface GigabitEthernet 0/0/0
 ip address 10.2.2.2 24
quit
rip 1
 version 2
 undo summary
 network 10.0.0.0
```

验证：`display ip routing-table`

## 五、任务 4：路由器单臂路由与高级 ACL

### 5.1 知识点回顾

R3 下连接入交换机，包含 VLAN 50（192.168.50.0/24）与 VLAN 60（192.168.60.0/24）；在 R3 的 GE 0/0/1 上配置子接口 GE 0/0/1.50 与 GE 0/0/1.60 作为对应 VLAN 的网关（IP 分别为 .254）；使用高级 ACL 3000 拒绝 VLAN 50 网段中的 PC 访问 VLAN 60 网段的任何流量，允许其他流量通行，并把 ACL 应用在正确的子接口及流向上。

### 5.2 单臂路由命令一览

| 命令 | 作用 | 使用视图 |
| --- | --- | --- |
| `interface GigabitEthernet 0/0/1.50` | 创建 / 进入子接口 | 系统视图 |
| `dot1q termination vid 50` | 子接口终结 VLAN 50 标签 | 子接口视图 |
| `ip address 192.168.50.254 255.255.255.0` | 配置该 VLAN 网关 | 子接口视图 |
| `arp broadcast enable` | 允许子接口发送 ARP 广播 | 子接口视图 |
| `undo portswitch` | 二层口切三层口（视型号可选） | 物理接口视图 |
| `display ip interface brief` | 查看接口 IP 与状态 | 用户 / 系统视图 |

### 5.3 单臂路由逐条详解

**1. interface GigabitEthernet 0/0/1.50**

- 意思：在物理口 GE 0/0/1 上创建编号为 50 的子接口（范围通常 1～4096），一个物理口可拆出多个子接口，每个子接口当一个 VLAN 的网关，这就是「单臂路由」——用一条物理链路承载多个 VLAN 的三层转发。编号习惯与 VLAN ID 保持一致，便于记忆，但不是强制要求。
- 怎么用：系统视图下执行；物理主接口本身不配 IP，只需 `undo shutdown`（若型号默认二层，先 `undo portswitch` 切成三层口）。

**2. dot1q termination vid 50**

- 意思：配置子接口对 802.1Q 标签的「终结」：只处理带 VLAN 50 标签的帧，收到后剥掉标签做三层转发，发出时再打回 VLAN 50 标签。这是单臂路由能区分不同 VLAN 的核心命令。
- 怎么用：子接口视图；vid 必须与交换机 trunk 上放行、以及 PC 所属 VLAN 一致。每个子接口只能终结一个 VLAN（若需终结多个，用 `dot1q termination vid 50 to 60` 配合 VLAN 组，或用 Vlanif）。

**3. ip address 192.168.50.254 255.255.255.0**

- 意思：给子接口配网关 IP，题目要求 VLAN 50 用 192.168.50.254、VLAN 60 用 192.168.60.254。
- 怎么用：子接口视图；PC 的默认网关必须指向对应子接口的这个地址。

**4. arp broadcast enable**

- 意思：允许子接口主动发送 ARP 广播报文。
- 怎么用：子接口视图。华为设备上这条必配：缺省情况下子接口不会主动发 ARP 请求，PC 无法学到网关 MAC，现象是「PC ping 不通自己的网关」。这是单臂路由实验最高频的踩坑点，两个子接口都要配。

**5. 接入交换机侧配套命令**

```shell
vlan batch 50 60
interface GigabitEthernet 0/0/1
 port link-type trunk
 port trunk allow-pass vlan 50 60
```

接 VLAN 50 PC 的口：`port link-type access` → `port default vlan 50`；VLAN 60 同理。

> [!WARNING]
> trunk 两端放行的 VLAN 必须包含 50 和 60，否则对应 VLAN 不通。

### 5.4 高级 ACL 命令一览

| 命令 | 作用 | 使用视图 |
| --- | --- | --- |
| `acl number 3000` | 创建高级 ACL（3000～3999） | 系统视图 |
| `rule 5 deny ip source <源网段> <反掩码> destination <目的网段> <反掩码>` | 拒绝指定流量 | ACL 视图 |
| `rule 10 permit ip` | 放行其余流量 | ACL 视图 |
| `traffic-filter inbound acl 3000` | 在接口入方向应用 ACL | 子接口视图 |
| `display acl 3000` | 查看规则与命中次数 | 用户 / 系统视图 |
| `display traffic-filter applied-record` | 查看 ACL 应用位置与方向 | 用户 / 系统视图 |

### 5.5 高级 ACL 逐条详解

**1. acl number 3000（可简写 acl 3000）**

- 意思：创建编号为 3000 的高级 ACL 并进入 ACL 视图。ACL 编号范围：2000～2999 为基本 ACL（只能匹配源 IP），3000～3999 为高级 ACL（可匹配源 IP、目的 IP、协议类型、源/目的端口、DSCP 等）。题目要求「拒绝 VLAN 50 访问 VLAN 60」，必须同时匹配源和目的，因此只能用高级 ACL。
- 怎么用：系统视图；也可用命名 ACL `acl name <名称> advance`。编号 3000 是题目指定值。

**2. rule 5 deny ip source 192.168.50.0 0.0.0.255 destination 192.168.60.0 0.0.0.255**

- 意思：写一条拒绝规则。逐段拆解：`rule 5` 是规则编号（步长 5，编号越小越先匹配，匹配到就停止，不再看后面的规则）；`deny` 表示丢弃；`ip` 表示匹配所有 IP 协议流量（也可写 tcp、udp、icmp 并追加端口号，实现更精细控制）；`source 192.168.50.0 0.0.0.255` 用反掩码表示整个 192.168.50.0/24 网段；`destination 192.168.60.0 0.0.0.255` 表示目的为 192.168.60.0/24。合起来即「禁止 VLAN 50 的 PC 访问 VLAN 60 的任何流量」。
- 怎么用：ACL 视图下执行。反掩码 `0.0.0.255` 对应 /24，不能写成正掩码；`source any` / `destination any` 可表示任意地址。规则编号建议手动写 5、10、15，便于日后在中间插入新规则。

**3. rule 10 permit ip**

- 意思：放行所有其他 IP 流量，满足题目「允许其他流量通行」。
- 怎么用：ACL 视图，编号要大于拒绝规则。虽然 traffic-filter 对未匹配任何规则的流量缺省是允许（permit），但显式写出这条更严谨，也方便通过命中计数观察放行了多少包。

**4. traffic-filter inbound acl 3000**

- 意思：把 ACL 3000 应用到接口的入方向。inbound 指流量「进入本路由器接口」时检查，outbound 指「从本接口发出」时检查。
- 怎么用：子接口视图下执行。题目要求「正确的子接口及流向」，最合理的是应用在 GigabitEthernet 0/0/1.50 的 inbound：VLAN 50 的 PC 发出的报文一进入路由器就被检查并丢弃，既不消耗后续转发资源，也不会误伤去往其他网段的正常流量和回程报文。
- 另一种等效做法：在 GigabitEthernet 0/0/1.60 上用 `traffic-filter outbound acl 3000`（在流量离开、发往 VLAN 60 时检查），效果相同但要在出方向检查所有流量，通常不作为首选。
- 注意：同一接口、同一方向只能绑定一个 ACL；ACL 只过滤「经过设备转发」的流量，不过滤设备自身产生的流量；应用后建议用 `display traffic-filter applied-record` 核对。
- 举例：`interface GigabitEthernet 0/0/1.50` → `traffic-filter inbound acl 3000`

**5. 验证命令**

- `display acl 3000`：查看规则内容与命中计数（matched 次数），确认拒绝规则确实在被匹配。
- `display acl all`：查看全部 ACL。
- `display traffic-filter applied-record`：查看 ACL 被应用在哪个接口、哪个方向。
- `ping -a 192.168.50.1 192.168.60.1`：应不通；`ping -a 192.168.50.1 <其他网段地址>`：应通。

### 5.6 完整配置示例（R3）

```shell
system-view
sysname R3
interface GigabitEthernet 0/0/1
 undo shutdown
quit
interface GigabitEthernet 0/0/1.50
 dot1q termination vid 50
 ip address 192.168.50.254 255.255.255.0
 arp broadcast enable
quit
interface GigabitEthernet 0/0/1.60
 dot1q termination vid 60
 ip address 192.168.60.254 255.255.255.0
 arp broadcast enable
quit
acl number 3000
 rule 5 deny ip source 192.168.50.0 0.0.0.255 destination 192.168.60.0 0.0.0.255
 rule 10 permit ip
quit
interface GigabitEthernet 0/0/1.50
 traffic-filter inbound acl 3000
quit
return
save
```

接入交换机侧：`vlan batch 50 60`；上联口 `port link-type trunk` + `port trunk allow-pass vlan 50 60`；接 PC 的 access 口分别 `port default vlan 50` / `port default vlan 60`。

## 六、验证命令速查表

配完每条任务后用这些命令核对结果，全部在用户视图（或系统视图）执行。

| 命令 | 看什么 | 在哪台设备上用 |
| --- | --- | --- |
| `display current-configuration` | 全部生效配置，查漏敲 / 敲错 | 任意设备 |
| `display this` | 当前视图下已配的命令 | 任意子视图 |
| `display ip interface brief` | 接口 IP、物理 / 协议状态是否 up | 任意设备 |
| `display vlan` | VLAN 是否创建、端口是否加入 | 交换机 |
| `display eth-trunk 1` | 聚合模式、成员口、活动链路数 | SW1、SW2 |
| `display lacp statistics eth-trunk 1` | LACP 报文收发是否正常 | SW1、SW2 |
| `display stp region-configuration` | MST 域名、修订级别、VLAN-实例映射是否生效 | 全部交换机 |
| `display stp instance 1 brief` | 实例 1 端口角色与阻塞状态 | 全部交换机 |
| `display vrrp` | Master / Backup 状态、虚拟 IP、优先级 | SW1、SW2 |
| `display ip pool name pool_vlan10` | 地址池使用情况、已分配地址 | SW1、SW2 |
| `display ospf peer brief` | OSPF 邻居是否为 Full | R1、R2 |
| `display ospf routing` | OSPF 进程算出的路由（含引入的外部路由） | R1、R2 |
| `display rip 1 route` | RIP 学到的路由与跳数 | R1、R0 |
| `display ip routing-table` | 完整路由表（题目要求的 R0 验证命令） | 任意，重点 R0 |
| `display ip routing-table protocol rip` | 只看 RIP 路由，确认 OSPF 网段已引入 | R0 |
| `display acl 3000` | 规则内容与命中计数 | R3 |
| `display traffic-filter applied-record` | ACL 应用在哪个接口、哪个方向 | R3 |
| `display interface brief` | 端口 up/down、速率双工 | 任意设备 |
| `ping -a <源 IP> <目的 IP>` | 指定源地址的连通性测试 | 任意设备 |
| `save` | 保存配置，防止重启丢失 | 任意设备（用户视图） |

## 附录 A：高频易错点清单

1. MSTP 配完域三要素后忘记执行 `active region-configuration`，配置不生效。
2. 各交换机的域名、修订级别、VLAN-实例映射不完全一致，被判定为不同 MST 域，负载分担失效。
3. 物理口加入 Eth-Trunk 前没清空已有配置，或 `mode lacp-static` 在成员口加入之后才配，导致报错。
4. 忘记全局 `lacp priority 4096` 配在了接口视图（应在系统视图），主动端选错。
5. DHCP 忘记先 `dhcp enable`；或 `gateway-list` 写成交换机真实 IP 而不是 VRRP 虚拟 IP，主备切换后 PC 断网。
6. OSPF `network` 用了正掩码而不是反掩码（/30 应为 `0.0.0.3`），接口没被宣告、邻居建不起来。
7. RIP 的 `network` 误写成 `10.2.2.0` 或带掩码；RIP 只认主类网络，应写 `network 10.0.0.0`，并记得 `version 2` + `undo summary`。
8. 双向引入只做了一半（只 RIP→OSPF），R0 学不到 OSPF 侧路由；或 `import-route ospf 1` 的 cost 设得过大导致跳数超限。
9. 子接口忘记 `arp broadcast enable`，PC ping 不通自己的网关；主接口误配了 IP。
10. ACL 规则序号没留余量、忘记写 `rule permit ip`，或应用在错误的接口 / 方向上；反掩码写成正掩码。
11. 配置完只在内存里，忘记在用户视图执行 `save`，重启后全部丢失。

## 附录 B：课堂复习答案中的补充命令与写法差异

同目录下的[《复习答案 - 第二节课（完善版）》](/posts/fuxi-daian-dier-jieke-wanshanban/)是完整拓扑的课堂配置：SW1/SW2 上 VLAN 10/20/30/40 的 Vlanif 地址分别为 192.168.37.x、192.168.42.x、192.168.47.x、192.168.52.x，AR6/AR2/AR3/AR4 之间用 10.x.x.x/30 互联，单臂路由做在 AR4 的 GE0/0/2.1 与 .2 上，VRRP 虚拟网关为 .254，DHCP 地址池为 pool_vlan40，ACL 3000 同样禁止 VLAN 50 访问 VLAN 60。样题是它的简化版。两套拓扑的命令体系完全相同，只是 IP、VLAN 号、接口号等参数不同：把本手册里的参数替换成课堂拓扑的数值即可直接套用。

### B.1 补充命令（课堂答案中出现、前文未单列）

**1. undo info-center enable**

- 意思：关闭信息中心的日志、调试与告警提示。配置过程中设备会不断弹出接口 Up/Down、协议状态变化等消息，容易打断输入。
- 怎么用：系统视图下执行，通常是实验 / 考试开局的第一步；需要恢复时用 `info-center enable`。它只关闭屏幕提示，不影响任何业务功能。

**2. port trunk allow-pass vlan 2 to 4094**

- 意思：一次性放行 VLAN 2 到 4094 的连续范围，等价于放行几乎所有 VLAN。
- 怎么用：trunk 接口视图；`to` 表示连续区间，比逐个列举更简洁；`port trunk allow-pass vlan all` 效果相同。课堂答案在所有上联 trunk 口统一使用这条。

**3. display stp brief**

- 意思：一览所有生成树实例的端口角色与状态，快速定位被阻塞的端口。
- 怎么用：用户 / 系统视图；相比 `display stp instance 1 brief`（只看单个实例）更概览。

**4. display vrrp brief**

- 意思：以简表形式查看所有 VRRP 备份组的状态、虚拟 IP 与优先级。
- 怎么用：用户 / 系统视图；`display vrrp` 显示更详细的信息。

**5. display ospf peer brief**

- 意思：简要查看 OSPF 邻居状态是否达到 Full。
- 怎么用：用户 / 系统视图；完整信息用 `display ospf peer`。

### B.2 同一命令的不同写法（效果相同，考试中均可）

| 本手册写法 | 课堂答案写法 | 说明 |
| --- | --- | --- |
| `interface Eth-Trunk 1` | `interface Eth-Trunk1` | 接口名与编号间的空格可省略，VRP 都能识别 |
| `acl number 3000` | `acl 3000` | 创建高级 ACL 的完整写法与简写 |
| `area 0` | `area 0.0.0.0` | 骨干区域的点分十进制写法 |
| `ospf 1 router-id 1.1.1.1` | `ospf 1` | Router ID 可省略由设备自动选举，生产环境建议手工指定 |
| `import-route ospf 1 cost 1` | `import-route ospf 1` | cost 可省略，缺省为 1 跳 |
| `vrrp vrid 10 preempt-mode timer delay 5` | `vrrp vrid 10 preempt-mode timer delay 20` | 抢占延时秒数按需设定，课堂答案用 20 秒 |
| `vrrp vrid 10 track interface ... reduced 30` | `vrrp vrid 10 track interface GigabitEthernet0/0/24` | 不写 reduced 时按缺省值 10 降低优先级 |
| `ip address 10.1.1.1 30` | `ip address 10.1.1.1 255.255.255.252` | 掩码写位数或点分十进制均可 |
| `interface GigabitEthernet 0/0/1.50` | `interface GigabitEthernet0/0/2.1` | 子接口编号不必与 VLAN ID 相同，靠 `dot1q termination vid` 建立对应 |

### B.3 课堂答案命令总清单（按出现顺序，便于逐个检索）

- **基础与接口**：`sysname`、`undo info-center enable`、`vlan batch`、`interface Vlanif <ID>`、`ip address`、`interface GigabitEthernet <编号>`、`port link-type trunk`、`port trunk allow-pass vlan 2 to 4094`、`port link-type access`、`port default vlan`。
- **单臂路由**：`interface GigabitEthernet0/0/2.1`、`dot1q termination vid 50`、`arp broadcast enable`（.2 对应 vid 60）。
- **链路聚合与 MSTP**：`lacp priority 4096`、`interface Eth-Trunk1`、`mode lacp-static`、`eth-trunk 1`、`stp mode mstp`、`stp region-configuration`（region-name、revision-level、instance 1 vlan ...、active region-configuration）、`stp instance 1 root primary`、`stp instance 2 root secondary`。
- **VRRP**：`vrrp vrid 10 virtual-ip ...`、`vrrp vrid 10 priority 120`、`vrrp vrid 10 preempt-mode timer delay 20`、`vrrp vrid 10 track interface ...`（VLAN 20/30/40 同理，SW1 与 SW2 主备角色对调）。
- **DHCP**：`dhcp enable`、`ip pool pool_vlan40`、`network ... mask 255.255.255.0`、`gateway-list`、`dns-list`、`dhcp select global`。
- **OSPF 与 RIP**：`ospf 1`、`area 0.0.0.0` / `area 0.0.0.1`、`network <网段> <反掩码>`、`rip 1`、`version 2`、`undo summary`、`network 10.0.0.0`、`import-route ospf 1`、`import-route rip 1`。
- **ACL 与验证**：`acl 3000`、`rule 5 deny ip source ... destination ...`、`rule 10 permit ip`、`traffic-filter inbound acl 3000`、`display eth-trunk 1`、`display stp brief`、`display vrrp brief`、`display ospf peer brief`、`display ip routing-table protocol rip`、`display ip routing-table`。

以上命令的含义与用法，均已在第一章至第六章逐条说明。
