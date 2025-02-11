# Day 12: Network Assurance and Wireless Concepts – Monitoring, Analysis, and Next-Gen Wi-Fi

---

## WHY NETWORK ASSURANCE AND WIRELESS MATTER

Modern enterprise networks juggle **wired** and **wireless** connectivity, **real-time** and **bulk** traffic, and the constant need for **visibility** into performance. On one hand, **network assurance**—via tools like **SNMP**, **Syslog**, **NetFlow**, and **DNA Center**—helps maintain a healthy, high-performing infrastructure. On the other hand, **wireless** demands specialized knowledge of **RF fundamentals** and **WLC configuration** to deliver seamless mobility and robust throughput.

By combining **network assurance** for overall monitoring and **wireless best practices** for coverage and capacity, you ensure users enjoy a **stable, secure, and high-speed environment**—whether they’re connected via cable or roaming on campus Wi-Fi.

---

## OBJECTIVES

By the end of **Day 12**, you will be able to:

1. **Explain Key Monitoring Tools** – Understand how **SNMP**, **Syslog**, and **NetFlow** provide essential device, event, and flow-level data.
2. **Configure SNMP** – Set up secure SNMP communities or users for real-time device polling and alerts.
3. **Leverage Flow Analysis** – Use **NetFlow** or advanced telemetry to identify top talkers, spot anomalies, and plan capacity.
4. **Use DNA Center Assurance** – Monitor network health, integrate AI-driven insights for swift troubleshooting.
5. **Understand SPAN, RSPAN, and ERSPAN** – Mirror traffic locally or remotely for deep packet inspection and forensic analysis.
6. **Troubleshoot Wireless with Packet Capture** – Capture 802.11 frames to diagnose association/authentication and roam issues.
7. **Explain Wireless Fundamentals** – Know the differences in frequency bands, 802.11 standards, and how they shape coverage.
8. **Discuss RF Signal Behavior** – Mitigate interference and multipath using correct channel planning and power settings.
9. **Configure Basic WLC Settings** – Create SSIDs, select security methods, and apply RRM for channel/power management.

---

## 1. NETWORK ASSURANCE: MONITORING AND FLOW ANALYSIS

### 1.1 SNMP (Simple Network Management Protocol)

**SNMP** is a core protocol for device status polling and trap-based alerts:

- **Versions**:
  - **v1/v2c**: Community strings, minimal security.
  - **v3**: Encryption, authentication, user-based security model.
- **MIBs** define the variables you can poll (e.g., interface counters, CPU usage).
- **Traps** or **Informs** let devices proactively alert the NMS (e.g., interface down).

**Configuration Example** (SNMPv3):
```
snmp-server group MYSEC-GRP v3 auth
snmp-server user MYUSER MYSEC-GRP v3 auth sha SECRET
snmp-server location HQ
snmp-server contact admin@example.com
```

> 🔥 **Exam Tip**: **SNMPv3** is recommended in production for secure monitoring—avoid plaintext community strings of SNMPv1/v2c.

---

### 1.2 Syslog

**Syslog** standardizes event logging from devices:

- **Severity Levels**: 0 (Emergency) → 7 (Debug).
- **Destinations**: 
  - Console,
  - Local buffer,
  - External syslog server (preferred for historical analysis).

Logging examples:
```
logging trap warnings
logging host 192.168.100.50
logging source-interface Loopback0
```

> 🔥**Exam Tip**: Always configure an **external syslog server** to store logs beyond the device’s limited buffer—critical for forensic or compliance.

---

### 1.3 NetFlow (or IPFIX)

**NetFlow** collects **flow-level data**—who’s talking to whom, on what ports, and how much volume. Useful for:

- **Bandwidth usage** analysis (top talkers, top protocols).
- **Security** (detect abnormal traffic patterns).
- **Capacity planning** (growing link usage over time).

**Basic NetFlow** config:
```
ip flow-export destination 192.168.100.10 9995
ip flow-export version 9

interface GigabitEthernet0/1
 ip flow ingress
 ip flow egress
```

> 🔥 **Exam Tip**: NetFlow v5 is legacy. v9 or IPFIX are more flexible for IPv6/MPLS data.

---

## 2. NETWORK HEALTH MONITORING: DNA CENTER ASSURANCE

### 2.1 DNA Center Assurance Overview

In **SD-Access** or modern Cisco-based campuses, **DNA Center** aggregates data from SNMP, Syslog, NetFlow, and identity sources:

- **Health Scores**: Show the overall status of devices, clients, and applications.
- **AI/ML**: Spots anomalies, degrade trends, or potential misconfigurations, offering recommended remediations.
- **Wired/Wireless** Integration: Single interface to troubleshoot both mediums.

**Use Cases**:
- **Proactive detection**: Pinpoint issues before a user reports them. 
- **Cross-domain correlation**: See if a client’s poor performance is due to a switch interface error, a wireless coverage gap, or app-level constraints.

---

## 3. SPAN, RSPAN, AND ERSPAN FOR TRAFFIC ANALYSIS

### 3.1 SPAN (Switched Port Analyzer)

Mirrors traffic from specific ports/VLANs to a local monitor port:

```
monitor session 1 source interface GigabitEthernet0/2
monitor session 1 destination interface GigabitEthernet0/3
```
- **Local** to the same switch.

### 3.2 RSPAN (Remote SPAN)

**RSPAN** extends SPAN across multiple switches by using a special VLAN trunked to the destination switch’s monitor port.

### 3.3 ERSPAN (Encapsulated RSPAN)

**ERSPAN** encapsulates mirrored traffic in GRE, enabling remote analysis across routed networks or data center interconnects:

- Perfect for sending traffic to a central IDS/IPS or packet capture solution in another site.

> 🔥 **Exam Tip**: SPAN is local, RSPAN is VLAN-based remote, and ERSPAN is IP/GRE-based remote. Each approach is chosen based on distance and network constraints.

---

## 4. TROUBLESHOOT WIRELESS ISSUES WITH PACKET CAPTURE

### 4.1 Why Wireless Packet Capture?

When a user complains of **authentication failures**, **random disconnects**, or **roaming issues**, a packet capture can reveal the raw 802.11 frames (or CAPWAP frames if capturing between AP and WLC). By examining these frames:

- **Authentication** – Check EAP exchanges, WPA2/WPA3 handshakes.
- **Association** – Confirm if the client obtains a valid association ID or if deauthentication frames appear.
- **DHCP** – Validate that the client properly requests and receives an IP address.
- **Roaming** – Inspect how the client transitions from one AP to another, ensuring correct re-association.

### 4.2 Methods to Capture Wireless Traffic

1. **AP in Sniffer Mode** – Some APs can be converted to a dedicated sniffer, sending all frames on a chosen channel to a remote analyzer.
2. **Local Wireless NIC in Monitor Mode** – A laptop with an appropriate adapter can capture over-the-air frames on a specific channel.
3. **WLC Packet Capture** – Some controllers allow capturing control-plane and data-plane traffic (like CAPWAP) for a specific client or radio.

> 🔥 **Exam Tip**: Ensure your capture device is on the **same channel** and the correct band (2.4, 5, or 6 GHz) as the target AP.

---

## 5. WIRELESS FUNDAMENTALS: FREQUENCY BANDS AND STANDARDS

### 5.1 Frequency Bands

- **2.4 GHz**: Channels 1–11 in the US, overlapping except 1/6/11. More interference from devices like microwaves.
- **5 GHz**: Many channels (36–165), less interference, can do wider channels (40/80/160 MHz). Shorter range but better performance.
- **6 GHz (Wi-Fi 6E)**: A newer band with large contiguous spectrum, minimal legacy interference.

### 5.2 802.11 Standards

- **802.11n (Wi-Fi 4)**: Up to ~600 Mbps with MIMO, 2.4 and/or 5 GHz.
- **802.11ac (Wi-Fi 5)**: 5 GHz only, MU-MIMO, channel widths up to 160 MHz.
- **802.11ax (Wi-Fi 6)**: OFDMA for high-density, 2.4/5 GHz, BSS coloring.
- **Wi-Fi 6E**: Extends 802.11ax to 6 GHz.

> 🔥 **Exam Tip**: Wi-Fi 4/5/6 correspond to 802.11n/ac/ax, each bringing throughput and efficiency improvements.

---

## 6. EXPLAIN RF SIGNAL BEHAVIOR AND INTERFERENCE

### 6.1 RF Propagation

Wireless signals degrade with distance (attenuation) and can reflect or scatter off surfaces, creating multipath. Some multipath can benefit MIMO technology, but too much reflection can cause coverage holes or high retries.

### 6.2 Interference

- **Co-Channel Interference**: Multiple APs on the same channel share airtime, reducing throughput.
- **Adjacent Channel Interference**: Overlapping channels degrade performance even more severely.
- **Non-802.11 sources**: Bluetooth, microwaves, radar in DFS channels.

**Mitigation**:
- Dynamic or manual channel planning.
- Proper Tx power to avoid excessive cell overlap.
- Possibly 5 GHz or 6 GHz to reduce legacy interference.

---

## 7. CONFIGURE BASIC WIRELESS SETTINGS ON WLC

### 7.1 Simple SSID Setup

1. **Create WLAN**: Name/SSID, choose security (WPA2/WPA3/802.1X).
2. **Assign Interface**: Map to a VLAN or dynamic interface.
3. **Radio Policy**: 2.4 GHz only, 5 GHz only, or dual-band.
4. **Security**: PSK for small setups, 802.1X for enterprise authentication.

### 7.2 RRM (Radio Resource Management)

Cisco WLC typically handles:
- **Dynamic Channel Assignment (DCA)**
- **Tx Power Control (TPC)**
- **Coverage Hole Detection**

> 🔥 **Exam Tip**: Let RRM do its job unless you have advanced requirements. Automatic optimization is suitable for most enterprise deployments.

---

## 🔥 EXAM TIPS

1. **SNMP, Syslog, NetFlow** – Fundamentals for device polling, logs, and traffic flow analysis. Secure SNMPv3 is best practice.
2. **DNA Center Assurance** – AI-driven analytics for faster troubleshooting in SD-Access environments.
3. **SPAN/RSPAN/ERSPAN** – Local vs. remote vs. GRE-encapsulated traffic mirroring. Use the one that fits your distance and topology.
4. **Wireless Packet Capture** – Must be on correct channel/band. AP sniffer mode or a laptop in monitor mode.
5. **Frequency & Standards** – 2.4 vs 5 vs 6 GHz, and 802.11n/ac/ax differences.
6. **RF Interference** – Co-channel vs. adjacent channel, non-Wi-Fi sources.
7. **WLC Basic Config** – Creating SSIDs, setting security, enabling RRM for auto channel/power.

