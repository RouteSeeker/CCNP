# Day 2: Packet Switching and Forwarding

## Objectives:
- Understand how packet switching works and its importance in modern networks.
- Learn the three main switching mechanisms: Process Switching, Fast Switching, and Cisco Express Forwarding (CEF).
- Explore the Forwarding Information Base (FIB) and Adjacency Table and their roles in routing and switching.
- Understand hardware acceleration in routers and switches, including ASICs, NPUs, and TCAM.
- Compare Layer 2 vs. Layer 3 forwarding and their use cases.
- Learn how to troubleshoot packet forwarding issues using Cisco commands.

---

## 1. Introduction to Packet Switching

### What is Packet Switching?
Packet switching is a networking technique used to efficiently transmit data across a network by breaking it into smaller packets that are sent independently to the destination. The receiving device reassembles these packets in the correct order.

### Why Packet Switching?
- **Efficient Use of Bandwidth** – Packets take different routes based on network availability.
- **Scalability** – Supports thousands or millions of simultaneous connections.
- **Fault Tolerance** – If a path fails, packets automatically reroute through alternate paths.
- **Optimized for Large Networks** – Allows dynamic load balancing and congestion management.

### Packet Switching vs. Circuit Switching
Unlike circuit switching, where a dedicated path is established between sender and receiver (such as in traditional telephone networks), packet switching allows multiple users to share network resources dynamically, reducing delays and optimizing bandwidth usage.

#### Examples:
- **Circuit Switching:** Traditional PSTN telephony requires a dedicated connection for each call.
- **Packet Switching:** Modern IP networks, VoIP, and the Internet use shared bandwidth, allowing multiple data streams to coexist.

---

## 2. Packet Switching Mechanisms in Cisco Devices

Cisco routers and switches use three main methods to forward packets.

### a) Process Switching (Slowest)
- Also known as **software switching**.
- The **CPU examines every incoming packet** and determines its destination using the routing table.
- Each packet is processed **one at a time**, leading to **high latency and increased CPU usage**.
- Typically used for packets that require special handling, such as **packets containing IP options, ICMP redirects, and control traffic**.

✅ **Example:**  
When a router receives an **unfamiliar destination**, it must consult the routing table and process-switch the packet before sending it to the correct interface.

🔹 **Command to Identify Process Switching:**
```bash
show processes cpu sorted
```

## b) Fast Switching (Optimized Cache-Based Switching)
- Introduced to improve process switching by caching routes.
- The first packet is **process-switched**, and subsequent packets use the **fast-switching cache**.
- Reduces CPU involvement but still requires periodic cache updates.
- Does **not scale well** for large networks with frequent routing table changes.

✅ **Example:**
If multiple packets are sent to the same destination, only the **first one is process-switched**; the rest follow the **cached route**, significantly reducing CPU overhead.

🔹 **Command to Verify Fast Switching:**
```bash
show ip cache
```
 ## c) Cisco Express Forwarding (CEF) (Fastest & Most Scalable)

- The **default and most efficient switching method** in Cisco devices.
- Uses **Forwarding Information Base (FIB)** and **Adjacency Tables** for **fast lookup**.
- Reduces dependency on **route caching** and improves **overall network performance**.
- Works in **software (standard CEF)** or **hardware (distributed CEF - dCEF)**.
- Supports **high-speed routing decisions** without CPU intervention.

✅ **Example:**  
A router receiving **high volumes of traffic** will use **CEF** to quickly forward packets **without CPU intervention**, reducing **network latency** and improving **throughput**.

🔹 **Commands to Enable and Verify CEF:**
```bash
ip cef
show ip cef
show cef interface GigabitEthernet 0/1
```
## 3. Forwarding Information Base (FIB) and Adjacency Tables

### **Forwarding Information Base (FIB)**
- The **FIB is a table used in CEF** to store **next-hop IP addresses** for all known destinations.
- Built from the **Routing Information Base (RIB)**, which comes from the routing table.
- Provides **fast destination-based forwarding** without requiring **route cache lookups**.
- Enables **line-rate forwarding** with minimal CPU intervention.

🔹 **Command to View FIB:**
```bash
show ip cef
```
### Adjacency Table

- Stores **Layer 2 information (MAC addresses, outgoing interfaces)** needed to forward packets.
- Created from the **ARP cache** and is used to **prepend Layer 2 headers** to packets before forwarding.
- Improves efficiency by ensuring **next-hop Layer 2 addresses** are immediately available.

🔹 **Command to View Adjacency Table:**
```bash
show adjacency detail
```
## 4. Hardware Acceleration in Switches and Routers

### Application-Specific Integrated Circuits (ASICs)
- Cisco switches use **ASICs** for **line-rate switching with minimal CPU involvement**.
- ASICs allow **high-speed switching operations** at line rates (10G/40G/100G Ethernet).

✅ **Example:**  
A Cisco Nexus switch uses **ASICs for high-performance data center networking**, achieving speeds of **10G/40G/100G per port**.

---

### Network Processing Units (NPUs)
- **Programmable processors** optimized for **high-performance network processing**.
- Common in **high-speed routers and SDN architectures**.

✅ **Example:**  
Cisco Catalyst 9000 Series uses **NPUs for SDN**.

---

### Ternary Content Addressable Memory (TCAM)
- A **high-speed memory** used for **storing ACLs, QoS policies, and Layer 3 forwarding information**.
- Enables **one-step lookups** instead of multiple CPU operations.

✅ **Example:**  
A **Layer 3 switch** uses **TCAM** to store routing entries for **faster packet forwarding**.

---

## 5. Layer 2 vs. Layer 3 Forwarding

| Feature              | Layer 2 Forwarding (Switching)         | Layer 3 Forwarding (Routing)         |
|----------------------|-----------------------------------------|---------------------------------------|
| **Decision Criteria** | MAC Address (CAM Table)                | IP Address (FIB Table)               |
| **Devices Used**     | Switches                                | Routers, Layer 3 Switches            |
| **Works Within**     | Same VLAN                               | Different VLANs, Subnets             |
| **Example**         | A switch forwards based on MAC address | A router forwards packets between subnets |

✅ **Example:**
- A switch receives a **frame** and forwards it based on the **MAC address table**.
- A router forwards **packets** between two different **subnets** using **CEF**.

---

## 6. Troubleshooting Packet Forwarding Issues

🔹 **Check if CEF is Enabled:**
```bash
show ip cef
```
### 🔹 Verify Adjacency Table:
```bash
 show adjacency detail
```
### 🔹 Check Process Switching Usage:

```bash
show processes cpu sorted
```
## 🔥 Exam Alert: Key Focus Areas for CCNP ENCOR 350-401

- **Understand packet switching mechanisms** (Process Switching, Fast Switching, and CEF).  
- **Know the role** of the Forwarding Information Base (FIB) and Adjacency Tables.  
- **Understand hardware acceleration** in routers and switches (ASICs, NPUs, TCAM).  
- **Compare Layer 2 vs. Layer 3 forwarding** and their applications.  



