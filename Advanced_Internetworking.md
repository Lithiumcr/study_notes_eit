---
Author: August
Title: Advanced Internetworking（Markus Hidell, et al.）
Office: 
Tel: +46 8 790 42 67
Email: mahidell@kth.se
Web: https://www.kth.se/student/kurser/kurs/IK2215
Lecture Nodes (Very Important!): https://canvas.kth.se/courses/41495
Notebook: Computer Networking, 8th Edition, Kurose & Ross
Score: 

---

# Advanced Internetworking

[TOC]

## 2023-08-28

course overview

quizes P/F

3 partial digital exams give A-F grades: 0918, 1010, 1025

Lab ddl: Monday 17:00 sharp, hard

### Recapitulation

Internet, OSI, protocal stack

application-layer message

transport-layer segment

network-layer datagram

link-layer frame

**(Network layer) IP is at the heart of all communication, hourglass model**

Network interface 网络端口: connection between host/router and physical link
* Router typically has multiple network interfaces
* Host 主机 typically has one network interface (or few)
* **IP addresses associated with each network interface**

NAT network address translation 网络地址翻译

**IP subnet 子网**

* Device interfaces with same subnet part of IP address
* Can physically reach each other without intervening router

**IP address: Classless Inter Domain Routing, CIDR 无类型域间选路**

a.b.c.d/x, x=length of subnet part; host part = 32-x

mask & IP Address == netID

### Network layer 网络层

Connectionless Services 无连接服务

* The network layer treats each **packet** independently
* Route lookup for each packet (**routing table**)
* IP is **connectionless**
* IP routers are **stateless** (compared to stateful, connection-oriented networks)

#### Indirect Delivery

* From router to router, last delivery is direct

* Destination address and routing table: Routing

**next-hop routing 跳表**

#### IP-RFC 791

Following the end2end argument, only the absolutely necessary functionality is in IP
* **Best-effort 尽力而为 service**: unreliable and connectionless
* **Application or Transport layer handles reliability** 

#### ICMP—RFC 792

**Internet Control Message Protocol 互联网控制报文协议**

Error-reporting; Query

## 2023-08-29

### Transport layer

For end-to-end, process-process delivery

**Segmentation and Reassembly**

#### TCP vs. UDP

UDP – User Datagram Protocol: Connectionless unreliable service

TCP – **Transmission Control Protocol: Connection-oriented reliable stream service**

* End-to-end Flow Control (not link level flow control)
* End-to-end Error Control (not link level error control)

#### UDP-RFC 768

* Datagram-oriented transport layer protocol
* Provides **connectionless unreliable service**
* Provides optional end-to-end checksum covering header and data
* Provides no feedback to control data rate
* An UDP datagram is silently discarded if checksum errors
* UDP messages can be lost, duplicated, or arrive out of order
* Application programs using UDP must deal with reliability 
problems

---

* 面向数据报的传输层协议
* 提供无连接的不可靠服务
* 提供可选的端到端校验，涵盖标头和数据
* 不提供反馈来控制数据速率
* 如果校验和错误，UDP 数据报将被默默丢弃
* UDP 消息可能会丢失、重复或乱序到达
* 使用UDP的应用程序必须处理可靠性问题
  * DNS, DHCP, SNMP, NFS, VoIP, etc. use UDP 
  * An advantage of UDP: a base to build your own  protocols on

#### TCP—RFC 793
* TCP is a **connection-oriented transport** protocol
* TCP connection: **full duplex connection between exactly two end-points**
  * Broadcast and multicast are not applicable to TCP
* TCP provides a **reliable byte stream** service
  * A stream of 8-bit bytes is exchanged across the TCP connection
  * No record markers inserted by TCP
  * The receiving (reading) end cannot tell what sizes the individual writes were at the sending end
* TCP decides how much data to send (not the application); each unit is a segment

---
* TCP 是一个 **面向连接的传输** 协议
* TCP连接：正好两个端口间的全双工连接
   * 广播和组播不适用于 TCP
* TCP提供可靠的字节流服务
   * 通过 TCP 连接交换字节流
   * TCP 没有插入记录标记
   * 接收（读取）端无法得知发送端各写入的大小
* TCP决定发送多少数据（而不是应用程序）； 每个单元都是个报文段

Lots of applications have been implemented on top of TCP: TELNET (virtual terminal), FTP (file transfers), SMTP (email), HTTP

TCP Connection Establishment: three-way handshake 三部握手

Sliding Windows 滑窗

congestion 拥塞

……

Topics covered today should be familiar

### Network layer again

connetionless 无连接, between hosts through routers

Virtual Circuits 虚电路

VC Forwarding Table 转发表

Internet Routing Tables 互联网路由表

Longest Prefix Matching 最长前缀匹配 以二进制位判断

IP Router Model

* **The data plane is fast and special purpose – handles packet forwarding in real time**
* **The control plane is general purpose – handles routing in the background**

IPv4 lookup, and other functions

IP forwarding

IP Header

Fragmentation

MTU (Maximum Transfer Unit) 最大传输单元

ICMP—**Internet Control Message Protocol 互联网控制报文协议**

**ARP Address Resolution Protocol 地址解析协议**

**RARP Reverse Address Resolution Protocol 逆地址解析协议**

## 2023-08-31 Lab Q&A

virtual box VM, lubuntu, Kathara container, Quagga

submit via canvas

## 2023-09-04 Routers

network control-plane & data-plane

forwarding rules 转发规则

Definition. A set of network-wide forwarding rules is **valid** if:

1. there are **no dead ends**
2. there are **no forwarding loops**

Human-based control-plane is cumbersome. A network operator would have to:
* … compute valid routes …
* … in the face of network failures … 
* … and sudden variations in traffic loads …
* … and changes in security policies …
* … in a network with thousand of forwarding nodes …
* … and milions of (possibly malicious) end hosts

It does not scale. Desiderata (Latin för *ting/person att åtrå, Things desired*): network automation! -> （网络层）路由协议，选路协议

The allocation of functionality and definition of interfaces among elements. The routing “architecture” is the decision about what are the control plane tasks and where are they located: 

* What: circuit or packet switched networks?
* Where: distributed route computation on the nodes? centralized?

> Software-defined networking (SDN) NOT INCLUDED in this course

on each node: OSPF, RIP, BGP

### **link-state vs. {path,distance}-vector mechanisms**

| Each node    | learns                           | selects                              | advertises |
| ------------ | -------------------------------- | ------------------------------------ | ---------- |
| link-state   | global topology through flooding | best route over global topology |       its and others’ adjacencies     |
| vector-based | best routes from  neighbors      | best routes among the learnt ones                                     |         best route to its neighbors   |

### Dijkstra Algorithm (Shortest Path First)

```pseudocode
Qa = {a:0, b:+∞ , c:+∞ , … } stores distances to a
while Q is not empty:
	node = extract-min-key(Q)
	for each neighbor neigh of node:
		if Q[node] + c(node, neigh) < Q[neigh]:
			decrease-key(neigh: (cost + c(node,neigh)))
```

Computational complexity: **O(m+nlg(n) using a heap structure**, larger than DV 

### Open Shortest Path First—OSPF [开放最短路优先](https://zh.wikipedia.org/wiki/开放最短路径优先)

OSPF is a link-state protocol

* Builds Link State Advertisements (LSAs) – the link adjacencies
* Distributes LSAs to all other routers
* Recreate the topology from the LSAs
* Computes delivery tree using the Dijkstra algorithm

OSPF messages travel on top of IP directly (protocol field = 89)
* Not UDP or TCP. Why

#### OSPF Network Hierarchical Topology

Area 0 is the backbone area. All traffic goes via the backbone.
All other areas are connected to the backbone (1-level hierarchy)
A Border area router has one interface in each area.
An AS Boundary Router—attaches to other AS:s

#### Load Balancing across multiple paths in OSPF/ECMP

#### Alternative to OSPF: IS-IS

Link-State Routing, relies on Layer 2 transport (rather than IP)

#### Distance-vector:  Bellman-ford distributed shortest-path computation

vector-based

\- when i receives a vector from j, **R(i) vector = min(R(j) + cost(i,j) , R(i))**

must **converge** to the shortest (monotone bounded principle)

**problem: Slow convergence**

Split Horizon 水平分割, cannot solve all problems 

### Routing Information Protocol - RIP 路由信息协议
RIP-1 (RFC 1058), RIP-2 (RFC 2453)

Metric is **Hop Counts**

* 1: directly connected
* **16: infinity**
  * RIP cannot support networks with diameter > 15.

RIP uses distance vector

* RIP messages contain a vector of **hop counts**.
* Every node sends its routes to its neighbours
* Route information gradually spreads through the network
* Every node selects the route with smallest metric.

RIP messages are carried via UDP datagrams. IP Multicast (RIP-2) or Broadcast (RIP-1)

Split Horizon

#### Advantage

* Lower memory overhead than OSPF (no need to store the topology)
* Computation of shortest paths sometimes easier (think why/when)
* RIP is generally available
* simple to configure

### Summary: comparison

## 2023-09-05 Lecture 4 – Routing: Inter-domain routing 域间路由

The Internet is **huge and federated**
* Necessary to divide the routing problem into sub-problems.
* Necessary to preserve routing control among domains
* Both scalability and control can be solved with **hierarchies**

External: the Internet is partitioned into **Autonomous systems (AS) 自治系统**

* An independent administrative domain 自治域
* Routing between ASes is called **inter-domain routing**
* **Based on commercial agreements – Policies, Service-level-agreements**

Internal: An AS may be further partitioned into areas.

* Routing inside an AS: Intra-domain routing / Internal routing
* Best path based on hop/bw metrics

Autonomous Systems—RFC1930

administered by a single entity: Operators, **ISPs (Internet Service Providers)**  

intra-domain vs inter-domain routing

Inter-domain additional constraints: preserve privacy of business critical routing policies, internal topology, … while scaling to global size!

### Inter-domain routing: The Border Gateway Protocol (BGP) 边界网关协议

abstracts networks by hiding internal details

De-facto inter-domain routing protocol (**BGP-4**)

Main purpose: **Network reachability between autonomous systems**

BGP messages carried by TCP

* TCP is reliable: reduces the protocol complexity
BGP uses **path-vector - enhancement of distance-vector**
BGP **supports policies – chosen by the local administrator**

**computes best routes among the learnt one**

established over TCP on port 179

#### Architecture

BGP interacts with the internal routing (OSPF/IS-IS/RIP/...)  

iBGP AS internal

eBGP AS eternal

#### BGP Router Operation

A BGP router receives routes from
* BGP peers (eBGP and iBGP)
* Redistribution: intradomain/static routes (e.g., OSPF)

A BGP router:

* **aggregates** routes 
* filters and modifies routes
* according to some **policy**

It advertises routes to its iBGP and eBGP neighbours

#### BGP Router Model

**Path-vector** extends distance-vector

* Instead of a simple cost, assign an **AS-Path** to every route
* There may be many paths to the same destination (network prefix)
* AS-Path used to implement **policies** and loop prevention
* **supports expressive routing policies** (avoid an AS, prefer routes regardless of AS-path length

---

* 为每条路由分配一个 AS-Path，而不是简单的成本
* 到同一目的地（网络前缀）可能有多条路径
* AS-Path用于实施策略和环路预防
* 支持表达性路由策略（避开某 AS；无论 AS 路径长度如何都优先选择路由）

#### External BGP scaling

#### Internal BGP scaling

**eBGP>IGP>iBGP**

#### scalability: BGP route aggregation 路由聚合

smaller routing tables, fewer prefixes

Threats: multi-homing 多宿主 and load-balancing 负载均衡

**Hierarchies: sub-AS divide-and-conquer 子自治域分治法**

#### Distributed Bellman Ford routing

**no guaranteed convergence; high routing expressiveness**

BGP Routing instabilities: the Bad Gadget  

## 2023-09-06 Lecture 5—Multicast 多播/组播

### IP Multicast Applications

### IP Multicast: Abstraction of HW Multicast

IP-multicast addresses, **class D addresses (binary prefix: 1110) 224.0.0.0 - 239.255.255.255**

28 bit multicast group id

### IP Multicast Service Model

Link-level/Hardware Multicast + Host-Router Protocal + Multicast Routing Protocals

链路层硬件多播+主机-路由协议+多播选路协议

* **PIM Protocol Independent Multicast 独立多播协议**
* **CBT Core-based trees 基于核心树**
* **DVMRP Distance Vector Multicast Routing Protocol 距矢多播选路协议**
* **MOSPF Multicast extension to OSPF 开放最短路优先的多播扩展**
* **MBGP Multiprotocol Extensions for BGP 边界网关协议的多播扩展**

#### Link-level/Hardware Multicast

E.g. Ethernet Network Interface Card support multicast. Multicast addresses: The low order bit of the high order byte is 1:

```
*1:**:**:**:**:**
```

Many NICs on the same network may listen to the same Ethernet multicast address

Other Link-level layers may not support multicast: ATM Asynchronous Transfer Mode 异步传输模式, Frame Relay 帧中继, X.25 分封交换网 Packet switched network ... But multicast can still be implemented over these!

#### Mapping IP Multicast to Ethernet: Translation

the IP multicast address is translated to an **Ethernet multicast address**.

The 23 low order bits of the IP multicast address, placed in the 23 low order bits of the Ethernet MAC address: `0x 01:00:5E:00:00:00`

IP to Ethernet multicast address mapping is **not unique**! 

* 32:1 overlap
* IP may receive multicast despite the lack of receiving process
* IP-layer must be able to do filtering (based on IP multicast address)

---

* 32:1 重叠
* 尽管缺少接收过程，IP仍可能接收组播
* IP层必须能够进行过滤（基于IP组播地址）

### IGMP—Internet Group Management Protocol 互联网组管理协议 v1, v2, v3

**Group membership** communication between hosts and multicast routers, not for routing of multicast packets

#### Position of IGMP in TCP/IP

Network Layer, **Encapsulated in IP** (like ICMP) 

always addressed to a multicast address

* often **all systems (224.0.0.1), all routers (224.0.0.2)**
* or to a specific multicast group  

#### IGMPv2 Messages

General membership query: Sent regularly by **routers** to query all membership
Specific membership query: Sent by **routers** to query specific group membership
Membership report: Sent by **hosts** to report joined groups
Leave group: Sent by **hosts** to leave groups

#### Host Behaviour and Dynamics

A process joins a multicast group on a given interface
* **Host sends IGMP report to group address when first process joins a group.**
– Host keeps a table of all groups which have a reference count > 0
* **Host sends IGMP Leave to 224.0.0.2 when last process leaves group**
– In IGMPv1 hosts did not send explicit leaves
* **Router sends IGMP queries to 224.0.0.1 at regular intervals.**
– general query: group = 0.0.0.0
– specific group query: group = multicast address of the group
* **Host responds to IGMP query by sending IGMP report** to group address
– Hosts snoop for other hosts’ reports
– Set random timer Suppress if other host on same segment sends it

#### IGMPv3

Enables: Source specific multicast 

A host can join a group and specific sender:

    (S, G) not only (*, G)

This may allow for pruning of certain senders

IGMPv3 is not commonly deployed

#### Multicast Router

Listens to all multicast traffic and forwards if necessary. 

Multicast router listens to **all** multicast addresses.

* Ethernet: 2^23 link layer multicast addresses
* Listens promiscuously 混杂侦听 to all LAN multicast traffic  

Communicates with directly connected hosts: via **IGMP**

Communicates with other multicast routers: **multicast**
**routing protocols**

The **network** replicates the packets—not the hosts

**Multicast is not Multiple Unicast** 多个单播

#### Multicast Delivery Trees 多播分发树

A tree structure, described multicast packets route from the sender to receivers. The sender is a root of the tree, and receivers are leaves. Build a **delivery tree** through a network: 

**Source Based Trees vs. Group Shared Trees**

**Source Based Trees (DVMRP MOSPF PIM-DM)**

* Each router needs to have one shortest path tree for each group
* Notation: (S, G)
* Uses more memory (O(S*G)), but can give optimal paths and delay

**Group Shared Trees (CBT PIM-SM)**

* One router (**center core or renedevous router**) is responsible for distributing multicast traffic
* Other routers encapsulates multicast packets in unicast packets and send them to the rendevous point for multicast distribution
* Notation: (*, G)
* Uses less memory (O(G)) but suboptimal paths and delays

---
基于源的树
* 每个路由器需要为每个组拥有一棵最短路径树
* 符号：(S, G)
* 使用更多内存 (O(S*G))，但可以提供最佳路径和延迟

组共享树

* 一台路由器（中心核心或独立路由器）负责分发组播流量
* 其他路由器将组播报文封装在单播报文中，发送到会合点进行组播分发
* 符号：(*, G)
* 使用较少的内存 (O(G))，但路径和延迟不是最优的

**Distance-Vector Multicast Routing Protocol - DVMRP**

* Based on unicast distance vector (e.g., RIP)
* Routers do not know network topology apart from closest neighbour
* Create multicast routing table by using information from the unicast distance vector tables
* Extend (Destination, Cost, Nexthop) ->  (Group, Cost, Nexthops)

DVMRP is **data-driven and uses source-based trees**

DVMRP uses **Reverse Path Multicasting (RPM)** 

---

* 基于单播距离向量（例如，RIP）
* 路由器不知道除了最近邻居之外的网络拓扑
* 使用单播距离向量表中的信息创建多播路由表
* 扩展（目的地、成本、下一跳） -> （组、成本、下一跳）

DVMRP 是**数据驱动的并使用基于源的树**

DVMRP 使用逆向路径多播 (RPM)

**Reverse Path Forwarding (RPF) 逆向转发**

Forward a multicast datagram only if it arrives on the interface that would be used to send unicast to the source - Send out on all other interfaces - Flooding!

仅当多播数据报到达用于向源发送单播的**那个**端口时，才转发多播数据报 - 至其余所有端口 - 洪泛

Make a lookup of the source address in the FIB 转发信息表

only shortest path packet forwarded

**Reverse Path Multicasting (RPM) 逆向路径多播** 

RPM refines RPF as follows

Only designated parent router may forward multicast packets from a source to a link 只有指定的亲路由可以将多播包从源转发到链路

* Removes duplicates

Flooding 洪泛 (Build the tree)
* First packet broadcast to every network 

Pruning 剪枝 (Cut the tree)
* Prune networks that do not have members
* IGMP leave (or timeout)
* **Propagate prune messages up the shortest path tree**

Grafting 嫁接 (Add a branch to the tree)
* Add a network with a listener
* IGMP join
* **Propagate graft messages up the shortest path tree**

---

洪泛（植树）
* 第一个数据包广播到每个网络

剪枝（砍树）
* 修剪没有成员的网络
* IGMP离开（或超时）
* **沿着最短路径树传播剪枝消息**

Grafting 嫁接（在树上添加一根树枝）
* 添加一个带有监听器的网络
* IGMP加入
* **沿最短路径树向上传播嫁接消息**

**Link-State Multicast: MOSPF 开放最短路优先的多播扩展**

**Add multicast to a given link-state routing protocol**

Uses the multiprotocol facility in OSPF to carry multicast information; Extend LSAs with group-membership LSA

* Only containing members of a group

Uses the link-state database in OSPF to build delivery trees
* Every router knows the topology of the complete network
* Least-cost source-based trees using metrics
* One tree for all (S,G) pairs with S as source

Expensive to keep all this information
* Cache active (S,G) pairs 
* MOSPF is **Data-driven**: computes Dijkstra when datagram arrive

#### Core Based Tree—CBT 基于核心树

**Group shared** multicast trees—(*, G)

**Demand-driven**

Routers send join messages when hosts join groups
Divide the Internet into regions where each region has a core router
When a host joins a multicast group the nearest multicast router attaches to the forwarding tree by sending a join request towards its core router
Multicast datagrams to the core router are encapsulated in unicast datagrams

---

当主机加入组时，路由器发送**加入**消息
将互联网划分为多个区域，每个区域都有一个核心路由器
当主机加入多播组时，最近的多播路由器通过向其核心路由器发送加入请求来连接到转发树
发送到核心路由器的组播数据报被封装在单播数据报中

#### Protocol Independent Multicasting—PIM

PIM-DM (dense mode)

* For dense multicast environment, like a LAN
* Uses RPF and pruning/grafting strategies—similar to DVMRP
  * Source-based tree
* Does not depend on a specific unicast protocol
* Relies on (any) correct unicast routing tables

PIM-SM (sparse mode)

* For non-broadcast environment (routers involved)
* Demand driven similar to CBT
  * uses **rendezvous points (RPs)** instead of core routers
* Extends CBT in that a router may know of more than one rendezvous point
* Can build both shared and source distribution trees

---

PIM-DM（密集模式）

* 适用于密集组播环境，如 LAN 局域网
* 使用 RPF 和剪枝/嫁接策略 - 类似于 DVMRP
   * 基于源的树
* 不依赖于特定的单播协议
* 依赖于（任何）正确的单播路由表

PIM-SM（稀疏模式）

* 针对非广播环境（涉及路由器）
* 类似于 CBT 的需求驱动
   * 使用**会合点 (RP)** 代替核心路由器
* 扩展 CBT，使路由器可以知道多个会合点
* 可以构建共享树和源分发树

#### Multicast Source Discovery Protocol, MSDP **多播源发现协议**

Interconnects multiple PIM-SM domains
* Enables rendevous-point (RP) redundancy
* Enables inter-domain multicasting

Tunnels can be configured between RPs in various domains
* RPs speak MSDP to each other
* Enough tunnels so that we have connectivity even when an RP fails

Drawbacks:
* Scaling problem—many (S,G) pairs can be active in the Internet
  * Info must be passed about all these pairs
* Configuration-intensive (many tunnels needed)

---

多个PIM-SM域互联
* 启用会合点 (RP) 冗余
* 启用域间组播

不同域内的RP之间可以配置隧道
* RP 互相讲 MSDP
* 足够多的隧道，即使 RP 发生故障也能保持连接

缺点：
* 扩展问题——许多（S，G）对可以在互联网上活跃
   * 必须传递所有这些对的信息
* 配置密集（需要许多隧道）

used in KTH Kista

#### MBGP 边界网关协议的多播扩展

Solves part of the inter-domain problem

Standard BGP configuration facilities

* Extends the BGP multiprotocol attributes
* Exchange multicast routing information
* Policies, capabilities,

Must still use, for example, PIM to build distribution trees and forward multicast traffic

**Tunneling**

can use tunneling over non-multicast enabled sub-nets

Multicast BackBONE 

**Deployment**

Multicast routing is in general **not** deployed in the current networks

Some sites (e.g., metropolitan area networks) have deployed local multicast delivery： Cable TV distribution

IP multicast is slowly gaining acceptance

### Summary

Multicast routing uses network resources more efficiently than unicast emulation

IP multicast

* Receiver-based
* Best effort delivery

Multicast routing protocols
* **DVMRP, MOSPF, CBT, PIM, MBGP**

Source-based trees vs shared group trees

Demand-driven vs Data-driven trees

Reverse Path Forwarding (RPF)

Reverse Path Multicasting (RPM)

## 2023-09-07 Project Introduction

https://canvas.kth.se/courses/41495/pages/project-assignment?module_item_id=710191

GUDPSocket.java file testing：

http://ik2215.ssvl.kth.se/

## 2023-09-11 Transport layer 运输层

Support communication between applications:
* **need** a way to distinguish between applications’ packets (**mux/demux （解）复用**)
* may offer **reliable service**
* may **dictate the transmission rate** of packets, like **congestion control 拥塞控制**
* may offer **encrypted communication**

#### Connectionless communication

low data delivery time (in the absence of congestion), but with risk of security attacks like DDoS

#### Unreliable transport protocols without congestion control, e.g., UDP

#### Reliable transport protocols without congestion control, e.g., the Internet in 1986

Advantages:
* All packets are eventually delivered
* low data delivery time (in the absence of congestion)

Disadvantages:

* **congestion collapse**!

--------knee cliff
thrpt	𠂆\
delay	丿

#### Reliable transport protocols, e.g., TCP

TCP congestion control

The way TCP adapt the sending window is crucial
$$
Throughput=\frac{Window_{avg}Packetsize}{RTT}\\
T_{stable}=\frac{0.75W_{max}P}{RTT}\\
Consider\ 1\ period:Sent=\frac{W_{max}}{2}RTT\frac{0.75W_{max}}{RTT}=\frac38{W_{max}^2}\\
p=P(1packetdrop)\le\frac1{\frac38{W_{max}^2}}\\
\rarr W_{max}\le\sqrt\frac{8}{3p}\\
\rarr T\le\sqrt\frac32 \frac{P}{RTT\sqrt p}
$$

#### Traffic fairness: the parking lot example

**Fairness in TCP: Multiple flows with same RTT sharing same link should get same bandwidth**

TCP is **AIMD 加增倍减**, with gentle ↑ aggressive ↓, converges to fairness

#### router buffer size impact T

$$
C=\frac{W_{max}P}2 \frac{1}{RTT}\le \frac {Buffer}{RTT} \rarr B\ge C\times RTT
$$

#### Improving throughput and convergence: TCP Cubic

beta factor

try to return to W_max ASAP, converging faster

#### Internet fragility

**high buffer occupancy** remains an open problem

**min-RTT and max-BW**

#### Bottleneck Bandwidth and Round-trip propagation time (BBR) congestion control

([slides-101-iccrg-an-update-on-bbr-work-at-google-00 (ietf.org)](https://datatracker.ietf.org/meeting/101/materials/slides-101-iccrg-an-update-on-bbr-work-at-google-00))

### TCP + TLS (Transport Layer Security 传输层安全性协议，原SSL 安全套接层，工作于应用层)

problems with TCP and TCP + TLS and HTTP 1.1, HTTP 2

...

#### QUIC: Quick UDP Internet Connections 

> paper: Yong Cui et al. ”Innovating Transport with QUIC: Design  Approaches and Research Challenges ”. In IEEE Internet Computing

QUIC ideas were already proposed in the past:
* **encryption** is based on TLS 1.3
* **0-RTT connection** similar of TCP Fast Open
* **stream multiplexing** similar of **SCTP 流控传输协议, SST 结构化流传输**, …
QUIC key ideas on deplayability and evolvability:
* build over UDP (pass through middleboxes, userspace implementation). There exists BBR open source over QUIC
* mostly encrypted headers (avoids “network ossification”)

### Optional: buffers on the receiver host



## 2023-09-13 P2P Network 对等网

### Application Architecture 应用体系结构

Client-Server Architecture

time to distribute File to N clients using client-server approach

$$
D_{C-S}\ge \max \{\frac{NF}{u_S}, \frac{F}{d_{min}}\}\\
D_{p2p}\ge \max \{\frac{F}{u_S}, \frac{F}{d_{min}}, \frac{NF}{u_S+\sum u_i}\}\\
u_S=server\ upload\ rate,\ d_{min} = min\ client\ download\ rate
$$

**Pure P2P is highly scalable but difficult to manage**

#### Operation

Mapping keys and resources to IP addresses, key -> number

**Static: localization, distribution**

**Dynamic: joining, leaving**

#### e.g. Centralized Directory: Napster

file transfer is decentralized, but locating content is highly centralized

**O(N) memory**

#### e.g. Gnutella Protocol

Gnutella Query Flooding

**O(N) communication overhead**

Issue: Can ”see” only local content. Some content invisible to a node.

#### e.g. Hierarchical Overlay Network

Each peer is either a group leader or assigned to a group leader

TCP connection between peer and its group leader; between some pairs of group leaders (overlay)  

Flooding limited to overlay of super peers

#### e.g. Distributed Hash Table—DHT 分布式散列表

> Stoica et al. ”Chord: A Scalable Peer-to-peer Lookup Service for  Internet Applications”. In SIGCOMM 2001, doi http://nms.csail.mit.edu/papers/chord.pdf

Hash functions are **stateless load balancers**, giving most likely a fairly uniform (not perfect) load balance. Widely adopted in P2P systems.

Creates a fully decentralized index that maps file IDs to locations; Allows a user to determine the location of a file, without generating an excessive amount of search traffic

**Locating and distributing content**

##### Chord DHT, Developed at MIT

**operations: insert(id, item); lookup(id)** 

m=160, assume =O(logN), hash() return id: [0, 2^m-1]

Each peer has **authority over its id and all the smaller ones until its predecessor peer 自身及至前驱对等点（不含）的那些区间id的权限; a pointer to its successor 后继**

**Routing**: simple algorithm 1, 2, and **Finger tables** keep log(n) successors at increasing distances: 

Peer k stores information about peer authority

$$
k + 2^i \mod 2^m,\ i=0,...,m-1
$$

closest smaller peer 最近的偏小对等点

**Routing with finger tables**

Each node stores a subset of successors: **O(log N) memory**

The search space is halved at each hop: **O(log N) communication overhead**

**More robust**: unless the authority peer of the key ID fails, lookup operations work correctly

**Joining**

1. computes its own id
2. computes its own finger table
3. ask any peer who has authority over id
4. trigger update of others’ tables
   1. without creating anomalies
   2. update succ/pred pointers
   3. safe to move resource mapping
   4. trigger finger tables update

*Theorem.* The communication overhead for updating all the tables is $O(log^2 N)$ on average

#### Optional: P2P file distribution: BitTorrent

Optional: Optional: Free-Riding Peer: downloading files from the system without ploading, A common problem in peer-to-peer file sharing.

Optional: Free-Riding: BitTorrent trading algorithm reduces free-riding

#### Peer-to-peer Internet Telephony—Skype

Unclear

#### Bitcoin: peer-to-peer payments

Requirements for a P2P payment systems:
* **tamper-proof persistence**
* **transaction consistency** (no double spending)
  * requires reaching some sort of consensus

DHT (e.g., Chord) is not suited for this task:

* **malicious** users may alter or remove data

Solution:

* ”proof-of-work” consensus

#### The Inter-Planetary File System (IPFS)

## 2023-09-18 Partial Exam A-1

## 2023-09-21

