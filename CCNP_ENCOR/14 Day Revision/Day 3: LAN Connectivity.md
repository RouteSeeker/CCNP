# Day 3: LAN Connectivity

## Why LAN Connectivity (VLANs) Matters
A well-designed LAN with VLAN segmentation ensures:
- **Efficient Traffic Management**: Reduces broadcast domains, minimizing unnecessary traffic.
- **Security**: Isolates sensitive departments or devices, preventing unauthorized access.
- **Scalability**: Easily add or move devices without massive rewiring.
- **Flexibility**: Group users logically rather than physically.
- **Manageability**: Simplifies troubleshooting and network changes.

---

## Objectives
- Understand core **VLAN concepts** and their role in segmentation.
- Configure **VLANs and trunking (802.1Q)** on Cisco switches.
- Implement **inter-VLAN routing** using both Router-on-a-Stick and Layer 3 switches.
- Explore **VTP (VLAN Trunking Protocol)** for centralized VLAN management.
- Learn best practices for **VLAN deployment** in campus networks.
- Configure and understand **VLAN Access Control Lists (VACLs)**.
- Recognize **VLAN security threats** (VLAN hopping, double tagging, rogue DHCP) and mitigation strategies.
- Troubleshoot **VLAN-related issues** using CLI commands.

---

## 1. VLANs and Trunking
**Virtual Local Area Networks (VLANs)** logically segment a network into smaller broadcast domains, even across multiple switches. This improves performance, security, and manageability.

### Key VLAN Types
- **Default VLAN (VLAN 1)**: Avoid using for critical traffic or management.
- **Data VLANs**: Typical user traffic (e.g., VLAN 10 for HR, VLAN 20 for IT).
- **Native VLAN**: Carries untagged traffic on trunk ports (best practice: change from VLAN 1).
- **Management VLAN**: Separate VLAN for switch management (e.g., VLAN 99).
- **Voice VLAN**: Dedicated VLAN for VoIP traffic (e.g., VLAN 100).
- **Private VLANs (PVLANs)**: Restrict host-to-host communication within the same VLAN.
- **Dynamic VLANs**: Assigned via VLAN Management Policy Server (VMPS).

### Configuring VLANs
- **Creating VLANs:**
  ```plaintext
  Switch(config)# vlan 10
  Switch(config-vlan)# name HR
  Switch(config)# vlan 20
  Switch(config-vlan)# name IT
  ```
### Assigning VLANs to Access Ports

```plaintext
Switch(config)# interface GigabitEthernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10

Switch(config)# interface GigabitEthernet 0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
```
### Trunking with 802.1Q

Trunk links carry multiple VLANs over a single physical connection:

- **Tagging**: Inserts a 4-byte VLAN ID into Ethernet frames.
- **Native VLAN**: Frames remain untagged; best practice is to use a non-default VLAN.

### Configure a Trunk Port
```plaintext
Switch(config)# interface GigabitEthernet 0/3
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,99,100
```
> **📌 Exam Tip**: Always prune unused VLANs on trunk links and change the native VLAN from 1 to something else (e.g., 999).

---
## 2. VLAN Trunking Protocol (VTP)
**VTP** automates VLAN creation and distribution across interconnected switches within a VTP domain.

| VTP Mode       | Function                                                   |
|---------------|------------------------------------------------------------|
| **Server**     | Creates/modifies/deletes VLANs and propagates changes.     |
| **Client**     | Receives and applies VLAN updates; cannot create or edit.  |
| **Transparent**| Maintains local VLAN database; does not propagate changes. |

### VTP Configuration
```plaintext
Switch(config)# vtp domain MyCompany
Switch(config)# vtp mode server
Switch(config)# vtp version 2
```
- Works only on **trunk ports**.
- Check status with:
  ```plaintext
  Switch# show vtp status
  ```
> **📌 Exam Tip**: Misconfiguring VTP (especially **Server** mode) can remove VLANs across an entire domain. Many organizations opt for **Transparent** mode or disable VTP to avoid unintended changes.

---

## 3. Inter-VLAN Routing
Devices in different VLANs require a Layer 3 device to forward traffic between them.

### a) Router-on-a-Stick
Utilizes **subinterfaces** on a single router port:

- Each subinterface is assigned a VLAN ID with `encapsulation dot1Q`.
- Each subinterface has a unique IP address for its VLAN.

**Example**:
```plaintext
Router(config)# interface GigabitEthernet 0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0

Router(config)# interface GigabitEthernet 0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
```

### b) Layer 3 Switch (SVI)
**Switched Virtual Interfaces** act like router interfaces inside the switch.

1. Enable IP routing on the switch:
   ```plaintext
   Switch(config)# ip routing
   ```
2. Create an SVI for each VLAN:
   ```plaintext
   Switch(config)# interface vlan 10
   Switch(config-if)# ip address 192.168.10.1 255.255.255.0
   ```
3. Verify IP routes:
   ```plaintext
   Switch# show ip route
   ```
> **📌 Exam Tip**: Be prepared to compare **Router-on-a-Stick** versus **Layer 3 SVIs**. Layer 3 switches typically deliver faster inter-VLAN routing due to hardware acceleration.

---

## 4. VLAN Access Control Lists (VACLs)
**VACLs** filter traffic within the same VLAN at **Layer 2**:

- Different from traditional ACLs, which filter at Layer 3 (routed interfaces).
- Applied to **VLANs**, not interfaces.

**Example**:
```plaintext
Switch(config)# access-list 101 permit tcp any any eq 80
Switch(config)# vlan access-map HTTP_FILTER 10
Switch(config-access-map)# match ip address 101
Switch(config-access-map)# action forward
Switch(config)# vlan filter HTTP_FILTER vlan 10
```
> **📌 Note**: VACLs operate within a VLAN (**intra-VLAN**). Use standard ACLs or firewall features for **inter-VLAN** traffic filtering.

---

## 5. VLAN Security Threats & Mitigation

### VLAN Hopping
Attackers inject **tagged frames** to gain unauthorized access.  
**Mitigation**:
```plaintext
Switch(config-if)# switchport mode access
Switch(config-if)# switchport nonegotiate
```
- Change native VLAN from the default VLAN 1.

### Double Tagging
Attackers craft frames with **two VLAN tags**; the second tag is read after the first is stripped.  
**Mitigation**:
- Use a **non-default native VLAN**.  
- Prune unnecessary VLANs on trunks.

### Rogue DHCP Server
An attacker introduces a **fake DHCP server** to hijack IP addresses.  
**Mitigation**:
```plaintext
Switch(config)# ip dhcp snooping
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# ip dhcp snooping trust
```
> **📌 Exam Tip**: Expect questions on VLAN hopping and how to secure trunks (e.g., `switchport nonegotiate`, native VLAN changes, DHCP snooping).

---

## 6. Best Practices & Troubleshooting

### Best Practices
- Use a **dedicated VLAN** for management (e.g., VLAN 99).
- Disable **unused ports** (`shutdown`) to reduce attack surface.
- Limit **allowed VLANs** on trunk ports to only necessary VLANs.
- Avoid **VLAN 1** for native or critical traffic; rename or move it.
- Enable **DHCP Snooping** and other security measures (e.g., ARP inspection) to protect endpoints.

### Common Troubleshooting Commands

- **Check VLAN Assignments**:
  ```plaintext
  Switch# show vlan brief
  ```
- **Verify Trunk Status**:
  ```plaintext
  Switch# show interfaces trunk
  ```
- **Check Router-on-a-Stick Configuration**:
  ```plaintext
  Router# show ip interface brief
  ```
- **View Layer 3 Routes**:
  ```plaintext
  Switch# show ip route
  ```
- **Verify VTP**:
  ```plaintext
  Switch# show vtp status
  ```
- **Check VACL**:
  ```plaintext
  Switch# show vlan access-map
  ```



## 🔥 Exam Alert: Key Focus Areas for ENCOR 350-401

- **VLAN Fundamentals**: Types of VLANs, configuration, best practices  
- **802.1Q trunking**: Native VLAN, tagging, pruning  
- **VTP**: Server, Client, Transparent modes, domain matching, and risks  
- **Inter-VLAN Routing**: Router-on-a-Stick vs. Layer 3 Switch (SVI)  
- **VACLs**: Layer 2 filtering inside a VLAN  
- **Security**: VLAN hopping, double tagging, rogue DHCP servers, and mitigation

