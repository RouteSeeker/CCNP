# Day 8: First-Hop Redundancy Protocols and Network Services

## Why FHRP and Core Services Matter
Building a **resilient, high-performing network** requires ensuring:

- **High Availability** at the **default gateway** level through **First-Hop Redundancy Protocols (FHRP)**.  
- **Reliable Host Configuration** with **DHCP** for automatic IP addresses, gateways, and DNS servers.  
- **Consistent Time Synchronization** using **NTP**, vital for accurate logs, time-based security, and coordination across devices.  
- **Efficient Address Usage** with **NAT** for IPv4 conservation and secure bridging between private and public networks.

All these elements—**FHRP, DHCP, NTP, and NAT**—combine to provide **continuity**, **automation**, and **security** in modern enterprise networks.

---

## Objectives
By the end of **Day 8**, you will be able to:

1. **Implement FHRP** – Compare and deploy **HSRP, VRRP, and GLBP** for gateway redundancy and load balancing.  
2. **Ensure Seamless Failover** – Understand priority/preemption in HSRP/VRRP, test transitions, and verify active/standby (or master/backup) roles.  
3. **Use GLBP for Load Balancing** – Distribute gateway responsibilities across multiple routers.  
4. **Configure and Verify DHCP** – Automate IP address assignment, exclude addresses, and confirm client bindings.  
5. **Synchronize Devices with NTP** – Achieve consistent time across routers, switches, and servers.  
6. **Understand NAT Types** – Distinguish static NAT, dynamic NAT, and PAT for address translation.  
7. **Monitor NAT** – Use `show ip nat translations` to verify real-time address mapping.

---

## 1. First-Hop Redundancy Protocols (FHRP)

### 1.1 Why FHRP?
End devices typically have **one default gateway**. If that router’s interface fails, hosts lose connectivity. **FHRP** prevents this by creating a **virtual IP** shared by multiple routers:

- **HSRP (Hot Standby Router Protocol)** – Cisco-proprietary Active/Standby mechanism.  
- **VRRP (Virtual Router Redundancy Protocol)** – Open-standard equivalent to HSRP.  
- **GLBP (Gateway Load Balancing Protocol)** – Cisco-proprietary solution that supports **active-active** gateway distribution.

#### 1.1.1 Key Benefits
- **Minimal Disruption**: Clients keep the same default gateway IP.  
- **Automated Failover**: Standby or Backup router becomes active if the primary fails.  
- **Traffic Optimization**: GLBP’s load balancing can spread traffic across multiple routers.

---

### 1.2 HSRP: Hot Standby Router Protocol

1. **Active/Standby**: One router actively forwards traffic (Active), another waits (Standby).  
2. **Virtual IP/MAC**: Hosts point to a **virtual IP** as the gateway; the active router responds using a **virtual MAC**.  
3. **HSRP States**:
   - **Initial**: Interface is down or just starting.  
   - **Learn**, **Listen**, **Speak**: Progression phases for discovering the active router.  
   - **Standby**: Next in line if the active fails.  
   - **Active**: Currently forwarding packets.

**HSRP Communication** uses **multicast (224.0.0.2)** over **UDP port 1985**.

#### 1.2.1 HSRP Configuration Example
```plaintext
! Router A
interface GigabitEthernet0/1
 standby 1 ip 192.168.10.254
 standby 1 priority 120
 standby 1 preempt

! Router B
interface GigabitEthernet0/1
 standby 1 ip 192.168.10.254
 standby 1 priority 100
 standby 1 preempt
```
- **Group 1** with virtual IP **192.168.10.254**.  
- **Priority**: Higher is preferred (120 > 100).  
- **Preempt**: If a router with higher priority recovers, it regains Active status.

##### Verification
```plaintext
Router# show standby
```
- Shows group number, state (Active/Standby), priority, virtual IP, and timers.

> 📌 **Exam Tip**: Without **preempt**, the first router to claim Active remains so even if a higher-priority router comes online later.

---

### 1.3 VRRP: Virtual Router Redundancy Protocol
- **Open-standard** solution, Master/Backup model.
- Routers share a **virtual IP**; the **Master** routes packets while **Backup** stands by.
- Uses **multicast address 224.0.0.18** with IP protocol 112.

#### 1.3.1 VRRP Configuration
```plaintext
interface GigabitEthernet0/1
 vrrp 5 ip 192.168.20.254
 vrrp 5 priority 110
 vrrp 5 preempt
```
- VRRP group **5**, virtual IP **192.168.20.254**.
- Priority determines the **Master** if multiple routers claim the same group.

##### Verification
```plaintext
Router# show vrrp
```
- Displays group ID, Master router IP, priority, and state.

---

### 1.4 GLBP: Gateway Load Balancing Protocol
- **Cisco-proprietary** approach allowing **active-active** usage of multiple routers.
- **AVG (Active Virtual Gateway)** coordinates multiple AVFs (Active Virtual Forwarders).
- **Load Balancing**: Clients get different virtual MAC addresses, distributing inbound traffic across multiple routers.

#### 1.4.1 GLBP Configuration
```plaintext
interface GigabitEthernet0/1
 glbp 10 ip 192.168.30.254
 glbp 10 priority 120
 glbp 10 load-balancing round-robin
```
**Load Balancing Modes**:
- `round-robin` – Evenly distributes requests among forwarders.
- `weighted` – Uses a weighted approach based on router capacity or link speed.
- `host-dependent` – Assigns a single AVF for each client indefinitely.

##### Verification
```plaintext
Router# show glbp
```
- Shows the **AVG** and each **AVF** with respective virtual MACs and forwarding roles.

> 📌 **Exam Tip**: **GLBP** is unique among these protocols for providing true load balancing by default.

---

## 2. DHCP (Dynamic Host Configuration Protocol)

### 2.1 DHCP Operation (DORA)
1. **Discover** – Client broadcasts to find a DHCP server.  
2. **Offer** – Server responds with an IP lease, gateway, DNS, etc.  
3. **Request** – Client requests offered parameters from the server.  
4. **Acknowledge** – Server finalizes the lease assignment.

### 2.2 DHCP Server on a Cisco Router

#### 2.2.1 Configuration
```plaintext
Router(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.10
Router(config)# ip dhcp pool VLAN10
Router(dhcp-config)# network 192.168.10.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.10.254
Router(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
Router(dhcp-config)# domain-name example.local
```
- **excluded-address** – Reserve IPs for static devices.  
- `default-router` – Must match the **FHRP virtual IP** for redundancy.

##### Verification
```plaintext
Router# show ip dhcp binding
Router# show ip dhcp pool
Router# show ip dhcp conflict
```
- Displays current leases, pool usage, any IP conflicts.

> ✅ **Key**: DHCP’s default gateway must match **HSRP/VRRP/GLBP** virtual IP.

---

## 3. NTP (Network Time Protocol)

### 3.1 Importance of Time Synchronization
- **Accurate Logs**: Timestamps across routers/switches are aligned for streamlined troubleshooting.
- **Certificate Validations**: Many security protocols depend on correct clock settings.
- **Time-Based ACLs**: Access rules that apply during specific times need correct system clocks.

### 3.2 NTP Hierarchy
- **Stratum** levels measure distance from an authoritative time source.
  - Stratum 1: Directly connected to a high-precision clock (e.g., GPS).
  - Stratum 2+: Sync from a lower stratum server.

### 3.3 Basic NTP Configuration
```plaintext
Router(config)# ntp server 192.168.2.10
Router(config)# ntp update-calendar
Router(config)# clock timezone PST -8
```
- `ntp update-calendar`: Updates hardware clock based on NTP time.
- `ntp server <ip>`: Points to a more authoritative server.

##### Verification
```plaintext
Router# show ntp associations
Router# show ntp status
```
- Confirms synchronization, stratum, and reference server.

> 📌 **Exam Tip**: If time isn’t syncing, check reachability, ACLs, or if an authentication key is required.

---

## 4. NAT (Network Address Translation)

### 4.1 Rationale
- **Conserves IPv4 addresses** by allowing multiple internal hosts to share fewer public IPs.
- **Enhances security** by hiding internal IP schemes from external hosts.

### 4.2 NAT Types

1. **Static NAT**: One private IP ↔ One public IP (1:1).  
   - Common for servers needing a consistent public identity.

2. **Dynamic NAT**: Pool of public IPs assigned to hosts on a first-come, first-served basis.  
   - Limited by the pool size.

3. **PAT (Port Address Translation)** or NAT Overload: Many private IPs share a single (or few) public IPs, differentiating by port numbers.  
   - Widely used in home routers for internet access.

---

### 4.3 NAT Configuration

#### 4.3.1 Interface Role
```plaintext
interface GigabitEthernet0/1
 ip nat inside

interface GigabitEthernet0/0
 ip nat outside
```
- **Inside** = LAN facing; **Outside** = WAN facing.

#### 4.3.2 Static NAT Example
```plaintext
ip nat inside source static 192.168.10.100 203.0.113.25
```
- Consistent public identity for a specific internal host.

#### 4.3.3 Dynamic NAT
```plaintext
ip nat pool MYPOOL 203.0.113.30 203.0.113.40 netmask 255.255.255.224
access-list 10 permit 192.168.10.0 0.0.0.255
ip nat inside source list 10 pool MYPOOL
```
- Pool from .30 to .40, assigned dynamically to internal hosts.

#### 4.3.4 PAT (Overload)
```plaintext
access-list 10 permit 192.168.10.0 0.0.0.255
ip nat inside source list 10 interface GigabitEthernet0/0 overload
```
- Many:1 mapping using the interface’s public IP.

---

### 4.4 NAT Verification
```plaintext
Router# show ip nat translations
Router# show ip nat statistics
```
- **show ip nat translations**: Lists active sessions with inside local/global and outside local/global addresses.
- **show ip nat statistics**: Summaries of NAT usage (hits/misses).

> 📌 **Exam Tip**: “Inside local” = real private IP; “Inside global” = NATed public IP.

---

## 5. Verifying Failover & Network Services

### 5.1 FHRP Verification
- **HSRP**: `show standby`  
- **VRRP**: `show vrrp`  
- **GLBP**: `show glbp`

Simulate failover (e.g., shutting down Active router’s interface) and see if Standby/Master/Backup transitions with minimal downtime.

### 5.2 DHCP Client Testing
- Use `ipconfig /release` and `ipconfig /renew` (Windows) or similar commands on other OSes.  
- Validate the client receives the correct IP, subnet mask, default gateway (the FHRP virtual IP), and DNS.

### 5.3 NTP Sync
- `show ntp status`, `show ntp associations` confirm synchronization state and stratum.

### 5.4 NAT Connectivity
- `show ip nat translations` to confirm private IP to public IP mappings while traffic flows.  
- Attempt external pings or HTTP requests from internal hosts to see NAT translations appear.

---

## 🔥 Exam Alert: Key Focus Areas
1. **FHRP Protocols** (HSRP, VRRP, GLBP) – Know configuration steps, states (Active/Standby, Master/Backup), load balancing (GLBP).  
2. **DHCP** – DORA process, server config (pools, default-router), verification commands.  
3. **NTP** – Hierarchy (stratum), `show ntp associations`, time accuracy for logs/security.  
4. **NAT Types** – Static, dynamic, PAT/overload, inside/outside definitions.  
5. **Verification** – `show standby/vrrp/glbp`, `show ip dhcp binding`, `show ntp status`, `show ip nat translations`.  
6. **Failover Testing** – Shutting down primary router interfaces to validate rapid standby transitions.

