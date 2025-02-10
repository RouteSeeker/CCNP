# Day 11: SD-Access, SD-WAN, and QoS – Modern Network Transformation

---

## WHY SD-ACCESS, SD-WAN, AND QoS MATTER

In today’s **software-defined era**, networks must be **agile**, **policy-driven**, and **optimized** for ever-changing application requirements. **SD-Access**, **SD-WAN**, and **QoS** collectively empower network engineers to build modern infrastructures that are simpler to manage, more secure, and able to guarantee performance for critical applications.

1. **SD-Access** leverages a fabric-based campus design, orchestrated by **Cisco DNA Center**, for streamlined policy enforcement and segmentation.
2. **SD-WAN** redefines wide-area connectivity by centralizing management and enabling **app-aware routing** across multiple links (MPLS, broadband, LTE).
3. **QoS** ensures real-time applications like voice and video maintain low latency and jitter, while bulk data traffic is efficiently queued or shaped.

By mastering **SD-Access**, **SD-WAN**, and **QoS** concepts, you can architect an **intelligent, policy-driven** network that adapts quickly to business needs and optimizes application performance.

---

## OBJECTIVES

By the end of **Day 11**, you will be able to:

1. **Explain SD-Access** – Understand the **fabric concept**, how **Cisco DNA Center** orchestrates policy, and the roles of control/data/policy planes.
2. **Compare Traditional WAN to SD-WAN** – Highlight central controller benefits, policy-based routing, and how SD-WAN optimizes app performance.
3. **Define QoS Mechanisms** – Classification, marking, queuing, shaping, and their respective roles in traffic prioritization.
4. **Configure Basic Class Maps and Policy Maps** – Assign DSCP values, set bandwidth guarantees for critical traffic, and shape or police as needed.
5. **Verify QoS Operation** – Use show commands to confirm that traffic is classified, queued, and shaped as intended.

---

## 1. SD-ACCESS: FABRIC CONCEPTS AND CISCO DNA CENTER

### 1.1 Fabric-Based Architecture

**Software-Defined Access** creates a **campus fabric** that abstracts the physical network into an **overlay**. Key elements:

- **Underlay**: A routed layer providing IP connectivity among network devices.
- **Overlay**: Constructs virtual networks using VXLAN for segmentation, ensuring each department or group is isolated as needed.
- **Control Plane**: Often uses **LISP** to track endpoint IDs and map them to device locators.
- **Cisco DNA Center**: Central orchestrator for device configuration, policy deployment, and analytics.

**Benefits**:
- **User-based policy**: Identity Services Engine (ISE) can integrate with DNA Center to apply consistent policies based on user or device role.
- **Automated deployment**: Single pane of glass to push configurations to all fabric edge nodes.

> 🔥 **Exam Tip**: Remember that **DNA Center** is crucial for automating large-scale campus deployments, simplifying tasks such as VLAN provisioning, segmentation, and QoS.

---

## 2. CONTROL, DATA, AND POLICY PLANES

### 2.1 SD-Access Planes

In **SD-Access**, three distinct planes handle different responsibilities:

1. **Control Plane**: LISP handles endpoint registration and lookup, mapping user devices (EIDs) to fabric edge devices (RLOCs).
2. **Data Plane**: Implements VXLAN or other encapsulation for forwarding traffic within the fabric, ensuring each segment is isolated.
3. **Policy Plane**: Cisco DNA Center defines user or device group policies, which the fabric enforces automatically.

**Why Separate?**
- Easier to scale: Each plane focuses on specific tasks (mapping, forwarding, policy).
- Simplified changes: Modifying policy or data encapsulation doesn’t require redoing the entire network config.

---

## 3. SD-WAN BENEFITS: CENTRALIZED MANAGEMENT, POLICY-BASED ROUTING

### 3.1 Traditional WAN vs. SD-WAN

| **Traditional WAN**                 | **SD-WAN**                                                                   |
|------------------------------------|------------------------------------------------------------------------------|
| Box-by-box config, reliant on MPLS | Centralized controller with multi-transport support (MPLS, Internet, LTE)   |
| Rigid topologies                   | Dynamic, **application-aware** routing                                       |
| Limited visibility, manual updates | Real-time performance monitoring, automated policy updates                  |

**SD-WAN** architecture introduces a controller that **pushes routing and security policies** to branch routers, enabling **policy-based routing** and **app-aware path selection**.

### 3.2 Key SD-WAN Capabilities

- **Dynamic Path Selection**: Measures link performance (latency, jitter) to direct critical apps over the highest-quality link.
- **Centralized Analytics**: Gains real-time insights into branch link usage, application SLAs, and policy compliance.
- **Security Integration**: Basic firewalling, IPS, or web filtering can be built into the SD-WAN edge, applying consistent security policies.

> 🔥 **Exam Tip**: **SD-WAN** drastically reduces operational overhead by allowing templates or policy objects that define app-based routing or security rules once—then pushing them globally.

---

## 4. DEFINE QoS MECHANISMS: CLASSIFICATION, MARKING, QUEUING, AND SHAPING

### 4.1 QoS Basics

**Quality of Service** ensures critical or latency-sensitive traffic is prioritized during congestion. Four main functions:

1. **Classification**: Identifying traffic by DSCP, IP addresses, protocols, etc.
2. **Marking**: Setting DSCP or CoS bits to indicate priority.
3. **Queuing**: Placing traffic into priority queues or weighting queues to meet bandwidth needs.
4. **Shaping or Policing**: Controlling the outbound rate, shaping bursts or policing to drop/remark excess traffic.

**Exam Tip**: The correct placement of classification and marking steps is crucial. Typically, mark traffic as close to the source as possible to influence downstream devices’ queuing decisions.

### 4.2 QoS in a Converged Network

- **Voice**: Needs low latency/jitter—often assigned EF (Expedited Forwarding).
- **Video**: Slightly less strict but still requires low latency—assigned AF4 DSCP classes.
- **Data**: Typically best-effort unless it’s business-critical (e.g., AF2 or AF3).

---

## 5. CONFIGURE CLASS MAPS AND POLICY MAPS

### 5.1 MQC (Modular QoS CLI)

The MQC approach involves:

1. **Class Maps**: Identify traffic (by ACL, DSCP, protocol).
2. **Policy Maps**: Define actions (e.g., bandwidth, priority, shape, mark).
3. **Service Policy**: Applied to an interface (input or output).

**Example**:
```bash
class-map match-any CRITICAL-TRAFFIC
 match dscp ef
 match protocol citrix

policy-map WAN-OUT
 class CRITICAL-TRAFFIC
  priority 1000
 class class-default
  fair-queue

interface GigabitEthernet0/0
 service-policy output WAN-OUT
```

**Priority** ensures CRITICAL-TRAFFIC is dequeued first, guaranteeing minimal latency.

### 5.2 Shaping vs. Policing

- **Shaping**: Buffers or delays excess traffic to maintain a steady rate. Used typically on outbound WAN edges.
- **Policing**: Enforces an instantaneous rate limit, dropping packets or remarking DSCP if exceeded.

> 🔥 **Exam Tip**: Shaping is especially important on **slower WAN links** to avoid dropping real-time traffic at the provider edge.

---

## 6. VERIFY QoS BEHAVIOR WITH 'SHOW' COMMANDS

**Key Commands**:
- `show policy-map interface [interface]`: Displays real-time stats—packets matched, queue usage, and drops.
- `show class-map [name]`: Summaries of class definitions.
- `show policy-map [name]`: Details all classes and actions in a policy.

If counters are not incrementing, verify the classification logic (ACL, DSCP match, or NBAR protocol) and ensure the traffic is actually flowing.

**Exam Tip**: Generating test traffic is often needed to see counters move. Tools like IP SLA or an external traffic generator can help confirm QoS policies.

---

## 🔥 EXAM ALERT: KEY FOCUS AREAS

1. **SD-Access** – Fabric-based campus using **DNA Center** for policy. Understand LISP-based control plane.
2. **SD-WAN** – Compare old vs. new WAN; note the application-aware approach, centralized management, zero-touch deployments.
3. **QoS Mechanisms** – Classification/Marking at the edge, LLQ or priority for voice, shaping/policing for WAN.
4. **MQC** – Class maps, policy maps, and service policies—always confirm with `show policy-map interface`.
5. **Troubleshooting** – Missing adjacency in SD-Access? Check LISP. SD-WAN performance issue? Check policy-based routing. QoS mismatch? Confirm DSCP markings and queue stats.

