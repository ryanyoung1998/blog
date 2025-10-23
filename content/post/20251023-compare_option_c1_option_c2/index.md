---
title: 详解 MPLS L3VPN 跨域 Option C1 和 Option C2 的区别
<!-- description: 详解 MPLS L3VPN 跨域 Option C1 和 Option C2 的区别 -->
date: 2025-10-20
slug: compare_option_c1_option_c2
image: cover.jpg
categories:
    - Datacom
    - MPLS L3VPN
tags:
    - Option C1
    - Option C2
    - Inter-AS
---

**无论哪种方案，其最终目标都是一致的——让两个AS的PE路由器能够建立多跳的MP-EBGP邻居关系，并直接交换VPNv4路由。**

这个目标依赖于：**PE必须能够通过一条端到端的公网LSP到达对端PE的Loopback地址。**

两种方案的巨大差异就在于 **“如何在本端AS内部，将去往对端PE的路由和标签传递给自己域的PE设备”**。

---

### Option C1：ASBR通过IBGP向PE发布带标签路由

这种方案更接近标准RFC 8277的实现，控制平面是 **“BGP标签路由”** 驱动的。

#### 工作机理详解

1. **ASBR间建立EBGP，交换带标签的公网路由**
   
   - 两个AS的ASBR之间建立EBGP邻居，并启用`label-route-capability`（标签路由能力）。
   - 双方通过EBGP会话，将自己AS内PE路由器的Loopback地址（例如AS100的PE1: 1.1.1.1/32）通告给对方。
   - **关键点**：通告这条路由时，会附带一个标签。这个标签是由**出方向ASBR（如ASBR2）为路由1.1.1.1/32分配的（通常是通过LDP分配的）**。

2. **ASBR通过IBGP向PE发布路由和标签**
   
   - 在本端AS内部（例如AS100），**ASBR1与PE1之间需要建立MP-IBGP邻居关系**。
   - ASBR1从对端ASBR2学到的带标签的路由（例如去往PE2的2.2.2.2/32），会通过这个**MP-IBGP会话通告给PE1**。
   - **关键点**：ASBR1在向PE1发送这条BGP更新时，会**保留或重新分配**一个标签。这个标签指向ASBR1，用于构建跨AS的LSP。

3. **PE建立端到端LSP并运行MP-EBGP**
   
   - 此时，PE1的路由表中，去往PE2（2.2.2.2/32）的路由是一条BGP路由，并且带有一个标签。
   - 这样，PE1就拥有了一条完整的、指向PE2的LSP路径。当PE1要发送VPNv4数据包给PE2时，它会压入两层标签：
     - **外层标签**：就是这条BGP标签路由提供的标签，用于将报文指引到ASBR1，并最终穿越AS边界到达PE2。
     - **内层标签**：PE2通过MP-EBGP分配的VPN标签。
   - 由于PE1和PE2的Loopback地址可以互相可达，它们之间就能成功建立**多跳的MP-EBGP**会话，并直接交换VPNv4路由。

#### Option C1 配置关键点 (以HUAWEI设备命令为例)

```bash
# 在ASBR1上的关键配置
bgp 100
  peer <ASBR2_IP> as-number 200
  peer <ASBR2_IP> connect-interface GigabitEthernet0/0/1 # 连接ASBR2的接口
  peer <ASBR2_IP> label-route-capability check-tunnel-reachable # 关键：启用标签路由能力，并检查隧道可达性
  #
  peer <PE1_IP> as-number 100
  peer <PE1_IP> connect-interface LoopBack0
  peer <PE1_IP> label-route-capability # 关键：向PE1发布带标签的路由也需要此能力
  #
  ipv4-family unicast
    peer <ASBR2_IP> enable
    peer <PE1_IP> enable
    peer <PE1_IP> route-policy PASS_ALL export # 通常需要策略允许路由发布
    peer <PE1_IP> advertise-community # 可选，用于传递团体属性

# 在PE1上的配置
bgp 100
  peer <PE2_Loopback> as-number 200
  peer <PE2_Loopback> connect-interface LoopBack0
  peer <PE2_Loopback> ebgp-max-hop 64
  #
  ipv4-family vpnv4
    peer <PE2_Loopback> enable
```

---

### Option C2：ASBR将BGP路由引入IGP，触发LDP分配标签

这种方案是 **“LDP标签路由”** 驱动的，它巧妙地利用了IGP和LDP的协同工作。

#### 工作机理详解

1. **ASBR间建立EBGP，交换带标签的公网路由**
   
   - 这一步与**方案一完全相同**。ASBR之间通过EBGP交换对端PE的Loopback路由和标签。

2. **ASBR将BGP路由引入IGP**
   
   - **这是与方案一最根本的区别**。
   - 在本端ASBR（如ASBR1）上，配置**路由策略**，将从EBGP对等体（ASBR2）学到的特定BGP路由（如2.2.2.2/32）**引入（重分发）到本地的IGP（如OSPF或IS-IS）中**。
   - 于是，这条路由（2.2.2.2/32）就变成了一条IGP路由，在本AS内部（AS100）泛洪。PE1通过IGP学习到了去往2.2.2.2/32的路由。

3. **IGP触发LDP分配标签**
   
   - 由于2.2.2.2/32现在是一条IGP路由，AS100内部的LDP进程会像为其他IGP路由分配标签一样，为这条路由分配和分发LDP标签。
   - 最终，PE1上会形成一条通过**LDP LSP**到达2.2.2.2/32的路径。

4. **PE建立端到端LSP并运行MP-EBGP**
   
   - 此时，PE1认为去往PE2（2.2.2.2/32）是通过一条**AS内部的LDP LSP**。
   - 当PE1要发送VPNv4数据包时，它压入的两层标签中的**外层标签，是LDP分配的标签**。
   - 数据包在AS100内部通过LDP LSP转发到ASBR1，ASBR1再使用从ASBR2学来的BGP标签，将数据包转发到AS200。
   - 同样，PE1和PE2的Loopback地址可达，可以建立多跳MP-EBGP。

#### Option C2 配置关键点 (以HUAWEI设备命令为例)

```bash
# 在ASBR1上的关键配置
bgp 100
  peer <ASBR2_IP> as-number 200
  peer <ASBR2_IP> label-route-capability # 启用标签路由能力
  ipv4-family unicast
    peer <ASBR2_IP> enable
    import-route bgp # 关键：将BGP路由引入到IGP中

# 同时，需要在IGP进程（以OSPF为例）中配置重分发
ospf 1 router-id <ASBR1_RID>
  import-route bgp # 关键：将BGP路由引入OSPF
  area 0.0.0.0
    network ... # 宣告其他网段

# 在PE1上，无需与ASBR1建立IBGP，正常配置MP-EBGP即可
bgp 100
  peer <PE2_Loopback> as-number 200
  peer <PE2_Loopback> connect-interface LoopBack0
  peer <PE2_Loopback> ebgp-max-hop 64
  ipv4-family vpnv4
    peer <PE2_Loopback> enable
```

---

### 核心区别总结对比

| 特性/方面         | Option C1 (BGP标签路由)                             | Option C2 (LDP标签路由)                          |
| ------------- | ----------------------------------------------- | -------------------------------------------- |
| **核心机理**      | 使用**BGP来传播标签和路由**，构建跨AS的LSP。                    | 使用**IGP和LDP来传播路由和标签**，构建AS内的LSP。             |
| **PE-ASBR关系** | **必须建立IBGP**邻居关系。                               | **无需建立IBGP**邻居关系。                            |
| **ASBR负担**    | 较高。ASBR需要维护所有对端PE的BGP路由，并向本端PE发布，消耗更多BGP内存和CPU。 | 相对较低。ASBR只需将BGP路由引入IGP，后续由IGP和LDP处理。         |
| **路由协议交互**    | 控制平面清晰，**BGP到BGP**。                             | 涉及**BGP与IGP的重分发**，需要小心设计路由策略以防环路。            |
| **网络扩展性**     | 更优。BGP本身为大规模路由设计，适合对端PE数量多的场景。                  | 稍差。将大量BGP路由引入IGP可能对IGP的稳定性和扩展性造成压力。          |
| **故障排查**      | 排查BGP标签路由和BGP隧道。                                | 排查IGP路由、LDP绑定和路由重分发。                         |
| **适用场景**      | 大型运营商网络，追求标准性、可扩展性和控制精细度。                       | 网络规模相对较小，或者希望避免在PE和ASBR之间建立全互联IBGP的场景。       |
| **配置复杂度**     | 较高。多PE场景需要单独为每一个PE配置IBGP对等体。                    | 较低。ASBR只需将BGP路由引入IGP，所有本端PE均可通过IGP学到对端PE的路由。 |

### 总结与选择建议

- **Option C1（BGP标签路由）** 是一种**更标准、更可控、扩展性更好**的方案。它将跨域的路由和标签信息严格限制在BGP协议内部传递，符合“协议边界清晰”的最佳实践。**在华为现网中，尤其是大型网络，推荐使用此方案。**

- **Option C2（LDP标签路由）** 是一种**更简化、利用现有机制**的方案。它避免了在PE-ASBR之间建立IBGP，但引入了路由重分发，可能会带来复杂性和潜在风险（如路由环路、IGP震荡）。它通常用于网络改造或特定需求场景。

简单来说，可以这样记忆：

- **Option C1**： ASBR对PE说：“我通过BGP告诉你，去对端PE的路由和标签是这样的。”
- **Option C2**： ASBR对整个AS内部说：“我通过IGP告诉大家，去对端PE的路由在这里。”然后全网的LDP自动为这条路由分配标签。
