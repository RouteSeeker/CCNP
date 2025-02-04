# Day 5: EIGRP (Enhanced Interior Gateway Routing Protocol) - The Ultimate Guide

---

## Why EIGRP Matters
A well-implemented **EIGRP environment** ensures:
- ✅ **Fast Convergence** – Uses the **DUAL algorithm** to precompute backup paths and instantly failover in case of link failure.  
- ✅ **Loop-Free Routing** – Prevents routing loops by maintaining **Successor** and **Feasible Successor** paths.  
- ✅ **Scalable and Efficient** – Supports **VLSM**, **CIDR**, and **route summarization**, making it ideal for large enterprises.  
- ✅ **Low Overhead** – Uses **multicast (224.0.0.10)** and sends **incremental updates**, minimizing bandwidth usage.  
- ✅ **Flexible Load Balancing** – Allows **equal-cost** and **unequal-cost** load balancing (via `variance`).  
- ✅ **Robust Security** – EIGRP authentication (MD5 or HMAC-SHA) prevents unauthorized routers from forming adjacencies.  
- ✅ **IPv4 & IPv6 Support** – EIGRP can be deployed in both IPv4 and IPv6 networks using **EIGRP Named Mode**.

---

## Objectives
By the end of this guide, you will:
- ✅ Understand **how EIGRP works**, including the **DUAL algorithm** and how it calculates **best paths**.  
- ✅ Be able to **configure EIGRP** for **basic and advanced** deployments (including **stub routing**, **named mode**, and **IPv6**).  
- ✅ Learn **route summarization** and how it helps reduce **routing overhead** in large-scale environments.  
- ✅ Implement **EIGRP authentication** for **secure routing** in an enterprise network.  
- ✅ Master **EIGRP troubleshooting**, using **show commands** and understanding **stuck-in-active** routes.  
- ✅ Know **EIGRP best practices** and the most **important topics for exams**.

---

## 1. Understanding EIGRP

### What is EIGRP?
**EIGRP (Enhanced Interior Gateway Routing Protocol)** is a **Cisco-proprietary advanced distance-vector routing protocol**. Unlike traditional distance-vector protocols (like RIP), EIGRP has features that allow it to behave like a **hybrid** protocol—incorporating both **distance-vector** and **link-state** principles.

### How EIGRP Works
1. **Neighbor Discovery and Adjacency Formation**  
   - EIGRP uses **Hello packets** (multicast to **224.0.0.10**) to **discover neighbors** and form adjacencies.  
   - Neighbors exchange **EIGRP Update packets**, containing **routing information**.  
   - Once formed, neighbors continuously **exchange Hello packets** to ensure connectivity.

2. **Topology Table and Route Selection**  
   - EIGRP **stores multiple paths** in its **Topology Table**, but only installs the **best path** in the **Routing Table**.  
   - It also maintains **backup paths** (Feasible Successors) for quick failover.

3. **DUAL Algorithm (Diffusing Update Algorithm)**  
   - **DUAL** guarantees a **loop-free topology**.  
   - It precomputes **Feasible Successors** so that if the primary route **fails**, failover occurs **instantly**.  
   - If a **backup path is unavailable**, the route enters an **Active State**, meaning EIGRP is actively searching for a new route.

> **📌 Exam Tip**: A route **stuck in Active State** is a sign of network **instability** or a **configuration issue**.

---

## 2. Key Features of EIGRP

### 2.1 Distance-Vector with Link-State Characteristics
- **Distance-Vector Features**:  
  - Uses **hop-by-hop** routing.  
  - Relies on neighbor updates for network topology.

- **Link-State Features**:  
  - Supports **rapid convergence** through **DUAL**.  
  - Builds a **topology table** to store multiple routes.

### 2.2 Successor vs. Feasible Successor
- **Successor**:  
  - The **best route** to a destination.  
  - **Stored in the Routing Table** (`show ip route`).

- **Feasible Successor**:  
  - A **precomputed backup path**.  
  - **Stored in the Topology Table** (`show ip eigrp topology`).  
  - Becomes the **primary route instantly** if the Successor fails.

### 2.3 Feasibility Condition (FC)
For a **Feasible Successor** to be valid, it must meet the **Feasibility Condition**:

```
RD(candidate) < FD(current successor)
```

Where:
- **RD (Reported Distance)** → Metric as advertised by a neighbor.  
- **FD (Feasible Distance)** → The metric **of the current best path**.

> **Example**: If the **best path** has an FD of **50,000**, a **backup path** with an RD of **40,000** qualifies as a **Feasible Successor**.

> **📌 Exam Tip**: If **no Feasible Successors exist**, EIGRP **enters Active State** and queries its neighbors.

---

## 3. Configuring EIGRP

### 3.1 Enabling Basic EIGRP
```plaintext
Router(config)# router eigrp 100
Router(config-router)# network 192.168.1.0 0.0.0.255
Router(config-router)# network 10.10.10.0 0.0.0.255
```
- The **100** is the **Autonomous System (AS) number** and must match between neighbors.

### 3.2 Enabling Passive Interfaces
```plaintext
Router(config-router)# passive-interface GigabitEthernet0/1
```
- **Stops** sending EIGRP **Hello packets** on that interface.

### 3.3 Verifying Configuration
```plaintext
Router# show ip eigrp neighbors
Router# show ip route eigrp
Router# show ip eigrp topology
```
- **Neighbors** → Checks adjacency.  
- **Route eigrp** → Lists routes labeled **"D"** for EIGRP.  
- **Topology** → Displays **Successors** and **Feasible Successors**.

> **📌 Exam Tip**: If neighbors **fail to form**, check **AS numbers**, **K-values**, and **network statements**.

---

## 4. EIGRP Metrics & Load Balancing

### 4.1 Metric Formula

```
EIGRP Metric = ((10^7 / Min Bandwidth) + (Sum of Delays / 10)) * 256
```

Where:
- **Bandwidth** → Uses the **lowest** link bandwidth.  
- **Delay** → The **sum of delays** along the path.

### 4.2 Load Balancing

#### Equal-Cost Load Balancing (Default)
By default, EIGRP will **split traffic equally** across multiple paths of the **same metric**.

#### Unequal-Cost Load Balancing (`variance`)
Allows EIGRP to **use additional paths** that have a **higher metric**, up to a defined threshold.
```plaintext
Router(config-router)# variance 2
```
> **Example**: If the **best path** has a metric of **10,000**, then with `variance 2`, EIGRP will use **any route with a metric ≤ 20,000**.

---

## 5. Advanced EIGRP Features

### 5.1 Summarization
Reduces routing table size:
```plaintext
Router(config-if)# ip summary-address eigrp 100 10.1.0.0 255.255.0.0
```

### 5.2 Stub Routing
Limits unnecessary updates on **branch routers**:
```plaintext
Router(config-router)# eigrp stub connected summary
```
> **📌 Use Case**: Ideal for **branch offices** to reduce **routing overhead**.

### 5.3 Authentication

1. **Configure a Key-Chain**:
   ```plaintext
   Router(config)# key chain EIGRP_KEYS
   Router(config-keychain)# key 1
   Router(config-keychain-key)# key-string SECRET
   ```

2. **Apply Authentication**:
   ```plaintext
   Router(config-if)# ip authentication mode eigrp 100 md5
   Router(config-if)# ip authentication key-chain eigrp 100 EIGRP_KEYS
   ```

---

## 🔥 Exam Alert: Key Takeaways

1. **Successor vs. Feasible Successor** – Understand DUAL’s **loop-free guarantees**.  
2. **Metric Calculation** – Know **Bandwidth & Delay** are the default values used.  
3. **Variance** – Enables **unequal-cost load balancing**.  
4. **Stub Routing** – Essential for **branch deployments**.  
5. **Summarization** – Reduces routing overhead.  
6. **Named Mode** – The **modern** way to configure EIGRP.  
7. **Redistribution** – Use **subnets** when redistributing EIGRP into OSPF.  
8. **Troubleshooting** – Use `show ip eigrp topology` and `show ip eigrp neighbors` to diagnose issues.

---

## Final Summary
**EIGRP** is one of the **fastest-converging**, **scalable**, and **most efficient** routing protocols. Understanding **DUAL**, **Successor & Feasible Successor selection**, and **metric calculations** will help you **deploy and troubleshoot** EIGRP **in real-world enterprise networks**.
```
