
# Day 7: BGP 

## Why BGP Matters
A well-implemented **Border Gateway Protocol (BGP)** plays a **pivotal** role in:
- ✅ **Inter-AS Connectivity** – BGP is the **de facto** Exterior Gateway Protocol (EGP) used to exchange routes among different autonomous systems worldwide.
- ✅ **Internet Scalability** – Handles hundreds of thousands of routes, powering global internet routing with robust policy controls.
- ✅ **Traffic Engineering** – Fine-grained manipulation of route attributes (AS-PATH, LOCAL_PREF, MED) enables load balancing, preferred path selection, and strategic routing decisions.
- ✅ **Multi-Homing** – Ensures redundancy and high availability by connecting enterprises to multiple ISPs, allowing failover or load sharing.
- ✅ **Policy Enforcement** – BGP route maps, prefix lists, and community tags let you shape inbound/outbound traffic according to business and security policies.
- ✅ **Key WAN Protocol** – Essential for large enterprises, service providers, and cloud interconnections (e.g., AWS Direct Connect, Azure ExpressRoute).

---

## Objectives
By the end of **Day 7**, you will be able to:
1. **Recognize the Purpose and Use Cases of BGP** – Understand how BGP is critical for inter-domain routing, multi-homing, and traffic engineering.  
2. **Configure Basic eBGP** – Set up external BGP sessions between different autonomous systems, ensuring essential connectivity.  
3. **Control Route Advertisements** – Learn how to advertise local prefixes and appreciate the BGP **best path selection** process.  
4. **Verify BGP Peering** – Use `show ip bgp summary` (and other commands) to monitor neighbor states, routes, and up/down times.  
5. **Grasp BGP Fundamentals** – The role of the AS-PATH attribute, NEXT_HOP, and local policies in shaping route decisions.  
6. **Set the Stage for Advanced BGP** – Prepare for upcoming topics like route filtering, attributes manipulation, iBGP scaling, confederations, and more advanced traffic engineering.

---

## 1. Purpose and Use Cases of BGP

### 1.1 Inter-AS (Exterior) Routing
- **Internet Backbone**: BGP is the foundation of global internet routing, interconnecting thousands of ASNs.
- **Service Providers**: ISPs use eBGP peering for exchanging routes with upstream/downstream providers.
- **Enterprise Edge**: Large enterprises deploy eBGP to connect to multiple ISPs, achieving redundancy and better control over external traffic paths.

### 1.2 Multi-Homing and Redundancy
- **Multiple ISP Links**: BGP is indispensable for organizations connecting to more than one provider, ensuring failover if a primary link goes down.
- **Load Balancing**: Employ BGP path attributes to balance inbound or outbound traffic across multiple links.

### 1.3 Traffic Engineering and Path Manipulation
- **Attribute Control**: Adjusting AS-PATH, LOCAL_PREF, or MED influences how traffic enters/leaves your network.
- **Policy-Based Routing**: Route maps and prefix lists let you implement custom business logic, prefer cheaper routes, or route traffic around congested links.

> 📌 **Exam Tip**: BGP is typically used **only** where you need fine control over external routes or large-scale route exchange. It’s not recommended for simple internal routing (where IGPs like OSPF/EIGRP suffice).

---

## 2. Basic eBGP Configuration

### 2.1 Assigning an Autonomous System Number
```plaintext
Router(config)# router bgp 65001
```
- Each router runs a **BGP process** identified by an **Autonomous System (AS) number**.
- Use **public ASNs** for internet routing; **private ASNs** (64512–65534) for testing or internal usage.

### 2.2 Defining an External Neighbor
```plaintext
Router(config-router)# neighbor 192.168.1.2 remote-as 65002
```
- eBGP session established with a neighbor in a **different** AS (65002).
- `neighbor <ip> remote-as <asn>` instructs the router to form a **TCP session** on port **179**.

### 2.3 Advertising Networks
```plaintext
Router(config-router)# network 10.10.10.0 mask 255.255.255.0
```
- Tells BGP to **advertise** the **10.10.10.0/24** prefix to neighbors.
- The prefix must exist in the **IP routing table** (learned via static or IGP).

> 📌 **Exam Tip**: The `network` command is not dynamic in BGP. The router only advertises **exactly** what you specify (prefix + mask) if it’s present in the RIB.

### 2.4 Confirming eBGP Configuration
- `show run | section router bgp` – Review BGP settings.
- `show ip bgp summary` – Displays BGP neighbors, routes, session states, and prefixes.

---

## 3. Route Advertisements and Best Path Selection

### 3.1 BGP UPDATE Messages
- **Advertising** new prefixes or **withdrawing** unreachable ones to neighbors.
- BGP maintains a **Routing Information Base (RIB)** and merges updates into the local table.

### 3.2 Attributes and Decision Process
BGP has a **unique** best path algorithm. Key attributes in order of importance:

1. **Highest Weight** (Cisco proprietary, local to the router).
2. **Highest LOCAL_PREF** – Commonly used to influence outbound traffic within an AS.
3. **Locally Originated** – Routes injected via `network` or `redistribute`.
4. **Shortest AS-PATH** – Minimizes the number of AS hops.
5. **Lowest ORIGIN** (IGP < EGP < Incomplete).
6. **Lowest MED** – Multi-Exit Discriminator, a hint to external neighbors about preferred entry points.
7. **Prefer eBGP** over iBGP paths if tie remains.
8. **Lowest IGP cost** to next hop.
9. **Oldest route** for route stability.
10. **Lowest neighbor BGP RID** as final tie-breaker.

> ✅ **Real-World Example**: You can prepend additional AS numbers to artificially lengthen your AS-PATH, making certain routes less preferred from an external perspective.

### 3.3 Controlling Outbound vs. Inbound Traffic
- **LOCAL_PREF**: Higher is better; shapes **outbound** traffic from your AS.
- **AS-PATH Prepending**: Lengthen your path to deter inbound traffic on certain links.
- **MED**: A suggestion to neighbor AS about preferred inbound route, though not always honored.

---

## 4. Verifying BGP Peering Using “show ip bgp summary”

### 4.1 Checking Session States
```plaintext
Router# show ip bgp summary
```
Sample Output:
```
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.1.2     4 65002     450     440       12    0    0 1d02h   25
```
- **State** should be **Established** for a full adjacency.
- **PfxRcd** shows how many routes you’re receiving from this neighbor.

### 4.2 Common BGP States
- **Idle** – Not attempting or stuck.
- **Connect** – Trying TCP handshake on port 179.
- **Active** – Attempting connection but no success yet.
- **OpenSent / OpenConfirm** – Negotiating session parameters.
- **Established** – Full adjacency; exchanging routes.

> 📌 **Exam Tip**: If stuck in **Active** or **Idle**, verify:
> - **IP reachability** between neighbors (ping).
> - **AS number** consistency (remote-as must match neighbor’s local-as).
> - **TCP port 179** not blocked by ACL or firewall.

---

## 🔥 Exam Alert: Key Focus Areas
1. **BGP Use Cases** – Inter-AS, multi-homing, advanced policy control.  
2. **Basic eBGP Configuration** – neighbor remote-as, network statements, route advertisement.  
3. **Attributes & Best Path** – Understand Weight, LOCAL_PREF, AS-PATH, MED, and how they shape decisions.  
4. **Peering Verification** – `show ip bgp summary` to check session states, routes, and stability.

