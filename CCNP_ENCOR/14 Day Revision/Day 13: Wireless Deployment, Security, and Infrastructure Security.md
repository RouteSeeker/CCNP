# Day 13: Wireless Deployment, Security, and Infrastructure Security

---

## WHY WIRELESS DEPLOYMENT, SECURITY, AND INFRASTRUCTURE SECURITY MATTER

In modern enterprise environments, **wireless connectivity** and **infrastructure security** go hand in hand. As networks become more distributed and user mobility grows, it’s essential to:

1. **Deploy access points** in the right mode (Local, FlexConnect, or Bridge) to fit various scenarios (campus vs. branch vs. wireless bridging).
2. **Configure WLC** settings for SSIDs and basic security to ensure stable, high-performing Wi-Fi coverage.
3. **Optimize coverage** and roaming so users can seamlessly move around without losing connectivity.
4. **Secure the entire environment** with robust authentication (802.1X, RADIUS integration) and enforce best practices on the wired side (port security, SSH-based device management).

By combining **proper wireless deployment** with **strong security** across both wired and wireless infrastructures, network engineers can deliver the speed and flexibility users expect—without sacrificing safety or reliability.

---

## OBJECTIVES

By the end of **Day 13**, you will be able to:

1. **Explain AP Modes** – Understand Local, FlexConnect, and Bridge modes, and when to use them.
2. **Configure WLC Interfaces and WLANs** – Build basic wireless networks, assigning VLANs and security.
3. **Optimize Coverage with RF Profiles** – Manage channel and power settings for robust coverage.
4. **Define Roaming Types** – Contrast Layer 2 vs. Layer 3 roaming and how they impact client mobility.
5. **Explain 802.1X Authentication** – Know how EAP processes integrate with RADIUS for secure wireless.
6. **Set Up a RADIUS Server** – Enable enterprise WLAN authentication.
7. **Contrast 802.1X, MAB, and WebAuth** – Recognize each method’s use case for user/device validation.
8. **Configure Port Security on Switches** – Limit MAC addresses and detect unauthorized devices.
9. **Secure Device Access** – Use SSH and password best practices for infrastructure protection.

---

## 1. ACCESS POINT MODES: LOCAL, FLEXCONNECT, AND BRIDGE

### 1.1 Local Mode (Centralized)

**Local mode APs** tunnel all client traffic back to the **WLC** using CAPWAP:

- **Best used** in larger campuses with reliable bandwidth to the controller.
- **All data** is switched by the controller, simplifying policy enforcement in one location.
- **Drawback**: If the WAN link to the WLC goes down, AP can’t service clients (unless fallback is configured).

> 🔥 **Exam Tip**: This is the default mode for campus deployments. The WLC does the heavy lifting (DHCP, user VLAN assignment).

### 1.2 FlexConnect Mode (Local Switching)

**FlexConnect** (formerly H-REAP) allows APs to locally switch traffic at remote sites:

- Minimizes backhaul across the WAN for local traffic (e.g., local printer or internet breakouts).
- If the WLC link fails, the AP can keep servicing its clients with minimal functionality (standalone mode).
- Common in branch deployments with intermittent WAN or desire to reduce latency.

> 🔥 **Exam Tip**: Ideal for distributed offices—policy is still centrally defined but “local switching” offloads local traffic.

### 1.3 Bridge Mode (Mesh / Point-to-Point)

**Bridge mode** APs form a **wireless bridge** linking two or more separate networks:

- Typically used for building-to-building links or sensor networks.
- The AP acts as a wireless backhaul, bridging entire subnets or VLANs across the link.

> 🔥 **Exam Tip**: In campus or industrial settings (like connecting distant warehouses), bridging can save cost on trenching or fiber lines.

---

## 2. CONFIGURE WLC INTERFACES AND WLANS

### 2.1 WLC Interfaces

Common WLC interfaces:

- **Management**: For WLC configuration/management, reachable by APs.
- **AP Manager**: In older designs, separate IP used by APs. Now often combined with management in newer controllers.
- **Virtual**: Used for web auth and mobility tasks.
- **Dynamic (VLAN)**: Binds a WLAN to a specific VLAN, used for user traffic.

> 🔥 **Exam Tip**: Ensure trunking is correct between the WLC and the switch. If the dynamic interface’s VLAN isn’t allowed on the trunk, clients get no DHCP or network access.

### 2.2 Create a Basic WLAN

1. **WLAN > Add**: Give it a name and SSID.
2. **Security**: Choose WPA2/WPA3, 802.1X, or PSK.
3. **Interface**: Map to a dynamic interface for user VLAN.
4. **Enable**: Mark the WLAN as active so APs can broadcast it.

**Optional** advanced settings: QoS profiles, band steering, or application visibility.

---

## 3. OPTIMIZE COVERAGE WITH RF PROFILES

### 3.1 RRM (Radio Resource Management)

**RRM** automatically assigns channels and power levels:

- **DCA (Dynamic Channel Assignment)**: Minimizes co-channel interference.
- **Tx Power Control (TPC)**: Ensures coverage without overshooting.
- **Coverage Hole Detection**: Alerts if client signals are too low.

> 🔥 **Exam Tip**: RRM is generally recommended for most environments. Only override if you have a unique environment needing strict manual channel/power.

### 3.2 RF Profiles

Different areas (high-density auditorium vs. standard office) might need distinct radio settings:

- “High Density” profile: Lower power, narrower channels, fewer SSIDs to reduce overhead.
- “Low Density” or “Typical” profile: Balanced power, standard channels.

---

## 4. DEFINE ROAMING TYPES: LAYER 2 AND LAYER 3

### 4.1 Layer 2 Roaming

- Client stays on the **same IP subnet** as it moves between APs.
- The WLC ensures the client’s identity (MAC, keys) is quickly recognized by all APs in that mobility domain.

### 4.2 Layer 3 Roaming

- Client crosses **different subnets** or VLANs.
- The original WLC or anchor keeps track of the client’s IP so it remains unchanged.
- More complex than Layer 2 roaming, but essential for large campus with multiple VLANs.

> 🔥 **Exam Tip**: 802.11r (Fast BSS Transition) can speed up handoffs, especially beneficial for voice or real-time applications.

---

## 5. EXPLAIN 802.1X AUTHENTICATION

### 5.1 802.1X Basics

**802.1X** uses a Supplicant (client), Authenticator (WLC/AP or switch), and Authentication Server (RADIUS) to control access:

- **EAP** frames pass from the client to the RADIUS server via the WLC.
- Secure credentials ensure only authorized users/devices connect.

### 5.2 RADIUS Server Setup

Define a RADIUS server on the WLC:

```bash
radius server MYRADIUS
 address ipv4 192.168.50.10 auth-port 1812 acct-port 1813
 key RADIUS-SECRET
```

Then specify the server in the WLAN security settings. The RADIUS server checks credentials (e.g., AD, local DB) before granting access.

> 🔥 **Exam Tip**: 802.1X + RADIUS is considered the gold standard for enterprise WLAN security.

---

## 6. EXPLAIN 802.1X, MAB, AND WEB AUTHENTICATION

### 6.1 802.1X

- **Full EAP** authentication using a supplicant.
- Most secure, typically for corporate devices.

### 6.2 MAB (MAC Authentication Bypass)

- For devices lacking 802.1X capability (printers, IP phones).
- Relies on the MAC address as an “identity,” easily spoofed.

### 6.3 WebAuth (Captive Portal)

- Redirects users to a login page.
- Common for guests, minimal device support needed (just a browser).

> 🔥 **Exam Tip**: Each method addresses different device populations and security levels.

---

## 7. CONFIGURE PORT SECURITY ON SWITCHES

### 7.1 Port Security Basics

**Port security** can limit the MAC addresses that appear on a switch port:

- **Maximum** MAC count (e.g., 2).
- **Violation** actions (protect, restrict, shutdown).
- **Sticky** learning for automatic MAC address capture.

**Example**:
```bash
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
```

> 🔥 **Exam Tip**: Good for small office or edge ports with known devices. For large environments, consider 802.1X or MAB.

---

## 8. SECURE DEVICE ACCESS WITH SSH AND PASSWORDS

### 8.1 SSH vs. Telnet

- **Telnet** is unencrypted, not recommended.
- **SSH** uses RSA keys for encrypted CLI sessions.

Enable SSH:
```bash
ip domain-name example.com
crypto key generate rsa modulus 2048
ip ssh version 2
!
line vty 0 4
 transport input ssh
```

### 8.2 Password Management

- **enable secret** for encrypted privileged EXEC.
- **Line console/vty** for local user or AAA-based logins.
- **Service password-encryption** obfuscates text passwords (though not highly secure).

> 🔥 **Exam Tip**: For robust security, prefer AAA with TACACS+ for device login. Keep track of user activity logs.

---

## 🔥 EXAM TIPS

1. **AP Modes** – Local, FlexConnect, Bridge. Know the scenarios and trade-offs.
2. **WLC Interfaces & WLANs** – Understand mapping to VLANs, security choices (PSK, 802.1X).
3. **RF Profiles & RRM** – Let the controller manage channels/power unless there’s a specific need.
4. **Roaming** – Layer 2 (same subnet), Layer 3 (mobility anchor for different subnets). 802.11r speeds up transitions.
5. **802.1X** – Main enterprise authentication. RADIUS server integration is key.
6. **MAB vs. WebAuth** – For devices without 802.1X or for guest portals.
7. **Port Security** – Simple approach to limit MAC addresses on access ports.
8. **SSH & Passwords** – Only use secure management protocols, strong secrets, and AAA if possible.
