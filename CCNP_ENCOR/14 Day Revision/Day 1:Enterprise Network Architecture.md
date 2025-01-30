# Day 1: Enterprise Network Architecture

## Why Enterprise Network Architecture Matters
A well-designed enterprise network ensures:
- Efficient traffic flow for applications, VoIP, video conferencing.
- High availability with redundancy and failover mechanisms.
- Scalability to accommodate business growth and new technologies.
- Security to protect sensitive data from cyber threats.
- Manageability through automation and centralized control.

---

## 1. Campus Network (Local Network Infrastructure)
The campus network is where employees, devices, and applications interact. It consists of:
- Switches at the access and distribution layers.
- Routers for Layer 3 connectivity and WAN access.
- Wireless LAN Controllers (WLCs) to manage wireless access points (APs).
- Security mechanisms like firewalls and access control lists (ACLs).

### Campus Network Design
- **Layer 2 Switching**: VLANs segment traffic logically.
- **Layer 3 Routing**: OSPF/EIGRP provide efficient routing.
- **Wireless Integration**: APs provide Wi-Fi connectivity.
- **Access Control**: 802.1X, MAC filtering, and dynamic VLAN assignment secure endpoints.

> 📌 **Exam Tip**: Understand how VLANs, trunking, and inter-VLAN routing work within the campus.

---

## 2. Branch Network (Remote Office Connectivity)
The branch network connects remote sites to corporate resources. It requires:
- Secure WAN connectivity (MPLS, SD-WAN, VPN).
- Reliable access to corporate applications (via cloud or data centers).
- Consistent security policies across locations.

### Branch Connectivity Options

| Technology | Pros                           | Cons                            |
|------------|--------------------------------|---------------------------------|
| MPLS       | Reliable, low latency         | Expensive                      |
| VPN        | Secure, cost-effective        | Performance depends on the internet |
| SD-WAN     | Intelligent routing, optimized traffic | Requires SD-WAN appliances |

> 📌 **Exam Tip**: Be familiar with MPLS vs. VPN vs. SD-WAN and their use cases.

---

## 3. Wide Area Network (WAN)
The WAN connects campus networks, branches, and data centers across large distances. A properly designed WAN:
- Reduces latency for critical applications.
- Ensures redundancy with dual links or failover strategies.
- Supports QoS policies to prioritize VoIP/video traffic.

### WAN Transport Technologies

| Technology  | Use Case |
|------------|----------|
| MPLS       | High-performance, private WAN transport |
| SD-WAN     | Cost-effective, intelligent WAN routing |
| IPSec VPN  | Secure site-to-site connectivity over the internet |
| 5G/LTE     | Backup WAN connectivity |

> 📌 **Exam Tip**: Expect questions comparing SD-WAN and MPLS performance, security, and cost.

---

## 4. Data Center (Centralized Application Hosting)
Data centers host enterprise applications, databases, and storage. Modern data centers use:
- Virtualization to improve resource utilization.
- Spine-Leaf architectures for high-speed connectivity.
- Cloud integration for scalability.

### Key Data Center Technologies

| Technology               | Purpose |
|--------------------------|---------|
| Spine-Leaf Architecture  | Improves data center scalability and speed |
| Virtual Machines (VMs)   | Run multiple applications on one server |
| Containers (Docker, Kubernetes) | Deploy applications in lightweight environments |
| Firewalls, IPS/IDS       | Protect against cyber threats |

> 📌 **Exam Tip**: Be ready to identify components of a modern data center, including cloud integration.

---

## Network Design Models

### 1. Hierarchical Network Design
The three-layer model provides:
1. **Core Layer** – High-speed backbone, no packet filtering.
2. **Distribution Layer** – Routing, security policies, and VLAN segmentation.
3. **Access Layer** – End-user connections, port security, and PoE.

#### Collapsed Core Design (Two-Tier)
- Used in small to medium networks.
- Combines core and distribution layers.
- Reduces cost but limits scalability.

> 📌 **Exam Tip**: Understand when to use a two-tier vs. three-tier design.

---

### 2. Software-Defined Networking (SDN)
Traditional networks use hardware-based forwarding, while SDN centralizes control via software.

| Feature       | Traditional Networks | SDN |
|--------------|----------------------|-----|
| Control Plane | Distributed          | Centralized |
| Management   | CLI-based             | Automated, policy-driven |
| Example      | OSPF/EIGRP Routing    | Cisco DNA Center |

> 📌 **Exam Tip**: Expect questions on how SDN differs from traditional networking.

---

## Cisco Controllers for Automation

### 1. Cisco DNA Center (Intent-Based Networking)
- Automates network configuration.
- Enhances security with AI-driven threat detection.
- Monitors network performance and provides insights.

### 2. APIC-EM (Older SDN Controller)
- Provided policy-based automation.
- Replaced by Cisco DNA Center.

> 📌 **Exam Tip**: Understand why Cisco DNA Center is the preferred SDN controller today.

---

## Enterprise Network Security Considerations
Security is critical in enterprise network design. Common security measures include:
- **Firewalls** – Filter inbound and outbound traffic.
- **IPS/IDS** – Detect and prevent network threats.
- **Zero Trust Security** – Restrict access based on identity and device posture.
- **802.1X Authentication** – Ensures secure user and device connections.

> 📌 **Exam Tip**: Expect security-related questions, including 802.1X, ACLs, and segmentation.

---

## Key Enterprise Networking Protocols

| Protocol  | Purpose |
|-----------|---------|
| OSPF      | Link-state routing for enterprise networks |
| EIGRP     | Cisco proprietary, fast convergence |
| BGP       | WAN and internet routing |
| VRRP/HSRP | Redundant default gateway protocol |
| VXLAN     | Data center network virtualization |

> 📌 **Exam Tip**: Understand when to use OSPF, EIGRP, and BGP in enterprise networks.

---

## 🔥 Exam Alert: Key Focus Areas for ENCOR 350-401
- Understand campus, branch, WAN, and data center network components.
- Compare three-tier vs. two-tier network design models.
- Know SDN vs. traditional networking.
- Understand MPLS, SD-WAN, and VPN technologies.
- Recognize key security features like firewalls, IPS/IDS, and zero trust.
- Be familiar with Cisco DNA Center and APIC-EM.
