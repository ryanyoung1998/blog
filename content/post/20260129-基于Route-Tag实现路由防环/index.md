---
title: 基于Route-Tag实现路由防环
description: 
date: 2026-01-29
slug: routing-loop-prevention-based-on-route-tag
image: https://ryan1998.dpdns.org/post/20260119-routing-loop-prevention-based-on-route-tag/cover.png
categories:
    - Datacom
    - MPLS L3VPN
tags:
    - Route Tag
    - Route Policy
---

## 组网说明

![topo](https://ryan1998.dpdns.org/post/20260119-routing-loop-prevention-based-on-route-tag/illus-1-topo.png)

### 网络整体架构

- **跨域方式**：
	
	- AS 100与AS 200之间采用Option A方式对接，实现L3VPN业务路由传递。
    
- **AS 100内部结构**：
    
    - ASBR_1同时作为路由反射器，为AS 100内部分发VPNv4路由。
        
    - PE_1与PE_2作为ASBR_1的RR客户端。
        
- **VPN业务设计**：
    
    - CE_1与CE_2上均部署两个VPN实例：`VPN 10`与`VPN 20`。
        
    - `VPN 10` 使用**IS-IS进程10**与PE设备对接。
        
    - `VPN 20` 使用**OSPF进程20**与PE设备对接。
        
- **路由交互策略**：
    
    - PE设备在IS-IS 10与OSPF 20中**引入BGP VPN路由**。
        
    - 在BGP `ip vpn-instance 10` 中**引入IS-IS 10**的路由。
        
    - 在BGP `ip vpn-instance 20` 中**引入OSPF 20**的路由。
        
- **CE间路由传递**：
	- CE_1与CE_2之间分别建立IS-IS 10与OSPF 20邻居，实现业务路由互通。

<div style="page-break-before: always;"></div>

### 存在的路由环路

由于PE与CE之间双向路由引入，形成以下环路路径：
1. **环路一**： PE_1 → CE_1 →  PE_2
2. **环路二**： PE_1 → CE_2 → PE_2
3. **环路二**： PE_1 → CE_1 → CE_2 → PE_2
![generate-routing-loop](https://ryan1998.dpdns.org/post/20260119-routing-loop-prevention-based-on-route-tag/illus-2-generate-routing-loop.png)

## 路由防环设计

为打破路由环路，采用**路由标记**与**策略过滤**机制。

### 防环配置过程

#### PE设备路由标记配置
**PE_1配置：**
```cfg
# 在IS-IS和OSPF引入BGP路由时标记为1
isis 10
 address-family ipv4 unicast
  import-route bgp tag 1
ospf 20
 import-route bgp tag 1
```

**PE_2配置：**
```cfg
# 在IS-IS和OSPF引入BGP路由时标记为2
isis 10
 address-family ipv4 unicast
  import-route bgp tag 2
ospf 20
 import-route bgp tag 2
```

#### 路由策略定义
**PE_1路由策略（过滤PE_2的标签2）：**
```cfg
route-policy Filter_Tag deny node 2
 if-match tag 2          # 拒绝PE_2发出的路由
route-policy Filter_Tag permit node 100
                         # 允许其他路由
```

**PE_2路由策略（过滤PE_1的标签1）：**
```cfg
route-policy Filter_Tag deny node 1
 if-match tag 1          # 拒绝PE_1发出的路由
route-policy Filter_Tag permit node 100
                         # 允许其他路由
```

#### IGP路由过滤应用
**PE_1/PE_2通用配置：**
```cfg
# IS-IS入方向过滤
isis 10
 address-family ipv4 unicast
  filter-policy route-policy Filter_Tag import

# OSPF入方向过滤  
ospf 20
 filter-policy route-policy Filter_Tag import
```

#### BGP路由引入过滤
**PE_1/PE_2 VPN实例配置：**
```cfg
bgp 100
 # VPN实例10：IS-IS→BGP引入时过滤
 ip vpn-instance 10
  address-family ipv4 unicast
   import-route isis 10 route-policy Filter_Tag
  
 # VPN实例20：OSPF→BGP引入时过滤
 ip vpn-instance 20
  address-family ipv4 unicast
   import-route ospf 20 route-policy Filter_Tag
```

<div style="page-break-before: always;"></div>

### 防环机制工作原理

**每台PE在将BGP路由引入IGP时打上唯一标签，并在接收IGP路由时过滤对端PE的标签**

#### 数据流示例：
1. **PE_1发出路由路径**：
![resolve-routing-loop](https://ryan1998.dpdns.org/post/20260119-routing-loop-prevention-based-on-route-tag/illus-3-resolve-routing-loop.png)

<div style="page-break-before: always;"></div>

1. **PE_2发出路由路径**：
![resolve-routing-loop](https://ryan1998.dpdns.org/post/20260119-routing-loop-prevention-based-on-route-tag/illus-4-resolve-routing-loop.png)
#### 环路阻断点：
- **PE_1侧**：过滤来自PE_2的`tag=2`路由
- **PE_2侧**：过滤来自PE_1的`tag=1`路由
## 路由防环说明
通过**标签识别+双向过滤**实现精确防环，具有以下特点：
1. **对称性设计**：PE_1与PE_2配置逻辑镜像对称
2. **双向防护**：在IGP接收和BGP引入两处过滤，增强可靠性
3. **可扩展性**：可通过调整标签值支持更多PE设备
4. **维护友好**：标签对应关系清晰，便于故障排查

<div style="page-break-before: always;"></div>

## 附录：配置文件

### ASBR_2
```cfg
#
 sysname ASBR_2
#
ip vpn-instance 10
 route-distinguisher 200:10
 vpn-target 200:10 import-extcommunity
 vpn-target 200:10 export-extcommunity
#
ip vpn-instance 20
 route-distinguisher 200:20
 vpn-target 200:20 import-extcommunity
 vpn-target 200:20 export-extcommunity
#
 router id 20.0.0.0
#
 lldp global enable
#
interface LoopBack0
 ip address 200.0.0.0 255.255.255.255
#
interface LoopBack10
 ip binding vpn-instance 10
 ip address 200.10.0.10 255.255.255.255
#
interface LoopBack20
 ip binding vpn-instance 20
 ip address 200.20.0.20 255.255.255.255
#
interface GigabitEthernet0/0/0
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/0.10
 description TO AS100_VPN10
 ip binding vpn-instance 10
 ip address 200.100.10.0 255.255.255.254
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/0.20
 description TO AS100_VPN20
 ip binding vpn-instance 20
 ip address 200.100.20.0 255.255.255.254
 vlan-type dot1q vid 20
#
bgp 200
 router-id 200.0.0.0
 #
 ip vpn-instance 10
  group AS100 external
  peer AS100 as-number 100
  peer 200.100.10.1 group AS100
  #
  address-family ipv4 unicast
   network 200.10.0.10 255.255.255.255
   peer AS100 enable
 #
 ip vpn-instance 20
  group AS100 external
  peer AS100 as-number 100
  peer 200.100.20.1 group AS100
  #
  address-family ipv4 unicast
   network 200.20.0.20 255.255.255.255
   peer AS100 enable
#
```

 ### ASBR_1
```cfg
#
 sysname ASBR_1
#
ip vpn-instance 10
 route-distinguisher 100:10
 vpn-target 100:10 import-extcommunity
 vpn-target 100:10 export-extcommunity
#
ip vpn-instance 20
 route-distinguisher 100:20
 vpn-target 100:20 import-extcommunity
 vpn-target 100:20 export-extcommunity
#
 router id 100.0.0.0
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0000.0000.00
#
 mpls lsr-id 100.0.0.0
#
 lldp global enable
#
mpls ldp
#
interface LoopBack0
 ip address 100.0.0.0 255.255.255.255
 isis enable 1
#
interface LoopBack10
 ip binding vpn-instance 10
 ip address 100.10.0.10 255.255.255.255
#
interface LoopBack20
 ip binding vpn-instance 20
 ip address 100.20.0.20 255.255.255.255
#
interface GigabitEthernet0/0/0
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/0.10
 description TO AS200_VPN10
 ip binding vpn-instance 10
 ip address 200.100.10.1 255.255.255.254
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/0.20
 description TO AS200_VPN20
 ip binding vpn-instance 20
 ip address 200.100.20.1 255.255.255.254
 vlan-type dot1q vid 20
#
interface GigabitEthernet0/0/1
 port link-mode route
 description TO PE_1
 combo enable copper
 ip address 100.0.1.0 255.255.255.254
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/2
 port link-mode route
 description TO PE_2
 combo enable copper
 ip address 100.0.2.0 255.255.255.254
 isis enable 1
 mpls enable
 mpls ldp enable
#
bgp 100
 router-id 100.0.0.0
 group AS100 internal
 peer AS100 connect-interface LoopBack0
 peer 100.0.0.1 group AS100
 peer 100.0.0.2 group AS100
 #
 address-family vpnv4
  undo policy vpn-target
  peer AS100 enable
  peer AS100 next-hop-local
  peer AS100 reflect-client
 #
 ip vpn-instance 10
  group AS200 external
  peer AS200 as-number 200
  peer 200.100.10.0 group AS200
  #
  address-family ipv4 unicast
   network 100.10.1.10 255.255.255.255
   peer AS200 enable
   peer AS200 route-policy AS100_VPN10 export
 #
 ip vpn-instance 20
  group AS200 external
  peer AS200 as-number 200
  peer 200.100.20.0 group AS200
  #
  address-family ipv4 unicast
   network 100.20.1.20 255.255.255.255
   peer AS200 enable
   peer AS200 route-policy AS100_VPN20 export
#
route-policy AS100_VPN10 permit node 10
 if-match ip address prefix-list AS100_VPN10
#
route-policy AS100_VPN10 deny node 100
#
route-policy AS100_VPN20 permit node 10
 if-match ip address prefix-list AS100_VPN20
#
route-policy AS100_VPN20 deny node 100
#
 ip prefix-list AS100_VPN10 index 10 permit 100.10.1.10 32
 ip prefix-list AS100_VPN10 index 20 permit 100.10.2.10 32
 ip prefix-list AS100_VPN10 index 30 permit 100.10.3.10 32
 ip prefix-list AS100_VPN10 index 40 permit 100.10.4.10 32
 ip prefix-list AS100_VPN20 index 10 permit 100.20.1.20 32
 ip prefix-list AS100_VPN20 index 20 permit 100.20.2.20 32
 ip prefix-list AS100_VPN20 index 30 permit 100.20.3.20 32
 ip prefix-list AS100_VPN20 index 40 permit 100.20.4.20 32
#
```

 ### PE_1
```cfg
#
 sysname PE_1
#
ip vpn-instance 10
 route-distinguisher 100:10
 vpn-target 100:10 import-extcommunity
 vpn-target 100:10 export-extcommunity
#
ip vpn-instance 20
 route-distinguisher 100:20
 vpn-target 100:20 import-extcommunity
 vpn-target 100:20 export-extcommunity
#
 router id 100.0.0.1
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0000.0001.00
#
isis 10 vpn-instance 10
 is-level level-2
 cost-style wide
 network-entity 10.0010.0000.0001.00
 #
 address-family ipv4 unicast
  import-route bgp tag 1
  filter-policy route-policy Filter_Tag import
#
ospf 20 vpn-instance 20
 import-route bgp tag 1
 filter-policy route-policy Filter_Tag import
 area 0.0.0.0
  network 100.20.13.0 0.0.0.0
  network 100.20.14.0 0.0.0.0
#
 mpls lsr-id 100.0.0.1
#
 lldp global enable
#
mpls ldp
#
interface LoopBack0
 ip address 100.0.0.1 255.255.255.255
 isis enable 1
#
interface GigabitEthernet0/0/0
 port link-mode route
 description TO ASBR_1
 combo enable copper
 ip address 100.0.1.1 255.255.255.254
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/1
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/1.10
 description TO CE_1 VPN10
 ip binding vpn-instance 10
 ip address 100.10.13.0 255.255.255.254
 isis enable 10
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/1.20
 description TO CE_1 VPN20
 ip binding vpn-instance 20
 ip address 100.20.13.0 255.255.255.254
 vlan-type dot1q vid 20
#
interface GigabitEthernet0/0/2
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/2.10
 description TO CE_2 VPN10
 ip binding vpn-instance 10
 ip address 100.10.14.0 255.255.255.254
 isis enable 10
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/2.20
 description TO CE_2 VPN20
 ip binding vpn-instance 20
 ip address 100.20.14.0 255.255.255.254
 vlan-type dot1q vid 20
#
bgp 100
 router-id 100.0.0.1
 group AS100 internal
 peer AS100 connect-interface LoopBack0
 peer 100.0.0.0 group AS100
 #
 address-family vpnv4
  peer AS100 enable
 #
 ip vpn-instance 10
  #
  address-family ipv4 unicast
   import-route isis 10 route-policy Filter_Tag
 #
 ip vpn-instance 20
  #
  address-family ipv4 unicast
   import-route ospf 20 route-policy Filter_Tag
#
route-policy Filter_Tag deny node 2
 if-match tag 2
#
route-policy Filter_Tag permit node 100
#
```

### PE_2
```cfg
#
 sysname PE_2
#
ip vpn-instance 10
 route-distinguisher 100:10
 vpn-target 100:10 import-extcommunity
 vpn-target 100:10 export-extcommunity
#
ip vpn-instance 20
 route-distinguisher 100:20
 vpn-target 100:20 import-extcommunity
 vpn-target 100:20 export-extcommunity
#
 router id 100.0.0.2
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 10.0000.0000.0004.00
#
isis 10 vpn-instance 10
 is-level level-2
 cost-style wide
 network-entity 10.0010.0000.0002.00
 #
 address-family ipv4 unicast
  import-route bgp tag 2
  filter-policy route-policy Filter_Tag import
#
ospf 20 vpn-instance 20
 import-route bgp tag 2
 filter-policy route-policy Filter_Tag import
 area 0.0.0.0
  network 100.20.23.0 0.0.0.0
  network 100.20.24.0 0.0.0.0
#
 mpls lsr-id 100.0.0.2
#
 lldp global enable
#
mpls ldp
#
interface LoopBack0
 ip address 100.0.0.2 255.255.255.255
 isis enable 1
#
interface GigabitEthernet0/0/0
 port link-mode route
 combo enable copper
 ip address 100.0.2.1 255.255.255.254
 isis enable 1
 mpls enable
 mpls ldp enable
#
interface GigabitEthernet0/0/1
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/1.10
 description TO CE_1 VPN10
 ip binding vpn-instance 10
 ip address 100.10.23.0 255.255.255.254
 isis enable 10
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/1.20
 description TO CE_1 VPN20
 ip binding vpn-instance 20
 ip address 100.20.23.0 255.255.255.254
 vlan-type dot1q vid 20
#
interface GigabitEthernet0/0/2
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/2.10
 description TO CE_2 VPN10
 ip binding vpn-instance 10
 ip address 100.10.24.0 255.255.255.254
 isis enable 10
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/2.20
 description TO CE_2 VPN20
 ip binding vpn-instance 20
 ip address 100.20.24.0 255.255.255.254
 vlan-type dot1q vid 20
#
bgp 100
 router-id 100.0.0.2
 group AS100 internal
 peer AS100 connect-interface LoopBack0
 peer 100.0.0.0 group AS100
 #
 address-family vpnv4
  peer AS100 enable
 #
 ip vpn-instance 10
  #
  address-family ipv4 unicast
   import-route isis 10 route-policy Filter_Tag
 #
 ip vpn-instance 20
  #
  address-family ipv4 unicast
   import-route ospf 20 route-policy Filter_Tag
#
route-policy Filter_Tag deny node 1
 if-match tag 1
#
route-policy Filter_Tag permit node 100
#
```

 ### CE_1
 ```cfg
 #
 sysname CE_1
#
ip vpn-instance 10
 route-distinguisher 100:10
 vpn-target 100:10 import-extcommunity
 vpn-target 100:10 export-extcommunity
#
ip vpn-instance 20
 route-distinguisher 100:20
 vpn-target 100:20 import-extcommunity
 vpn-target 100:20 export-extcommunity
#
isis 10 vpn-instance 10
 is-level level-2
 cost-style wide
 network-entity 10.0010.0000.0003.00
#
ospf 20 vpn-instance 20
 area 0.0.0.0
  network 100.20.3.20 0.0.0.0
  network 100.20.13.1 0.0.0.0
  network 100.20.23.1 0.0.0.0
  network 100.20.34.0 0.0.0.0
#
 lldp global enable
#
interface LoopBack10
 ip binding vpn-instance 10
 ip address 100.10.3.10 255.255.255.255
 isis enable 10
#
interface LoopBack20
 ip binding vpn-instance 20
 ip address 100.20.3.20 255.255.255.255
#
interface GigabitEthernet0/0/0
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/0.10
 description TO CE_2 VPN10
 ip binding vpn-instance 10
 ip address 100.10.34.0 255.255.255.254
 isis enable 10
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/0.20
 description TO CE_2 VPN20
 ip binding vpn-instance 20
 ip address 100.20.34.0 255.255.255.254
 vlan-type dot1q vid 20
#
interface GigabitEthernet0/0/1
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/1.10
 description TO PE_1 VPN10
 ip binding vpn-instance 10
 ip address 100.10.13.1 255.255.255.254
 isis enable 10
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/1.20
 description TO PE_1 VPN20
 ip binding vpn-instance 20
 ip address 100.20.13.1 255.255.255.254
 vlan-type dot1q vid 20
#
interface GigabitEthernet0/0/2
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/2.10
 description TO PE_2 VPN10
 ip binding vpn-instance 10
 ip address 100.10.23.1 255.255.255.254
 isis enable 10
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/2.20
 description TO PE_2 VPN20
 ip binding vpn-instance 20
 ip address 100.20.23.1 255.255.255.254
 vlan-type dot1q vid 20
#
 ```

 ### CE_2
 ```cfg
 #
 sysname CE_2
#
ip vpn-instance 10
 route-distinguisher 100:10
 vpn-target 100:10 import-extcommunity
 vpn-target 100:10 export-extcommunity
#
ip vpn-instance 20
 route-distinguisher 100:20
 vpn-target 100:20 import-extcommunity
 vpn-target 100:20 export-extcommunity
#
isis 10 vpn-instance 10
 is-level level-2
 cost-style wide
 network-entity 10.0010.0000.0004.00
#
ospf 20 vpn-instance 20
 area 0.0.0.0
  network 100.20.4.20 0.0.0.0
  network 100.20.14.1 0.0.0.0
  network 100.20.24.1 0.0.0.0
  network 100.20.34.1 0.0.0.0
#
 lldp global enable
#
interface LoopBack10
 ip binding vpn-instance 10
 ip address 100.10.4.10 255.255.255.255
 isis enable 10
#
interface LoopBack20
 ip binding vpn-instance 20
 ip address 100.20.4.20 255.255.255.255
#
interface GigabitEthernet0/0/0
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/0.10
 description TO CE_1 VPN10
 ip binding vpn-instance 10
 ip address 100.10.34.1 255.255.255.254
 isis enable 10
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/0.20
 description TO CE_1 VPN20
 ip binding vpn-instance 20
 ip address 100.20.34.1 255.255.255.254
 vlan-type dot1q vid 20
#
interface GigabitEthernet0/0/1
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/1.10
 description TO PE_1 VPN10
 ip binding vpn-instance 10
 ip address 100.10.14.1 255.255.255.254
 isis enable 10
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/1.20
 description TO PE_1 VPN20
 ip binding vpn-instance 20
 ip address 100.20.14.1 255.255.255.254
 vlan-type dot1q vid 20
#
interface GigabitEthernet0/0/2
 port link-mode route
 combo enable copper
#
interface GigabitEthernet0/0/2.10
 description TO PE_2 VPN10
 ip binding vpn-instance 10
 ip address 100.10.24.1 255.255.255.254
 isis enable 10
 vlan-type dot1q vid 10
#
interface GigabitEthernet0/0/2.20
 description TO PE_2 VPN20
 ip binding vpn-instance 20
 ip address 100.20.24.1 255.255.255.254
 vlan-type dot1q vid 20
#
 ```