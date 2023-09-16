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

**IP subnet 子网**

* Device interfaces with same subnet part of IP address
* Can physically reach each other without intervening router

**Classless Inter Domain Routing, CIDR 无类型域间选路**

a.b.c.d/x

mask & IP Address == netID

### Network layer

Connectionless Services

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

### network layer

connetionless, between hosts through routers

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

ICMP—Internet Control Message Protocol 互联网控制消息协议

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

It does not scale.

Desiderata: network automation! -> 路由协议，选路协议

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

Link-State Routing

relies on Layer 2 transport (rather than IP)

#### Distance-vector:  Bellman-ford distributed shortest-path computation

vector-based

\- when i receives a vector from j, **R(i) vector = min(R(j) + cost(i,j) , R(i))**

must **converge** to the shortest (monotone bounded principle)

**problem: Slow convergence**

Split Horizon 水平分割, cannot solve all problems 

### Routing Information Protocol - RIP 路由信息协议
RIP-1 (RFC 1058), RIP-2 (RFC 2453)

Metric is Hop Counts

* 1: directly connected
* 16: infinity
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

## 2023-09-05 Lecture 4 – Routing: Inter-domain routing

The Internet is huge and federated
* Necessary to divide the routing problem into sub-problems.
* Necessary to preserve routing control among domains
* Both scalability and control can be solved with hierarchies
External: the Internet is partitioned into Autonomous systems (AS)
* An independent administrative domain
* Routing between ASes is called inter-domain routing
* Based on commercial agreements – Policies, Service-level-agreements
Internal: An AS may be further partitioned into areas.
* Routing inside an AS: Intra-domain routing / Internal routing
* Best path based on hop/bw metrics

Autonomous Systems—RFC1930

intra-domain vs inter-domain routing

### inter-domain routing: The Border Gateway Protocol (BGP) 边界网关协议

abstracts networks by hiding internal details

De-facto inter-domain routing protocol (version 4)
Main purpose: Network reachability between autonomous systems
BGP messages carried by TCP
* TCP is reliable: reduces the protocol complexity
BGP uses path-vector - enhancement of distance-vector. 
BGP supports policies – chosen by the local administrator

**computes best routes among the learnt one**

#### Architecture

Autonomous system, AS 自治域

iBGP ASinternal

eBGP AS exo

#### BGP Router Operation

A BGP router receives routes from
* BGP peers (eBGP and iBGP)
* Redistribution: intradomain/static routes (e.g., OSPF)

A BGP router:

* aggregates routes 
* filters and modifies routes
* according to some policy

It advertises routes to its iBGP and eBGP neighbours

#### BGP Router Model

**Path-vector** extends distance-vector

* Instead of a simple cost, assign an AS-Path to every route
* There may be many paths to the same destination (network prefix)
* AS-Path used to implement policies and loop prevention
* supports expressive routing policies (avoid an AS, prefer routes regardless of AS-path length

---

* 为每条路由分配一个 AS-Path，而不是简单的成本
* 到同一目的地（网络前缀）可能有多条路径
* AS-Path用于实施策略和环路预防
* 支持表达性路由策略（避开某 AS；无论 AS 路径长度如何都优先选择路由）

#### External BGP scaling

#### Internal BGP scaling

eBGP>IGP>iBGP

Distributed Bellman Ford routing: convergence vs expressiveness

## 2023-09-06 Lecture 5—Multicast

### IP Multicast Applications

### IP Multicast: Abstraction of HW Multicast

IP-multicast addresses, class D addresses (binary prefix: 1110) 224.0.0.0 - 239.255.255.255

28 bit multicast group id

### Link-level/Hardware Multicast

Ethernet multicast addresses:

The low order bit of the high order byte is 1:

```
*1:**:**:**:**:**
```

Many NICs on the same network may listen to the same Ethernet multicast address

### Mapping IP Multicast to Ethernet: Translation

IP to Ethernet multicast address mapping is not unique! 
* 32:1 overlap
* IP may receive multicast despite the lack of receiving process
* IP-layer must be able to do filtering (based on IP multicast address)

### IGMP—Internet Group Management Protocol 互联网组管理协议

Group membership communication between hosts and multicast routers

#### Position of IGMP in TCP/IP

#### IGMPv2 Messages

Position of IGMP in TCP/I

#### Host Behaviour and Dynamics

#### v3

#### Multicast Router

Listens to all multicast traffic and forwards if necessary. 

Multicast router listens to all multicast addresses.

The network replicates the packets—not the hosts

**Multicast is not Multiple Unicast** 多个单播

#### Delivery Trees

Build a **delivery tree** through a network

Source Based Trees vs. Group Shared Trees

**Distance-Vector Multicast Routing Protocol - DVMRP**

**Reverse Path Forwarding (RPF)**

Forward a multicast datagram only if it arrives on the interface that would be  used to send unicast to the source - Flooding!

Make a lookup of the source address in the FIB

only shortest path packet forwarded

**Reverse Path Multicasting (RPM)** 

RPM refines RPF as follows

Only designated parent router may forward multicast packets from a source to a link 只有指定的亲路由可以将多播包从源转发到链路

* Removes duplicates

Flooding 洪泛 (Build the tree)
* First packet broadcast to every network 

Pruning 剪枝 (Cut the tree)
* Prune networks that do not have members
* IGMP leave (or timeout)
* **Propagate prune messages up the shortest path tree.**

Grafting 嫁接 (Add a branch to the tree)
* Add a network with a listener
* IGMP join
* **Propagate graft messages up the shortest path tree**

**Link-State Multicast: MOSPF**

**Add multicast to a given link-state routing protocol**

Uses the multiprotocol facility in OSPF to carry multicast information; Extend LSAs with group-membership LSA

* Only containing members of a group

Uses the link-state database in OSPF to build delivery trees
* Every router knows the topology of the complete network
* Least-cost source-based trees using metrics
* One tree for all (S,G) pairs with S as source

Expensive to keep all this information
* Cache active (S,G) pairs 
* MOSPF is Data-driven: computes Dijkstra when datagram arrive

#### Core Based Tree—CBT

#### Protocol Independent Multicasting—PIM

#### Multicast Source Discovery Protocol MSDP

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

used in KTH Kista

#### MBGP

### Tunneling

can use tunneling over non-multicast enabled sub-nets

Multicast BackBONE 

### Deployment

Multicast routing is in general not deployed in the current networks
Some sites (e.g., metropolitan area networks) have deployed local multicast delivery

* Cable TV distribution
IP multicast is slowly gaining acceptance

### Summary

Multicast routing uses network resources more efficiently than unicast emulation IP multicast
* Receiver-based
* Best effort delivery

Multicast routing protocols
* DVMRP, MOSPF, CBT, PIM, MBGP

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
* may **dictate the transmission rate** of packets, like **congestion control**
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

TCP is AIMD, with gentle ↑ aggressive ↓, converges to fairness

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

### TCP + TLS (Transport Layer Security)

problems with TCP and TCP + TLS and HTTP 1.1, HTTP 2

#### QUIC

> paper: Yong Cui et al. ”Innovating Transport with QUIC: Design  Approaches and Research Challenges ”. In IEEE Internet Computing

QUIC ideas were already proposed in the past:
* encryption is based on TLS 1.3
* 0-RTT connection similar of TCP Fast Open
* stream multiplexing similar of SCTP, SST, …
QUIC key ideas on deplayability and evolvability:
* build over UDP (pass through middleboxes, userspace
implementation). There exists BBR open source over QUIC
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



Locating and distributing content Gnutella Query Flooding

### Distributed Hash Table—DHT

Hash functions are **stateless load balancers**

#### Operation

Static: localization, distribution

Dynamic: joining, leaving

#### e.g. Chord DHT Developed at MIT

> Stoica et al. ”Chord: A Scalable Peer-to-peer Lookup Service for  Internet Applications”. In SIGCOMM 2001, doi http://nms.csail.mit.edu/papers/chord.pdf

**Finger tables** keep log(n) successors at increasing distances

closest smaller peer

**Routing with finger tables**

Each node stores a subset of successors: **O(log N) memory**

The search space is halved at each hop: **O(log N) communication**

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

*Theorem.* The communication overhead  for updating all the tables is $O(log^2 N)$ on average

#### Optional: P2P file distribution: BitTorrent

Optional: Optional: Free-Riding Peer: downloading files from the system without ploading, A common problem in peer-to-peer file sharing.

BitTorrent trading algorithm reduces free-riding

#### Peer-to-peer Internet Telephony—Skype

#### Bitcoin: peer-to-peer payments

Requirements for a P2P payment systems:
* tamper-proof persistence
* transaction consistency (no double spending)
* requires reaching some sort of consensus

DHT (e.g., Chord) is not suited for this task:

* malicious users may alter or remove data

Solution:

* ”proof-of-work” consensus

#### The Inter-Planetary File System (IPFS)