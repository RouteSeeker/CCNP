# Day 4: Spanning Tree Protocol (STP) and Port Aggregation

## Objectives:
- Understand the purpose of **Spanning Tree Protocol (STP)** in preventing loops.  
- Explore **STP variations**: PVST+, RPVST+, and MST.  
- Configure **root bridge priority** in STP.  
- Learn **STP verification commands**.  
- Define **EtherChannel** and its benefits.  
- Understand **EtherChannel protocols**: PAgP and LACP.  
- Configure and verify **EtherChannel**.  
- Troubleshoot **aggregation issues** effectively.  
- Deep dive into **STP topology changes**, convergence time, and BPDU details.  
- Explore **load balancing techniques** within EtherChannel.  
- Explain **STP protection mechanisms**, including **BPDU Guard, Root Guard, and Loop Guard**.  
- Compare **STP** with modern alternatives like **Shortest Path Bridging (SPB)** and **TRILL**.

---

## 1. Understanding Spanning Tree Protocol (STP)

### What is STP and Why is it Needed?
The **Spanning Tree Protocol (STP)** is a **Layer 2 protocol** designed to **prevent loop formation** in Ethernet networks with redundant links. Without STP, redundant connections in a switch topology would lead to:
- **Broadcast storms**  
- **MAC table instability**  
- **Multiple frame copies**, which can cripple network performance  

### How STP Works
1. **Bridge ID (BID)**: Each switch has a **unique Bridge ID**, consisting of:
   - **Priority value** (default: **32768**)  
   - **MAC address**  
2. **Root Bridge Election**: The switch with the **lowest BID** becomes the **Root Bridge** and serves as the reference point for all path calculations.  
3. **Path Cost Calculation**: STP assigns path costs based on interface speed:
   - **10 Mbps** = **100**  
   - **100 Mbps** = **19**  
   - **1 Gbps** = **4**  
   - **10 Gbps** = **2**  
4. **Port States**:
   - **Blocking**: Prevents loops, does not forward traffic.  
   - **Listening**: Listens for **BPDUs** to determine topology.  
   - **Learning**: Learns MAC addresses but does not forward frames.  
   - **Forwarding**: Actively forwards frames.  
   - **Disabled**: Port is administratively shut down.  

### BPDU (Bridge Protocol Data Unit) and STP Convergence
- **BPDU frames** are exchanged between switches to determine the network topology.  
- **Convergence Time**: Standard STP takes **30–50 seconds** to stabilize.
  - **Rapid Spanning Tree (RSTP)** speeds up convergence significantly.  

### STP Configuration Example
```plaintext
Switch(config)# spanning-tree vlan 10 priority 8192
Switch(config)# spanning-tree mode pvst
```

### STP Verification Commands
```plaintext
Switch# show spanning-tree vlan 10
Switch# show spanning-tree root
Switch# show spanning-tree interface GigabitEthernet 0/1
```
## 2. STP Variants: PVST+, RPVST+, and MST

1. **Per-VLAN Spanning Tree Plus (PVST+)**  
   - **Cisco proprietary** enhancement of STP.  
   - Runs **a separate STP instance per VLAN**, allowing **VLAN-specific load balancing**.  
   - Can be **resource-intensive** in large networks.  

2. **Rapid PVST+ (RPVST+)**  
   - Cisco proprietary enhancement of **RSTP**.  
   - Offers **faster convergence** than PVST+.  
   - Each **VLAN** runs its **own independent RSTP instance**.  

3. **Multiple Spanning Tree (MST)**  
   - IEEE Standard **(802.1s)** for **STP scalability**.  
   - Groups multiple VLANs under a **single STP instance**, reducing overhead.  
   - **More scalable** than PVST+ and RPVST+.  

### Configuring Different STP Modes
```plaintext
Switch(config)# spanning-tree mode pvst
Switch(config)# spanning-tree mode rapid-pvst
Switch(config)# spanning-tree mode mst
```

---

## 3. Root Bridge Priority Configuration

By default, STP elects the **Root Bridge** based on the **lowest BID** (Bridge ID).  
- **BID = Priority (Default: 32768) + MAC Address**

### Configuring a Root Bridge Priority
```plaintext
Switch(config)# spanning-tree vlan 10 priority 4096
```

### Verifying Root Bridge Role
```plaintext
Switch# show spanning-tree root
```

---

## 4. STP Protection Mechanisms

### BPDU Guard
- Prevents **unauthorized switches** from participating in STP by **shutting down a port** if it receives a BPDU.  

**Configuration:**
```plaintext
Switch(config)# interface GigabitEthernet 0/5
Switch(config-if)# spanning-tree bpduguard enable
```
**Verification:**
```plaintext
Switch# show spanning-tree interface GigabitEthernet 0/5 detail
```

### Root Guard
- Prevents a switch **from becoming the Root Bridge** if it receives superior BPDUs.
```plaintext
Switch(config-if)# spanning-tree guard root
```

### Loop Guard
- **Detects unidirectional link failures** that could cause loops.
```plaintext
Switch(config)# spanning-tree loopguard default
```

---

## 5. Introduction to EtherChannel

### What is EtherChannel?
- EtherChannel **combines multiple physical links** into a **single logical link**.  
- Provides **increased bandwidth** and **redundancy**.  

### EtherChannel Protocols
| **Protocol** | **Description**                                                 |
|-------------|-----------------------------------------------------------------|
| **PAgP**    | Cisco proprietary protocol that dynamically negotiates EtherChannel. |
| **LACP**    | IEEE standard (**802.3ad**) supporting multi-vendor environments.    |

### Configuring EtherChannel with LACP
```plaintext
Switch(config)# interface range GigabitEthernet 0/3 - 4
Switch(config-if-range)# channel-group 2 mode active
Switch(config-if-range)# exit
Switch(config)# interface port-channel 2
Switch(config-if)# switchport mode trunk
```

### Verifying EtherChannel Configuration
```plaintext
Switch# show etherchannel summary
Switch# show interfaces port-channel 2
```

---

## 6. Troubleshooting Aggregation Issues

### Common Issues & Solutions
| **Issue**                   | **Solution**                                                              |
|-----------------------------|---------------------------------------------------------------------------|
| Mismatched Configuration    | Ensure both sides use the **same mode** (PAgP or LACP).                   |
| Speed/Duplex Mismatch       | All bundled interfaces must have **identical speed and duplex settings**. |
| Incorrect Trunking          | Ensure EtherChannel interfaces are **configured correctly** for access or trunk mode. |

### Troubleshooting Commands
```plaintext
Switch# show etherchannel summary
Switch# show interfaces trunk
Switch# show spanning-tree interface port-channel 2
```

## 🔥 Exam Alert: Key Takeaways

1. **STP prevents loops** by blocking redundant paths.  
2. **Root Bridge selection** is based on **priority** (default: **32768**, lower is preferred).  
3. **RPVST+ converges faster** than PVST+.  
4. **MST reduces CPU load** by **grouping VLANs** under a **single STP instance**.  
5. **EtherChannel increases bandwidth** and prevents STP **blocking links**.  
6. **BPDU Guard and Root Guard** prevent **STP-based attacks**.  
7. Use **show spanning-tree** and **show etherchannel summary** for troubleshooting.  
