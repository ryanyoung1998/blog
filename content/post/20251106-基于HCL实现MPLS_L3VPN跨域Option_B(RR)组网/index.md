---
title: 基于HCL实现MPLS L3VPN跨域Option B (RR)组网
description: 使用H3C Cloud Lab实现MPLS L3VPN跨域Option B (RR)组网
date: 2025-11-06
slug: mpls_l3vpn_opton_b
image: https://ryan1998.dpdns.org/cover/post/mpls/cover-mpls-l3vpn-option-b.png
categories:
    - Datacom
    - MPLS L3VPN
tags:
    - Inter-AS
    - Option B
    - HCL
---

## 一、组网规划

![组网拓扑图](https://ryan1998.dpdns.org/illus/mpls-l3vpn-inter-as-option-b/00-topo.png)


如图所示：

- 全局使用 ISIS 作为公网 IGP 协议

- 全局使用 MPLS + LDP 处理标签分发与转发

- RR 与 PE(含ASBR) 之间建立 MP-iBGP 传递 VPNv4 路由

- ASBR 之间建立 **MP-eBGP** 传递 VPNv4 路由

- PE 与 CE 之间使用 IGP 传递 VPNv4 路由

---

## 二、公网地址规划与配置

### 1. 配置Loopback地址、全局Router-id、全局LSR-id

> Loopback地址编址规范（仅实验环境）：AS.0.0.x/32

| AS  | 设备     | 地址          | AS  | 设备     | 地址          |
| --- | ------ | ----------- | --- | ------ | ----------- |
| 10  | RR_1   | 10.0.0.1/32 | 20  | RR_2   | 20.0.0.1/32 |
| 10  | ASBR_1 | 10.0.0.2/32 | 20  | ASBR_2 | 20.0.0.2/32 |
| 10  | PE_1   | 10.0.0.3/32 | 20  | PE_3   | 20.0.0.3/32 |
| 10  | PE_2   | 10.0.0.4/32 | 20  | PE_4   | 20.0.0.4/32 |
| 10  | P_1    | 10.0.0.5/32 | 20  | P_2    | 20.0.0.5/32 |
| 10  | CE_1   | 10.0.0.6/32 | 20  | CE_3   | 20.0.0.6/32 |
| 10  | CE_2   | 10.0.0.7/32 | 20  | CE_4   | 20.0.0.7/32 |

#### AS 10

- RR_1
```cmd
#
interface LoopBack0
 ip address 10.0.0.1 32
#
 router id 10.0.0.1
#
 mpls lsr-id 10.0.0.1
#
mpls ldp
#
```

- ASBR_1
```cmd
#
interface LoopBack0
 ip address 10.0.0.2 32
#
 router id 10.0.0.2
#
 mpls lsr-id 10.0.0.2
#
mpls ldp
#
```

- PE_1
```cmd
#
interface LoopBack0
 ip address 10.0.0.3 32
#
 router id 10.0.0.3
#
 mpls lsr-id 10.0.0.3
#
mpls ldp
#
```

- PE_2
```cmd
#
interface LoopBack0
 ip address 10.0.0.4 32
#
 router id 10.0.0.4
#
 mpls lsr-id 10.0.0.4
#
mpls ldp
#
```

- P_1
```cmd
#
interface LoopBack0
 ip address 10.0.0.5 32
#
 router id 10.0.0.5
#
 mpls lsr-id 10.0.0.5
#
mpls ldp
#
```

- CE_1
```cmd
#
interface LoopBack0
 ip address 10.0.0.6 32
#
 router id 10.0.0.6
#
```

- CE_2
```cmd
#
interface LoopBack0
 ip address 10.0.0.7 32
#
 router id 10.0.0.7
#
```


#### AS 20
- RR_2
```cmd
#
interface LoopBack0
 ip address 20.0.0.1 32
#
 router id 20.0.0.1
#
 mpls lsr-id 20.0.0.1
#
mpls ldp
#
```

- ASBR_2
```cmd
#
interface LoopBack0
 ip address 20.0.0.2 32
#
 router id 20.0.0.2
#
 mpls lsr-id 20.0.0.2
#
mpls ldp
#
```

- PE_3
```cmd
#
interface LoopBack0
 ip address 20.0.0.3 32
#
 router id 20.0.0.3
#
 mpls lsr-id 20.0.0.3
#
mpls ldp
#
```

- PE_4
```cmd
#
interface LoopBack0
 ip address 20.0.0.4 32
#
 router id 20.0.0.4
#
 mpls lsr-id 20.0.0.4
#
mpls ldp
#
```

- P_2
```cmd
#
interface LoopBack0
 ip address 20.0.0.5 32
#
 router id 20.0.0.5
#
 mpls lsr-id 20.0.0.5
#
mpls ldp
#
```

- CE_3
```cmd
#
interface LoopBack0
 ip address 20.0.0.6 32
#
 router id 20.0.0.6
#
```

- CE_4
```cmd
#
interface LoopBack0
 ip address 20.0.0.7 32
#
 router id 20.0.0.7
#
```

---

### 2. 配置AS内部互联地址

> 互联地址编制规范（仅实验环境）： AS.0.AZ.x/24

#### AS 10

| AS | A端设备 | A端接口 | A端地址 | Z端地址 | Z端接口 | Z端地址 |
| --- | ------ | ------- | ------------- | ------------- | ------- | ------ |
| 10 | RR_1 | GE0/0/0 | 10.0.15.1/24 | 10.0.15.5/24 | GE0/0/0 | P_1 |
| 10 | ASBR_1 | GE0/0/0 | 10.0.25.1/24 | 10.0.25.5/24 | GE0/0/1 | P_1 |
| 10 | PE_1 | GE0/0/0 | 10.0.35.3/24 | 10.0.35.5/24 | GE0/0/7 | P_1 |
| 10 | PE_2 | GE0/0/0 | 10.0.45.4/24 | 10.0.45.5/24 | GE0/0/8 | P_1 |
| 10 | CE_1 | GE0/0/0 | 10.0.36.3/24 | 10.0.36.6/24 | GE0/0/1 | PE_1 |
| 10 | CE_2 | GE0/0/0 | 10.0.47.4/24 | 10.0.47.7/24 | GE0/0/1 | PE_2 |

- RR_1
```cmd
#
 interface GigabitEthernet0/0/0
  description TO P_1
  ip address 10.0.15.1 24
#
```

- ASBR_1
```cmd
#
 interface GigabitEthernet0/0/0
  description TO P_1
  ip address 10.0.25.2 24
#
```

- PE_1
```cmd
#
 interface GigabitEthernet0/0/0
  description TO P_1
  ip address 10.0.35.3 24
#
 interface GigabitEthernet0/0/1
  description TO CE_1
  ip address 10.0.36.3 24
#
```

- PE_2
```cmd
#
 interface GigabitEthernet0/0/0
  description TO P_1
  ip address 10.0.45.4 24
#
 interface GigabitEthernet0/0/1
  description TO CE_2
  ip address 10.0.47.4 24
#
```

- P_1
```cmd
#
 interface GigabitEthernet0/0/0
  description TO RR_1
  ip address 10.0.15.5 24
#
 interface GigabitEthernet0/0/1
  description TO ASBR_1
  ip address 10.0.25.5 24
#
 interface GigabitEthernet0/0/7
  description TO PE_1
  ip address 10.0.35.5 24
#
 interface GigabitEthernet0/0/8
  description TO PE_2
  ip address 10.0.45.5 24
#
```

- CE_1
```cmd
#
 interface GigabitEthernet0/0/0
  description TO PE_1
  ip address 10.0.36.6 24
#
```

- CE_2
```cmd
#
 interface GigabitEthernet0/0/0
  description TO PE_2
  ip address 10.0.47.7 24
#
```

#### AS 20

| AS | A端设备 | A端接口 | A端地址 | Z端地址 | Z端接口 | Z端地址 |
| --- | ------ | ------- | ------------- | ------------- | ------- | ------ |
| 20 | RR_2 | GE0/0/0 | 20.0.15.1/24 | 20.0.15.5/24 | GE0/0/0 | P_2 |
| 20 | ASBR_2 | GE0/0/0 | 20.0.25.1/24 | 20.0.25.5/24 | GE0/0/1 | P_2 |
| 20 | PE_3 | GE0/0/0 | 20.0.35.3/24 | 20.0.35.5/24 | GE0/0/7 | P_2 |
| 20 | PE_4 | GE0/0/0 | 20.0.45.4/24 | 20.0.45.5/24 | GE0/0/8 | P_2 |
| 20 | CE_3 | GE0/0/0 | 20.0.36.3/24 | 20.0.36.6/24 | GE0/0/1 | PE_3 |
| 20 | CE_4 | GE0/0/0 | 20.0.47.4/24 | 20.0.47.7/24 | GE0/0/1 | PE_4 |

- RR_2
```cmd
#
 interface GigabitEthernet0/0/0
  description TO P_2
  ip address 20.0.15.1 24
#
```

- ASBR_2
```cmd
#
 interface GigabitEthernet0/0/0
  description TO P_2
  ip address 20.0.25.2 24
#
```

- PE_3
```cmd
#
 interface GigabitEthernet0/0/0
  description TO P_2
  ip address 20.0.35.3 24
#
 interface GigabitEthernet0/0/1
  description TO CE_3
  ip address 20.0.36.3 24
#
```

- PE_4
```cmd
#
 interface GigabitEthernet0/0/0
  description TO P_2
  ip address 20.0.45.4 24
#
 interface GigabitEthernet0/0/1
  description TO CE_4
  ip address 20.0.47.4 24
#
```

- P_2
```cmd
#
 interface GigabitEthernet0/0/0
  description TO RR_2
  ip address 20.0.15.5 24
#
 interface GigabitEthernet0/0/1
  description TO ASBR_2
  ip address 20.0.25.5 24
#
 interface GigabitEthernet0/0/7
  description TO PE_3
  ip address 20.0.35.5 24
#
 interface GigabitEthernet0/0/8
  description TO PE_4
  ip address 20.0.45.5 24
#
```

- CE_3
```cmd
#
 interface GigabitEthernet0/0/0
  description TO PE_3
  ip address 20.0.36.6 24
#
```

- CE_4
```cmd
#
 interface GigabitEthernet0/0/0
  description TO PE_4
  ip address 20.0.47.7 24
#
```

### 3. 配置AS之间互联地址

> 互联地址编址规范（仅实验环境）： AS1.AS2.0.ASx/24

| A端AS | A端设备 | A端接口 | A端地址 | Z端地址 | Z端接口 | Z端设备 | Z端AS |
| --- | ------ | -------- | ---------------- | ---------------- | -------- | ------ | --- |
| 10 | ASBR_1 | GE0/0/10 | 10.20.0.10/24 | 10.20.0.20/24 | GE0/0/10 | ASBR_2 | 20 |

#### AS 10

- ASBR_1
```cmd
#
 interface GigabitEthernet0/0/10
  description TO AS20_ASBR_2
  ip address 10.20.0.10 24
#
```

#### AS 20

- ASBR_2
```cmd
#
 interface GigabitEthernet0/0/10
  description TO AS10_ASBR_1
  ip address 10.20.0.20 24
#
```


## 三、配置公网IGP、LDP协议

### 1. 配置公网ISIS进程

> 公网ISIS网络实体编址规范（仅实验环境）：AS.0000.LOOPBACK.00

#### AS 10

- RR_1
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0001.00
#
```

- ASBR_1
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0002.00
#
```

- PE_1
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0003.00
#
```

- PE_2
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0004.00
#
```

- P_1
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0005.00
#
```

- CE_1
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0006.00
#
```

- CE_2
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0007.00
#
```


#### AS 20

- RR_2
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0001.00
#
```

- ASBR_2
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0002.00
#
```

- PE_3
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0003.00
#
```

- PE_4
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0004.00
#
```

- P_2
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0005.00
#
```

- CE_3
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0006.00
#
```

- CE_4
```cmd
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0007.00
#
```

### 2. 在互联接口启用ISIS


#### AS 10

- RR_1
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
```

- ASBR_1
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
```

- PE_1
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
 interface GigabitEthernet0/0/1
  isis enable 1
#
```

- PE_2
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
 interface GigabitEthernet0/0/1
  isis enable 1
#
```

- P_1
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
 interface GigabitEthernet0/0/1
  isis enable 1
#
 interface GigabitEthernet0/0/7
  isis enable 1
#
 interface GigabitEthernet0/0/8
  isis enable 1
#
```

- CE_1
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
```

- CE_2
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
```

#### AS 20

- RR_2
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
```

- ASBR_2
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
```

- PE_3
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
 interface GigabitEthernet0/0/1
  isis enable 1
#
```

- PE_4
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
 interface GigabitEthernet0/0/1
  isis enable 1
#
```

- P_2
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
 interface GigabitEthernet0/0/1
  isis enable 1
#
 interface GigabitEthernet0/0/7
  isis enable 1
#
 interface GigabitEthernet0/0/8
  isis enable 1
#
```

- CE_3
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
```

- CE_4
```cmd
#
 interface loopback 0
  isis enable 1
#
 interface GigabitEthernet0/0/0
  isis enable 1
#
```

> ISIS配置完成后，可验证ISIS邻居状态及路由学习情况
> 
> `display isis peer`
> 
> `display ip routing-table protocol isis`


### 3. 在互联接口启用MPLS LDP 

> 因PE与CE之间使用IGP传递VPNv4路由，所以PE与CE互联接口不需要开启mpls和ldp

#### AS 10

- RR_1
```cmd
#
 interface GigabitEthernet0/0/0
  mpls enable
  mpls ldp enable
#
```

- ASBR_1
```cmd
#
 interface GigabitEthernet0/0/0
  mpls enable
  mpls ldp enable
#
 interface GigabitEthernet0/0/10
  mpls enable
  mpls ldp enable
#
```

- PE_1
```cmd
#
 interface GigabitEthernet0/0/0
  mpls enable
  mpls ldp enable
#
```

- PE_2
```cmd
#
 interface GigabitEthernet0/0/0
  mpls enable
  mpls ldp enable
#
```

- P_1
```cmd
#
 interface GigabitEthernet0/0/0
  mpls enable
  mpls ldp enable
#
 interface GigabitEthernet0/0/1
  mpls enable
  mpls ldp enable
#
 interface GigabitEthernet0/0/7
  mpls enable
  mpls ldp enable
#
 interface GigabitEthernet0/0/8
  mpls enable
  mpls ldp enable
#
```

#### AS 20

- RR_2
```cmd
#
 interface GigabitEthernet0/0/0
  mpls enable
  mpls ldp enable
#
```

- ASBR_2
```cmd
#
 interface GigabitEthernet0/0/0
  mpls enable
  mpls ldp enable
#
 interface GigabitEthernet0/0/10
  mpls enable
  mpls ldp enable
#
```

- PE_3
```cmd
#
 interface GigabitEthernet0/0/0
  mpls enable
  mpls ldp enable
#
```

- PE_4
```cmd
#
 interface GigabitEthernet0/0/0
  mpls enable
  mpls ldp enable
#
```

- P_2
```cmd
#
 interface GigabitEthernet0/0/0
  mpls enable
  mpls ldp enable
#
 interface GigabitEthernet0/0/1
  mpls enable
  mpls ldp enable
#
 interface GigabitEthernet0/0/7
  mpls enable
  mpls ldp enable
#
 interface GigabitEthernet0/0/8
  mpls enable
  mpls ldp enable
#
```

> MPLS、LDP配置完成后，可验证LDP邻居状态
> 
> `display mpls ldp peer`


## 四、配置MP-BGP传递VPNv4路由

### 1. 配置AS域内的MP-iBGP

> 1. 因为 RR 和 ASBR 上不接业务，也不配置 VPN 实例，所以需要在 VPNv4地址簇下配置
> 
>    ` undo policy vpn-target`
> 
> 2. 因为AS域内的RR和PE均没有ASBR之间互联地址的路由，所以需要在ASBR侧 VPNv4地址簇下配置
> 
>    `peer RR next-hop-local`

#### AS 10

- RR_1
```cmd
#
bgp 10
 router-id 10.0.0.1
 group PE internal
 peer PE connect-interface LoopBack0
 peer 10.0.0.2 group PE
 peer 10.0.0.3 group PE
 peer 10.0.0.4 group PE
 #
 address-family vpnv4
  undo policy vpn-target
  peer PE enable
  peer PE reflect-client
#
```

- ASBR_1
```cmd
#
bgp 10
 router-id 10.0.0.2
 group RR internal
 peer RR connect-interface LoopBack0
 peer 10.0.0.1 group RR
 #
 address-family vpnv4
  undo policy vpn-target
  peer RR enable
  peer RR next-hop-local
#
```

- PE_1
```cmd
#
bgp 10
 router-id 10.0.0.3
 group RR internal
 peer RR connect-interface LoopBack0
 peer 10.0.0.1 group RR
 #
 address-family vpnv4
  peer RR enable
#
```

- PE_2
```cmd
#
bgp 10
 router-id 10.0.0.4
 group RR internal
 peer RR connect-interface LoopBack0
 peer 10.0.0.1 group RR
 #
 address-family vpnv4
  peer RR enable
#
```


#### AS 20

- RR_2
```cmd
#
bgp 20
 router-id 20.0.0.1
 group PE internal
 peer PE connect-interface LoopBack0
 peer 20.0.0.2 group PE
 peer 20.0.0.3 group PE
 peer 20.0.0.4 group PE
 #
 address-family vpnv4
  undo policy vpn-target
  peer PE enable
  peer PE reflect-client
#
```

- ASBR_2
```cmd
#
bgp 20
 router-id 20.0.0.2
 group RR internal
 peer RR connect-interface LoopBack0
 peer 20.0.0.1 group RR
 #
 address-family vpnv4
  undo policy vpn-target
  peer RR enable
  peer RR next-hop-local
#
```

- PE_3
```cmd
#
bgp 20
 router-id 20.0.0.3
 group RR internal
 peer RR connect-interface LoopBack0
 peer 20.0.0.1 group RR
 #
 address-family vpnv4
  peer RR enable
#
```

- PE_4
```cmd
#
bgp 20
 router-id 20.0.0.4
 group RR internal
 peer RR connect-interface LoopBack0
 peer 20.0.0.1 group RR
 #
 address-family vpnv4
  peer RR enable
#
```


### 2. 配置AS之间的MP-eBGP

#### AS 10

- ASBR_1
```cmd
#
bgp 10
 group AS20 external
 peer AS20 as-number 20
 peer 10.20.0.20 group AS20
 #
 address-family vpnv4
  peer AS20 enable
#
```

#### AS 20

- ASBR_2
```cmd
#
bgp 20
 group AS10 external
 peer AS10 as-number 10
 peer 10.20.0.10 group AS10
 #
 address-family vpnv4
  peer AS10 enable
#
```


> MP-BGP配置完成后，可检查BGP对等体建立情况情况
> 
> `display bgp peer vpnv4`

## 五、私网地址规划与配置

### 1. 配置VPN实例

> VPN实例RD、RT分配规范（交叉RT）
> 
> RD ：AS:VPN
> 
> import-RT：对端ASVPN:本端ASVPN
> 
> export-RT:   本端ASVPN:对端ASVPN
> 
> MPLS L3VPN Inter-AS Option B方式要求route-target必须要匹配
#### AS 10

- PE_1
```cmd
#
ip vpn-instance 11
 route-distinguisher 10:11
 vpn-target 2011:1011 import-extcommunity
 vpn-target 1011:2011 export-extcommunity
# 
```

- CE_1
```cmd
#
ip vpn-instance 11
 route-distinguisher 10:11
 vpn-target 2011:1011 import-extcommunity
 vpn-target 1011:2011 export-extcommunity
# 
```

- PE_2
```cmd
#
ip vpn-instance 22
 route-distinguisher 10:22
 vpn-target 2022:1022 import-extcommunity
 vpn-target 1022:2022 export-extcommunity
#
```

- CE_2
```cmd
#
ip vpn-instance 22
 route-distinguisher 10:22
 vpn-target 2022:1022 import-extcommunity
 vpn-target 1022:2022 export-extcommunity
#
```

#### AS 20

- PE_3
```cmd
#
ip vpn-instance 11
 route-distinguisher 20:11
 vpn-target 1011:2011 import-extcommunity
 vpn-target 2011:1011 export-extcommunity
# 
```

- CE_3
```cmd
#
ip vpn-instance 11
 route-distinguisher 20:11
 vpn-target 1011:2011 import-extcommunity
 vpn-target 2011:1011 export-extcommunity
# 
```

- PE_4
```cmd
#
ip vpn-instance 22
 route-distinguisher 20:22
 vpn-target 1022:2022 import-extcommunity
 vpn-target 2022:1022 export-extcommunity
#
```

- CE_4
```cmd
#
ip vpn-instance 22
 route-distinguisher 20:22
 vpn-target 1022:2022 import-extcommunity
 vpn-target 2022:1022 export-extcommunity
#
```


### 2. 配置VPN私网地址

> VPN私网地址编制规范（仅实验环境）：
> 
> 私网互联地址 ：AS.VRF.AZ.x/24
> 
> AS 10私网地址：10.VRF.1.x/32
> 
> AS 20私网地址：20.VRF.2.x/32

#### AS 10

- PE_1
```cmd
#
interface LoopBack11
 ip binding vpn-instance 11
 ip address 10.11.1.3 32
#
interface GigabitEthernet0/0/1.11
 description TO CE_1_VPN_11
 ip binding vpn-instance 11
 ip address 10.11.36.3 24
 vlan-type dot1q vid 11
#
```

- CE_1
```cmd
#
interface LoopBack11
 ip binding vpn-instance 11
 ip address 10.11.1.6 32
#
interface GigabitEthernet0/0/0.11
 description TO PE_1_VPN_11
 ip binding vpn-instance 11
 ip address 10.11.36.6 24
 vlan-type dot1q vid 11
#
```

- PE_2
```cmd
#
interface LoopBack22
 ip binding vpn-instance 22
 ip address 10.22.2.4 32
#
interface GigabitEthernet0/0/1.22
 description TO_CE_2_VPN_22
 ip binding vpn-instance 22
 ip address 10.22.47.4 24
 vlan-type dot1q vid 22
#
```

- CE_2
```cmd
#
interface LoopBack22
 ip binding vpn-instance 22
 ip address 10.22.2.7 32
#
interface GigabitEthernet0/0/0.22
 description TO PE_2_VPN_22
 ip binding vpn-instance 22
 ip address 10.22.47.7 24
 vlan-type dot1q vid 22
#
```

#### AS 20

- PE_3
```cmd
#
interface LoopBack11
 ip binding vpn-instance 11
 ip address 20.11.1.3 32
#
interface GigabitEthernet0/0/1.11
 description TO CE_3_VPN_11
 ip binding vpn-instance 11
 ip address 20.11.36.3 24
 vlan-type dot1q vid 11
#
```

- CE_3
```cmd
#
interface LoopBack11
 ip binding vpn-instance 11
 ip address 20.11.1.6 32
#
interface GigabitEthernet0/0/0.11
 description TO PE_3_VPN_11
 ip binding vpn-instance 11
 ip address 20.11.36.6 24
 vlan-type dot1q vid 11
#
```

- PE_4
```cmd
#
interface LoopBack22
 ip binding vpn-instance 22
 ip address 20.22.2.4 32
#
interface GigabitEthernet0/0/1.22
 description TO_CE_4_VPN_22
 ip binding vpn-instance 22
 ip address 20.22.47.4 24
 vlan-type dot1q vid 22
#
```

- CE_4
```cmd
#
interface LoopBack22
 ip binding vpn-instance 22
 ip address 20.22.2.7 32
#
interface GigabitEthernet0/0/0.22
 description TO PE_4_VPN_22
 ip binding vpn-instance 22
 ip address 20.22.47.7 24
 vlan-type dot1q vid 22
#
```

> 可使用以下命令验证配置是否生效
> 
> `display ip vpn-instance`
> 
> `display current-configuration configuration vpn-instance`
> 
> `display ip interface brief`

### 3. 配置私网IGP协议

> 私网ISIS network-entity 编址规范（仅实验环境）：AS.VPN.LOOPBACK.00
> 
> 私网OSPF domain-di 编址规范（仅实验环境）：0.0.0.VPN

#### AS 10

- PE_1
```cmd
#
isis 11 vpn-instance 11
 is-level level-2
 network-entity 10.0011.0100.0000.0003.00
#
```

- CE_1
```cmd
#
isis 11 vpn-instance 11
 is-level level-2
 network-entity 10.0011.0100.0000.0006.00
#
```

- PE_2
```cmd
#
ospf 22 router-id 10.22.2.4 vpn-instance 22
 domain-id 0.0.0.22
 area 0.0.0.0
#
```

- CE_2
```cmd
#
ospf 22 router-id 10.22.2.7 vpn-instance 22
 domain-id 0.0.0.22
 area 0.0.0.0
#
```

#### AS 20

- PE_3
```cmd
#
isis 11 vpn-instance 11
 is-level level-2
 network-entity 20.0011.0200.0000.0003.00
#
```

- CE_3
```cmd
#
isis 11 vpn-instance 11
 is-level level-2
 network-entity 20.0011.0200.0000.0006.00
#
```

- PE_4
```cmd
#
ospf 22 router-id 20.22.2.4 vpn-instance 22
 domain-id 0.0.0.22
 area 0.0.0.0
#
```

- CE_4
```cmd
#
ospf 22 router-id 20.22.2.7 vpn-instance 22
 domain-id 0.0.0.22
 area 0.0.0.0
#
```

### 4. 在私网接口启用IGP

#### AS 10

- PE_1
```cmd
#
interface LoopBack11
 isis enable 11
#
interface GigabitEthernet0/0/1.11
 isis enable 11
#
```

- CE_1
```cmd
#
interface LoopBack11
 isis enable 11
#
interface GigabitEthernet0/0/0.11
 isis enable 11
#
```

- PE_2
```cmd
#
ospf 22 router-id 10.22.2.4 vpn-instance 22
 area 0.0.0.0
  network 10.22.2.4 0.0.0.0
  network 10.22.47.4 0.0.0.0
#
```

- CE_2
```cmd
#
ospf 22 router-id 10.22.2.7 vpn-instance 22
 area 0.0.0.0
  network 10.22.2.7 0.0.0.0
  network 10.22.47.7 0.0.0.0
#
```

#### AS 20

- PE_3
```cmd
#
interface LoopBack11
 isis enable 11
#
interface GigabitEthernet0/0/1.11
 isis enable 11
#
```

- CE_3
```cmd
#
interface LoopBack11
 isis enable 11
#
interface GigabitEthernet0/0/0.11
 isis enable 11
#
```

- PE_4
```cmd
#
ospf 22 router-id 20.22.2.4 vpn-instance 22
 area 0.0.0.0
  network 20.22.2.4 0.0.0.0
  network 20.22.47.4 0.0.0.0
#
```

- CE_4
```cmd
#
ospf 22 router-id 20.22.2.7 vpn-instance 22
 area 0.0.0.0
  network 20.22.2.7 0.0.0.0
  network 20.22.47.7 0.0.0.0
#
```

> 检查私网IGP协议运行状态
> 
> `display isis peer`
> 
> `display ospf peer`

### 5. 把IGP和BGP的路由进行相互注入

#### AS 10

- PE_1
```cmd
#
bgp 10
 #
 ip vpn-instance 11
  #
  address-family ipv4 unicast
   import-route isis 11
#
isis 11
 #
 address-family ipv4 unicast
  import-route bgp
#
```

- PE_2
```cmd
#
bgp 10
 #
 ip vpn-instance 22
  #
  address-family ipv4 unicast
   import-route ospf 22
#
ospf 22
 import-route bgp
#
```

#### AS 20

- PE_1
```cmd
#
bgp 20
 #
 ip vpn-instance 11
  #
  address-family ipv4 unicast
   import-route isis 11
#
isis 11
 #
 address-family ipv4 unicast
  import-route bgp
#
```

- PE_2
```cmd
#
bgp 20
 #
 ip vpn-instance 22
  #
  address-family ipv4 unicast
   import-route ospf 22
#
ospf 22
 import-route bgp
#
```


## 六、验证结果

### PE_1

1. 查看接口地址：`display ip interface brief`
   ![`display ip interface brief`](https://ryan1998.dpdns.org/illus/mpls-l3vpn-inter-as-option-b/pe1-display-ip-interface-brief.png)


2. 查看ISIS邻居状态：`display isis peer`
   ![`display isis peer`](https://ryan1998.dpdns.org/illus/mpls-l3vpn-inter-as-option-b/pe1-display-isis-peer.png)


3. 查看LDP邻居状态：`display mpls ldp peer`
   ![display mpls ldp peer](https://ryan1998.dpdns.org/illus/mpls-l3vpn-inter-as-option-b/pe1-displa-%20mpls-ldp-peer.png)


4. 查看MP-BGP对等体状态：`display bgp peer vpnv4`
   ![display bgp peer vpnv4](https://ryan1998.dpdns.org/illus/mpls-l3vpn-inter-as-option-b/pe1-display-bgp-peer-vpnv4.png)


5. 查看VPN路由表：`display ip routing-table vpn-instance 11`
   ![display ip routing-table vpn-instance 11](https://ryan1998.dpdns.org/illus/mpls-l3vpn-inter-as-option-b/pe1-display-ip-routing-table-vpn-instance-11.png)
### CE_1
 
1. 查看接口地址：`display ip interface brief`
   ![display ip interface brief](https://ryan1998.dpdns.org/illus/mpls-l3vpn-inter-as-option-b/ce1-display-ip-interface-brief.png)


2. 查看 ISIS 邻居状态：`display isis peer`
   ![display isis peer](https://ryan1998.dpdns.org/illus/mpls-l3vpn-inter-as-option-b/ce1-display-isis-peer.png)


3. 查看 VPN 路由表：`display ip routing-table vpn-instance 11`
   ![display ip routing-table vpn-instance 11](https://ryan1998.dpdns.org/illus/mpls-l3vpn-inter-as-option-b/ce1-display-ip-routing-table-vpn-instance-11.png)


4. 使用CE1 Loop11和CE3 Loop11进行Ping测试：`ping -a 10.11.1.6 -vpn-instance 11 20.11.1.6`
   ![display ip routing-table vpn-instance 11](https://ryan1998.dpdns.org/illus/mpls-l3vpn-inter-as-option-b/ce1-ping-a-10.11.1.6-vpn-instance-11-20.11.1.6.png)

<div style="page-break-before: always;"></div>

## 附录：配置文件

### AS 10

- RR_1
```cmd
#
 sysname RR_1
#
 router id 10.0.0.1
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0001.00
#
 mpls lsr-id 10.0.0.1
#
mpls ldp
#
interface LoopBack0
 ip address 10.0.0.1 255.255.255.255
 isis enable 1
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO P_1
 combo enable copper
 ip address 10.0.15.1 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
bgp 10
 router-id 10.0.0.1
 group PE internal
 peer PE connect-interface LoopBack0
 peer 10.0.0.2 group PE
 peer 10.0.0.3 group PE
 peer 10.0.0.4 group PE
 #
 address-family vpnv4
  undo policy vpn-target
  peer PE enable
  peer PE reflect-client
#
```

- ASBR_1
```cmd
#
 sysname ASBR_1
#
 router id 10.0.0.2
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0002.00
#
 mpls lsr-id 10.0.0.2
#
mpls ldp
#
interface LoopBack0
 ip address 10.0.0.2 255.255.255.255
 isis enable 1
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO P_1
 combo enable copper
 ip address 10.0.25.2 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/10
 port link-mode route
 description TO AS20_ASBR_2
 combo enable copper
 ip address 10.20.0.10 255.255.255.0
 mpls enable
 mpls ldp enable
#
bgp 10
 router-id 10.0.0.2
 group AS20 external
 peer AS20 as-number 20
 group RR internal
 peer RR connect-interface LoopBack0
 peer 10.0.0.1 group RR
 peer 10.20.0.20 group AS20
 #
 address-family vpnv4
  undo policy vpn-target
  peer AS20 enable
  peer RR enable
  peer RR next-hop-local
#
```

- PE_1
```cmd
#
 sysname PE_1
#
ip vpn-instance 11
 route-distinguisher 10:11
 vpn-target 2011:1011 import-extcommunity
 vpn-target 1011:2011 export-extcommunity
#
 router id 10.0.0.3
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0003.00
#
isis 11 vpn-instance 11
 is-level level-2
 network-entity 10.0011.0100.0000.0003.00
 #
 address-family ipv4 unicast
  import-route bgp
#
 mpls lsr-id 10.0.0.3
#
mpls ldp
#
interface LoopBack0
 ip address 10.0.0.3 255.255.255.255
 isis enable 1
#
interface LoopBack11
 ip binding vpn-instance 11
 ip address 10.11.1.3 255.255.255.255
 isis enable 11
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO P_1
 combo enable copper
 ip address 10.0.35.3 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/1
 port link-mode route
 description TO CE_1
 combo enable copper
 ip address 10.0.36.3 255.255.255.0
 isis enable 1
#
interface GigabitEthernet0/0/1.11
 description TO CE_1_VPN_11
 ip binding vpn-instance 11
 ip address 10.11.36.3 255.255.255.0
 isis enable 11
 vlan-type dot1q vid 11
#
bgp 10
 router-id 10.0.0.3
 group RR internal
 peer RR connect-interface LoopBack0
 peer 10.0.0.1 group RR
 #
 address-family vpnv4
  peer RR enable
 #
 ip vpn-instance 11
  #
  address-family ipv4 unicast
   import-route isis 11
#
```

- PE_2
```cmd
#
 sysname PE_2
#
ip vpn-instance 22
 route-distinguisher 10:22
 vpn-target 2022:1022 import-extcommunity
 vpn-target 1022:2022 export-extcommunity
#
 router id 10.0.0.4
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0004.00
#
ospf 22 router-id 10.22.2.4 vpn-instance 22
 import-route bgp
 domain-id 0.0.0.22
 area 0.0.0.0
  network 10.22.2.4 0.0.0.0
  network 10.22.47.4 0.0.0.0
#
 mpls lsr-id 10.0.0.4
#
mpls ldp
#
interface LoopBack0
 ip address 10.0.0.4 255.255.255.255
 isis enable 1
#
interface LoopBack22
 ip binding vpn-instance 22
 ip address 10.22.2.4 255.255.255.255
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO P_1
 combo enable copper
 ip address 10.0.45.4 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/1
 port link-mode route
 description TO CE_2
 combo enable copper
 ip address 10.0.47.4 255.255.255.0
 isis enable 1
#
interface GigabitEthernet0/0/1.22
 description TO_CE_2_VPN_22
 ip binding vpn-instance 22
 ip address 10.22.47.4 255.255.255.0
 vlan-type dot1q vid 22
#
bgp 10
 router-id 10.0.0.4
 group RR internal
 peer RR connect-interface LoopBack0
 peer 10.0.0.1 group RR
 #
 address-family vpnv4
  peer RR enable
 #
 ip vpn-instance 22
  #
  address-family ipv4 unicast
   import-route ospf 22
#
```

- P_1
```cmd
#
 sysname P_1
#
 router id 10.0.0.5
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0005.00
#
 mpls lsr-id 10.0.0.5
#
mpls ldp
#
interface LoopBack0
 ip address 10.0.0.5 255.255.255.255
 isis enable 1
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO RR_1
 combo enable copper
 ip address 10.0.15.5 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/1
 port link-mode route
 description TO ASBR_1
 combo enable copper
 ip address 10.0.25.5 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/7
 port link-mode route
 description TO PE_1
 combo enable copper
 ip address 10.0.35.5 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/8
 port link-mode route
 description TO PE_2
 combo enable copper
 ip address 10.0.45.5 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
```

- CE_1
```cmd
#
 sysname CE_1
#
ip vpn-instance 11
 route-distinguisher 10:11
 vpn-target 2011:1011 import-extcommunity
 vpn-target 1011:2011 export-extcommunity
#
 router id 10.0.0.6
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0006.00
#
isis 11 vpn-instance 11
 is-level level-2
 network-entity 10.0011.0100.0000.0006.00
#
interface LoopBack0
 ip address 10.0.0.6 255.255.255.255
 isis enable 1
#
interface LoopBack11
 ip binding vpn-instance 11
 ip address 10.11.1.6 255.255.255.255
 isis enable 11
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO PE_1
 combo enable copper
 ip address 10.0.36.6 255.255.255.0
 isis enable 1
#
interface GigabitEthernet0/0/0.11
 description TO PE_1_VPN_11
 ip binding vpn-instance 11
 ip address 10.11.36.6 255.255.255.0
 isis enable 11
 vlan-type dot1q vid 11
#
```

- CE_2
```cmd
#
 sysname CE_2
#
ip vpn-instance 22
 route-distinguisher 10:22
 vpn-target 2022:1022 import-extcommunity
 vpn-target 1022:2022 export-extcommunity
#
 router id 10.0.0.7
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0100.0000.0007.00
#
ospf 22 router-id 10.22.2.7 vpn-instance 22
 domain-id 0.0.0.22
 area 0.0.0.0
  network 10.22.2.7 0.0.0.0
  network 10.22.47.7 0.0.0.0
#
interface LoopBack0
 ip address 10.0.0.7 255.255.255.255
 isis enable 1
#
interface LoopBack22
 ip binding vpn-instance 22
 ip address 10.22.2.7 255.255.255.255
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO PE_2
 combo enable copper
 ip address 10.0.47.7 255.255.255.0
 isis enable 1
#
interface GigabitEthernet0/0/0.22
 description TO PE_2_VPN_22
 ip binding vpn-instance 22
 ip address 10.22.47.7 255.255.255.0
 vlan-type dot1q vid 22
#
```


### AS 20

- RR_2
```cmd
#
 sysname RR_2
#
 router id 20.0.0.1
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0001.00
#
 mpls lsr-id 20.0.0.1
#
mpls ldp
#
interface LoopBack0
 ip address 20.0.0.1 255.255.255.255
 isis enable 1
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO P_2
 combo enable copper
 ip address 20.0.15.1 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
bgp 20
 router-id 20.0.0.1
 group PE internal
 peer PE connect-interface LoopBack0
 peer 20.0.0.2 group PE
 peer 20.0.0.3 group PE
 peer 20.0.0.4 group PE
 #
 address-family vpnv4
  undo policy vpn-target
  peer PE enable
  peer PE reflect-client
#
```

- ASBR_2
```cmd
#
 sysname ASBR_2
#
 router id 20.0.0.2
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0002.00
#
 mpls lsr-id 20.0.0.2
#
mpls ldp
#
interface LoopBack0
 ip address 20.0.0.2 255.255.255.255
 isis enable 1
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO P_2
 combo enable copper
 ip address 20.0.25.2 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/10
 port link-mode route
 description TO AS10_ASBR_1
 combo enable copper
 ip address 10.20.0.20 255.255.255.0
 mpls enable
 mpls ldp enable
#
bgp 20
 router-id 20.0.0.2
 group AS10 external
 peer AS10 as-number 10
 group RR internal
 peer RR connect-interface LoopBack0
 peer 10.20.0.10 group AS10
 peer 20.0.0.1 group RR
 #
 address-family vpnv4
  undo policy vpn-target
  peer AS10 enable
  peer RR enable
  peer RR next-hop-local
#
```

- PE_3
```cmd
#
 sysname PE_3
#
ip vpn-instance 11
 route-distinguisher 20:11
 vpn-target 1011:2011 import-extcommunity
 vpn-target 2011:1011 export-extcommunity
#
 router id 20.0.0.3
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0003.00
#
isis 11 vpn-instance 11
 is-level level-2
 network-entity 20.0011.0200.0000.0003.00
 #
 address-family ipv4 unicast
  import-route bgp
#
 mpls lsr-id 20.0.0.3
#
mpls ldp
#
interface LoopBack0
 ip address 20.0.0.3 255.255.255.255
 isis enable 1
#
interface LoopBack11
 ip binding vpn-instance 11
 ip address 20.11.1.3 255.255.255.255
 isis enable 11
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO P_2
 combo enable copper
 ip address 20.0.35.3 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/1
 port link-mode route
 description TO CE_3
 combo enable copper
 ip address 20.0.36.3 255.255.255.0
 isis enable 1
#
interface GigabitEthernet0/0/1.11
 description TO CE_3_VPN_11
 ip binding vpn-instance 11
 ip address 20.11.36.3 255.255.255.0
 isis enable 11
 vlan-type dot1q vid 11
#
bgp 20
 router-id 20.0.0.3
 group RR internal
 peer RR connect-interface LoopBack0
 peer 20.0.0.1 group RR
 #
 address-family vpnv4
  peer RR enable
 #
 ip vpn-instance 11
  #
  address-family ipv4 unicast
   import-route isis 11
#
```

- PE_4
```cmd
#
 sysname PE_4
#
ip vpn-instance 22
 route-distinguisher 20:22
 vpn-target 1022:2022 import-extcommunity
 vpn-target 2022:1022 export-extcommunity
#
 router id 20.0.0.4
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0004.00
#
ospf 22 router-id 20.22.2.4 vpn-instance 22
 import-route bgp
 domain-id 0.0.0.22
 area 0.0.0.0
  network 20.22.2.4 0.0.0.0
  network 20.22.47.4 0.0.0.0
#
 mpls lsr-id 20.0.0.4
#
mpls ldp
#
interface LoopBack0
 ip address 20.0.0.4 255.255.255.255
 isis enable 1
#
interface LoopBack22
 ip binding vpn-instance 22
 ip address 20.22.2.4 255.255.255.255
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO P_2
 combo enable copper
 ip address 20.0.45.4 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/1
 port link-mode route
 description TO CE_4
 combo enable copper
 ip address 20.0.47.4 255.255.255.0
 isis enable 1
#
interface GigabitEthernet0/0/1.22
 description TO_CE_4_VPN_22
 ip binding vpn-instance 22
 ip address 20.22.47.4 255.255.255.0
 vlan-type dot1q vid 22
#
bgp 20
 router-id 20.0.0.3
 group RR internal
 peer RR connect-interface LoopBack0
 peer 20.0.0.1 group RR
 #
 address-family vpnv4
  peer RR enable
 #
 ip vpn-instance 22
  #
  address-family ipv4 unicast
   import-route ospf 22
#
```

- P_2
```cmd
#
 sysname P_2
#
 router id 20.0.0.5
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0005.00
#
 mpls lsr-id 20.0.0.5
#
mpls ldp
#
interface LoopBack0
 ip address 20.0.0.5 255.255.255.255
 isis enable 1
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO RR_2
 combo enable copper
 ip address 20.0.15.5 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/1
 port link-mode route
 description TO ASBR_2
 combo enable copper
 ip address 20.0.25.5 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/7
 port link-mode route
 description TO PE_3
 combo enable copper
 ip address 20.0.35.5 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/8
 port link-mode route
 description TO PE_4
 combo enable copper
 ip address 20.0.45.5 255.255.255.0
 isis enable 1
 mpls enable
 mpls ldp enable
#
```

- CE_3
```cmd
#
 sysname CE_3
#
ip vpn-instance 11
 route-distinguisher 20:11
 vpn-target 1011:2011 import-extcommunity
 vpn-target 2011:1011 export-extcommunity
#
 router id 20.0.0.6
#
isis 11 vpn-instance 11
 is-level level-2
 network-entity 20.0011.0200.0000.0006.00
#
interface LoopBack0
 ip address 20.0.0.6 255.255.255.255
#
interface LoopBack11
 ip binding vpn-instance 11
 ip address 20.11.1.6 255.255.255.255
 isis enable 11
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO PE_3
 combo enable copper
 ip address 20.0.36.6 255.255.255.0
#
interface GigabitEthernet0/0/0.11
 description TO PE_3_VPN_11
 ip binding vpn-instance 11
 ip address 20.11.36.6 255.255.255.0
 isis enable 11
 vlan-type dot1q vid 11
#
```

- CE_4
```cmd
#
 sysname CE_4
#
ip vpn-instance 22
 route-distinguisher 20:22
 vpn-target 1022:2022 import-extcommunity
 vpn-target 2022:1022 export-extcommunity
#
 router id 20.0.0.7
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 20.0000.0200.0000.0007.00
#
ospf 22 router-id 20.22.2.7 vpn-instance 22
 domain-id 0.0.0.22
 area 0.0.0.0
  network 20.22.2.7 0.0.0.0
  network 20.22.47.7 0.0.0.0
#
interface LoopBack0
 ip address 20.0.0.7 255.255.255.255
 isis enable 1
#
interface LoopBack22
 ip binding vpn-instance 22
 ip address 20.22.2.7 255.255.255.255
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO PE_4
 combo enable copper
 ip address 20.0.47.7 255.255.255.0
 isis enable 1
#
interface GigabitEthernet0/0/0.22
 description TO PE_4_VPN_22
 ip binding vpn-instance 22
 ip address 20.22.47.7 255.255.255.0
 vlan-type dot1q vid 22
#
```
