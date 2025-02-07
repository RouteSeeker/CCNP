# Day 6: Day 6: OSPFv2 and Advanced OSPFv2/OSPFv3

## Why OSPF Matters
A well-tuned **Open Shortest Path First (OSPF)** network enables:
- ✅ **Fast Convergence** using the Dijkstra SPF algorithm.
- ✅ **Scalability** through multi-area configurations (Backbone, Stub, NSSA).
- ✅ **Loop-Free Routing** with Link-State Advertisements (LSAs) and a global Link-State Database (LSDB).
- ✅ **IPv6 Support** via OSPFv3, which can also carry IPv4 routes.
- ✅ **Security** options, including MD5 (OSPFv2) or IPsec (OSPFv3).
- ✅ **Minimal Overhead** through area segmentation, route summarization, and LSA filtering.
- ✅ **Designated Router (DR) / Backup DR (BDR)** mechanics that reduce flooding on multi-access networks.
- ✅ **Integration** with MPLS VPNs, external route injection, and advanced redistribution.

---

## Objectives
By the end of **Day 6**, you will be able to:

1. **Understand OSPF Fundamentals** – LSA types, Link-State Database, and SPF calculations.  
2. **Configure and Verify OSPFv2** – Basic deployment for IPv4, including neighbor formation and DR/BDR elections.  
3. **Explore OSPFv3** – Extend OSPF to IPv6 and learn differences from OSPFv2.  
4. **Optimize OSPF** – Use performance tuning, route summarization, and advanced area types (Stub, NSSA) to manage large topologies.  
5. **Secure OSPF** – Implement MD5 (OSPFv2) or IPsec (OSPFv3) authentication to protect routing updates.  
6. **Implement DR/BDR Election Optimizations** – Understand priorities and stable DR/BDR roles in multi-access networks.  
7. **Integrate OSPF over MPLS** – Deploy OSPF as a PE-CE protocol and leverage sham-links for correct route preference.  
8. **Prevent Loops and Redundancy** – Redistribute routes, apply route-tagging, and use LSA filtering.  
9. **Plan Migration Strategies** – Transition from OSPFv2 to OSPFv3 or run dual-stack networks.  
10. **Troubleshoot OSPF** – Diagnose neighbor adjacency issues, SPF recalculations, and LSDB synchronization problems.

---

## 1. OSPF Fundamentals

### 1.1 Link-State Advertisements (LSAs) and the LSDB
- **LSAs** describe the router’s interfaces, connections, and external routes.  
- Collected LSAs form the **Link-State Database (LSDB)**, used by the **SPF algorithm** to calculate shortest paths.  
- **Type 1** (Router LSA), **Type 2** (Network LSA), **Type 3** (Summary LSA), **Type 5** (External LSA), etc.

> 📌 **Exam Tip**: Know how Type 5 LSAs represent external routes in OSPFv2 and how Type 7 LSAs work in NSSA areas.

### 1.2 How SPF Calculates Routes
1. Each OSPF router runs **Dijkstra’s algorithm** locally.  
2. Routes are computed from the router’s perspective to every destination.  
3. **Shortest path tree** is generated and placed in the **routing table**.

> ✅ **Key Benefit**: Link-state routing allows faster convergence since each router has a **complete view** of the network.

---

## 2. OSPF Area Types: Backbone, Stub, and NSSA

### 2.1 Backbone Area (Area 0)
- The **primary** OSPF area through which all other areas must connect.
- **ABRs** (Area Border Routers) sit between the backbone and non-backbone areas.

### 2.2 Stub and Totally Stubby Areas
- **Stub**: Blocks Type 5 External LSAs, using a default route instead.  
- **Totally Stubby**: Blocks Type 3, 4, and 5 LSAs, drastically reducing LSA flooding.

> 📌 **Exam Tip**: Configure a **stub** or **totally stubby** area to reduce routing overhead in branch offices.

### 2.3 NSSA (Not-So-Stubby Area)
- Similar to a stub area but **allows** external routes in the form of **Type 7 LSAs**, later converted to Type 5 by the ABR.  
- Useful when a branch site must inject local external routes but still wants limited LSAs from the backbone.

> ✅ **Real-World Example**: A branch that has a small ISP link but mainly relies on a hub site for connectivity.

---

## 3. OSPFv2 Configuration Basics

### 3.1 Enabling OSPF for IPv4
```plaintext
Router(config)# router ospf 1
Router(config-router)# network 192.168.1.0 0.0.0.255 area 0
Router(config-router)# network 10.10.10.0 0.0.0.255 area 1
```
- OSPF uses **wildcard masks**.  
- The **area** assignment determines which area each interface belongs to.

### 3.2 Verifying Neighbor Relationships
- `show ip ospf neighbor` → Check adjacency states and see DR/BDR.  
- `show ip ospf interface` → Verify Hello/Dead intervals, priority, and area info.  
- `show ip route ospf` → Inspect installed OSPF routes.

> 📌 **Exam Tip**: If **neighbors** aren’t forming, check **area IDs, IP addressing, network statements, MTU sizes, and authentication**.

---

## 4. DR/BDR Election Optimizations

### 4.1 Understanding DR/BDR Roles
- On **broadcast** or **NBMA** networks (like Ethernet), a **Designated Router (DR)** and **Backup DR** are elected to reduce LSA flooding.  
- The **router with the highest priority** wins DR (if tied, highest Router ID).

### 4.2 Manipulating OSPF Priority
```plaintext
Router(config-if)# ip ospf priority 255
```
*(Forces a router to become DR.)*

```plaintext
Router(config-if)# ip ospf priority 0
```
*(Prevents a router from becoming DR or BDR.)*

> ✅ **Recommended**: Manually setting priority avoids **unpredictable** DR changes in large, critical segments.

---

## 5. OSPF Over MPLS and VPN Integration

### 5.1 OSPF as PE-CE Protocol
- Service providers often use OSPF to connect **customer edge (CE)** routers.
- Configure **VRF**-aware OSPF for each customer’s VPN instance:
```plaintext
Router(config-router)# router ospf 1 vrf CUSTOMER_VRF
Router(config-router)# network 172.16.0.0 0.0.255.255 area 0
```

### 5.2 OSPF Sham-Link in MPLS VPNs
- **Sham-links** are virtual links that preserve **customer’s OSPF** across the MPLS core.
- Ensures the **customer’s internal OSPF routes** aren’t overshadowed by MPLS default routing.

> 📌 **Exam Tip**: Sham-links are critical to maintaining correct **OSPF path selection** in MPLS-based networks.

---

## 6. OSPFv3 for IPv6 (and IPv4)

### 6.1 Interface-Based Configuration
```plaintext
Router(config)# ipv6 unicast-routing
Router(config)# router ospfv3 10
Router(config-router)# address-family ipv6 unicast
Router(config-if)# ipv6 ospf 10 area 0
```
- No `network` commands; OSPFv3 is **enabled per interface**.

### 6.2 OSPFv3 with Dual-Stack
- OSPFv3 can **carry IPv4** routes too:
```plaintext
Router(config-router)# address-family ipv4 unicast
Router(config-router)# address-family ipv6 unicast
```
> ✅ **Single OSPFv3 process** for both IPv4 and IPv6 → Reduces complexity in dual-stack deployments.

---

## 7. Advanced OSPF Concepts: Route Summarization, Filtering, and External Routes

### 7.1 Summarization at ABRs and ASBRs
- **Inter-Area Summarization** (ABR):
  ```plaintext
  Router(config-router)# area 1 range 10.1.0.0 255.255.0.0
  ```
- **External Summarization** (ASBR):
  ```plaintext
  Router(config-router)# summary-address 192.168.0.0 255.255.248.0
  ```

> 📌 **Exam Tip**: Summarizing reduces LSA flooding and minimizes CPU usage on routers.

### 7.2 LSA Filtering and Route-Map Tagging
- **Distribute-Lists** block certain LSAs from entering the routing table:
  ```plaintext
  Router(config-router)# distribute-list prefix BLOCK_ROUTES in
  ```
- **Tagging** helps avoid loopbacks when redistributing between OSPF, BGP, and EIGRP:
  ```plaintext
  Router(config-route-map)# set tag 100
  Router(config-router)# redistribute ospf 1 route-map OSPF_TO_BGP
  ```

### 7.3 External Route Injection
- Type 5 LSAs for **external routes** (redistributed from another protocol).
- Type 7 LSAs in NSSA → converted to Type 5 at the ABR.

---

## 8. OSPF Authentication and Security

### 8.1 OSPFv2 (IPv4) Authentication
1. **MD5** or Cleartext configured on interfaces:
   ```plaintext
   Router(config-if)# ip ospf authentication message-digest
   Router(config-if)# ip ospf message-digest-key 1 md5 MY_KEY
   ```
2. **Area-Wide Authentication**:
   ```plaintext
   Router(config-router)# area 1 authentication message-digest
   ```

> ✅ **Ensures** only routers with the same key can form adjacency → blocks rogue devices.

### 8.2 OSPFv3 (IPv6) IPsec Authentication
```plaintext
Router(config-if)# ipv6 ospf authentication ipsec spi 10 esp sha256 SECUREKEY
```
- Stronger encryption than MD5; supports advanced crypto algorithms (SHA-256, etc.).

---

## 9. OSPF Performance Tuning and Convergence

### 9.1 SPF and LSA Throttle Timers
- **SPF Throttle**:
  ```plaintext
  Router(config-router)# timers throttle spf 5 100 5000
  ```
  *(Initial 5ms, min hold 100ms, max hold 5000ms)*

- **LSA Throttle**:
  ```plaintext
  Router(config-router)# timers throttle lsa all 10 100 5000
  ```
  *(Controls how frequently LSAs can be reissued.)*

> 📌 **Exam Tip**: Adjusting these timers is crucial in large, dynamic networks to prevent CPU spikes.

### 9.2 Fast Hello and BFD
- **Fast Hello**:
  ```plaintext
  Router(config-if)# ip ospf hello-interval 1
  Router(config-if)# ip ospf dead-interval 3
  ```
  *(Detects link failures in ~3 seconds)*

- **Bidirectional Forwarding Detection (BFD)**:
  ```plaintext
  Router(config-if)# ip ospf bfd
  ```
  *(Sub-second link failure detection for critical links.)*

> ✅ Minimizes downtime in event of interface or neighbor failure.

---

## 10. Troubleshooting OSPF

### 10.1 Common Issues
- **Neighbors stuck in EXSTART/EXCHANGE**: Check **MTU**  
- **Stuck in INIT**: Possibly the neighbor’s Hello is not seen (area mismatch or no “network” statement).  
- **Missing External Routes**: Type 5 LSAs blocked by a stub area or incomplete redistribution config.

### 10.2 Verification Commands
- `show ip ospf neighbor` – Adjacency states (FULL, 2WAY, etc.).  
- `show ip ospf interface` – Hello/Dead timers, priority, cost, area.  
- `show ip ospf database` – Inspect LSA types and sequence numbers.  
- `show ip route ospf` – Confirm routes are installed.

> 📌 **Exam Tip**: Check for **area mismatches**, **authentication errors**, or **timer differences** if adjacency fails to form.

---

## 🔥 Exam Alert: Key Focus Areas
1. **OSPF LSA Types** – Distinguish Type 1, 2, 3, 5, and 7 for area operations.  
2. **Area Design** – Backbone, Stub, NSSA for controlling LSA flooding.  
3. **OSPFv2 vs. OSPFv3** – Interface-based config in OSPFv3, IPsec authentication, dual-stack.  
4. **DR/BDR Elections** – Priority manipulations for stable multi-access networks.  
5. **Summarization & Filtering** – Minimizes overhead and protects from routing loops.  
6. **Authentication** – MD5 (OSPFv2) vs. IPsec (OSPFv3).  
7. **Performance Tuning** – Throttle SPF and LSAs, use BFD for sub-second detection.  
8. **MPLS Integration** – PE-CE OSPF, sham-links, route preference.


