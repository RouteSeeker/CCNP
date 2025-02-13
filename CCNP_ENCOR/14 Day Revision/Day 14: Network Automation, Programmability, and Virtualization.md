# Day 14: Network Automation, Programmability, and Virtualization – The Modern Network Frontier

---

## WHY NETWORK AUTOMATION, PROGRAMMABILITY, AND VIRTUALIZATION MATTER

In an era of **software-defined** everything, networks have grown beyond manually configured CLI devices to agile, **policy-driven** infrastructures. **Network automation** and **programmability** enable consistent, repeatable, and policy-driven changes across thousands of devices with minimal human error. Meanwhile, **virtualization** techniques (VRF, GRE, MPLS) underpin multi-tenant architectures, dynamic overlays, and flexible designs that match the agility demanded by modern businesses.

1. **Automation**: Tools like Python and Ansible deliver reproducible, consistent changes across a fleet of routers and switches.  
2. **NETCONF/YANG**: A structured, model-driven approach for device configuration, replacing unstructured CLI text.  
3. **Virtualization**: VRF for traffic separation on a single device, GRE for flexible tunnels, and MPLS for robust multi-tenant or WAN solutions.  
4. **DNA Center**: Cisco’s platform combining automation, analytics, and assurance with a powerful REST API.

By embracing these pillars, **network teams** can evolve from reactive, manual operations to a DevOps-inspired approach—faster, consistent, and ready to meet rapidly changing business demands.

---

## OBJECTIVES

By the end of **Day 14**, you will be able to:

1. **Explain Programmability Concepts** – Understand how Python, Ansible, and REST-based solutions integrate into network automation workflows.  
2. **Configure NETCONF/YANG** – Employ structured, model-driven approaches to device provisioning.  
3. **Write and Use Scripts** – Harness Python or Ansible to automate repetitive tasks, from simple config changes to complex multi-device orchestrations.  
4. **Define REST API Basics** – Understand GET, POST, PUT, and DELETE for controlling network states.  
5. **Use Postman for API Testing** – Interactively explore network device or controller APIs and interpret JSON responses.  
6. **Retrieve Device Inventory via DNA Center API** – Integrate with Cisco’s SD-Access environment for real-time device data.  
7. **Explain Virtualization (VRF, GRE, MPLS)** – Distinguish how each solution fosters traffic isolation or advanced overlay designs.  
8. **Configure VRFs** on Cisco Gear – Spin up separate routing tables and confirm isolation with show commands.  
9. **Define DNA Center’s Capabilities** – See how automation, assurance, and open APIs converge in a single management console.  
10. **Walkthrough: Automation & APIs** – Orchestrate device changes, gather analytics, and unify your environment with Cisco’s DNA Center or external scripts.  
11. **Examine Cisco Tools** – EEM, Tcl scripts, DNA Center for repeated or event-triggered tasks.  
12. **Configure Automated Network Tasks** – Create schedules or triggers for updates, compliance checks, or backups.

---

## 1. EXPLAIN PROGRAMMABILITY CONCEPTS: PYTHON, ANSIBLE, AND REST

### 1.1 Python

**Python** is the de facto scripting language for network automation. It can interact with devices via SSH or APIs, parse text or structured outputs, and integrate with libraries like:

- **netmiko**: A higher-level SSH library for Cisco, Arista, Juniper, etc.  
- **paramiko**: Lower-level SSH library.  
- **requests**: For REST API calls.  
- **pyATS**: Cisco’s test automation framework, bridging config checks and advanced scenario testing.

> 🔥 **Exam Tip**: Basic Python knowledge—functions, loops, data structures—can drastically reduce configuration overhead for repeated tasks (like deploying a new VLAN to 200 switches).

### 1.2 Ansible

**Ansible** is an **agentless** automation tool using YAML playbooks:

- **Inventory**: Group devices by roles (core, distribution, access).  
- **Playbooks**: Outline steps to configure or verify each device’s final “state.”  
- **Modules**: e.g., ios_config for Cisco IOS, nxos_config for NX-OS.

**Advantages**:
- Simplicity: Single file can push uniform changes to many devices.  
- Idempotency: Re-running the same playbook doesn’t break the network if it’s already correct.

> 🔥 **Exam Tip**: Understand the concept of “gather_facts,” how to skip or use it, and how to reference variables in a playbook for dynamic tasks.

### 1.3 REST

**REST** (Representational State Transfer) underpins most modern network controllers (DNA Center, ACI, Meraki). The basics:

- **HTTP Methods**: GET (fetch), POST (create), PUT/PATCH (update), DELETE (remove).  
- **Headers**: Often contain `Authorization: Bearer <token>` or `Basic <encodedcreds>`.  
- **JSON**: Common data format for requests/responses.

**Use Cases**:
- Pull device inventory from a central controller.  
- Update VLAN or ACL info in a single API call that triggers changes on multiple network devices.

---

## 2. CONFIGURE NETCONF/YANG FOR DEVICE MANAGEMENT

### 2.1 NETCONF

**NETCONF** typically runs over SSH (port 830). It:

- Supports “transactional” changes, meaning partial commits can be rolled back.  
- Minimizes unstructured CLI parsing.  
- Allows robust error checking (the device can confirm if a YANG-based config is valid or not).

> 🔥 **Exam Tip**: Tools like “ncclient” in Python or “yangsuite” can help connect, browse YANG models, and push changes systematically.

### 2.2 YANG

**YANG** models define the structure of configuration/state data. Key YANG modules might describe:

- Interfaces, OSPF/BGP processes, VLANs, QoS policies.  
- “**openconfig**” is a vendor-neutral set of YANG models widely adopted.

**Use Cases**:
- Programmatically read interface counters or push BGP neighbors with minimal error risk.

---

## 3. USE SCRIPTS FOR AUTOMATION TASKS

### 3.1 Python Scripting Approach

**Example**: Basic Python to gather “show version” from multiple routers:
```python
from netmiko import ConnectHandler

devices = [
  {'device_type': 'cisco_ios', 'ip': '10.1.1.1','username': 'admin','password': 'Cisco123'},
  {'device_type': 'cisco_ios', 'ip': '10.1.1.2','username': 'admin','password': 'Cisco123'}
]

for dev in devices:
   conn = ConnectHandler(**dev)
   output = conn.send_command('show version')
   print(f"Device {dev['ip']}:\n{output}\n")
   conn.disconnect()
```
> 🔥 **Exam Tip**: netmiko simplifies SSH connections to Cisco/Arista/Juniper devices. Compare to paramiko which is raw SSH.

### 3.2 Ansible Playbook Examples

**ios_config** module example:
```yaml
- name: Deploy Access-List
  hosts: cisco_switches
  gather_facts: no
  tasks:
  - name: create ACL
    ios_config:
      lines:
        - ip access-list extended BLOCK_SSH
        - deny tcp any any eq 22
        - permit ip any any
      provider:
        username: admin
        password: Cisco123
```
Re-run anytime to ensure the ACL is present on all target switches.

---

## 4. DEFINE REST API BASICS: GET, POST, PUT, DELETE

### 4.1 Basic HTTP

- **GET**: “Show me data.”  
- **POST**: “Create something new.”  
- **PUT** or **PATCH**: “Change an existing resource.”  
- **DELETE**: “Remove that resource.”

**Headers** like `Content-Type: application/json` or `Accept: application/json` define how data is parsed. The response typically includes a status code (200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, etc.).

### 4.2 JSON Handling

**Python**:
```python
import requests
resp = requests.get('http://api.example.com/devices', headers=...)
data = resp.json()
for device in data:
   print(device['hostname'], device['ip'])
```

---

## 5. USE POSTMAN FOR API TESTING

### 5.1 Why Postman?

**Postman** is user-friendly for building and testing REST calls before automating them. Capabilities:

- **Collections**: Save sets of requests for a given API.
- **Environment Vars**: Store tokens or base URLs.
- **Integrated Testing**: Write scripts in JavaScript to check response codes, parse JSON, or chain requests.

> 🔥 **Exam Tip**: Once you find the correct REST call in Postman, replicate the headers and payload in your Python or Ansible workflow.

---

## 6. EXAMPLE: RETRIEVE DEVICE INVENTORY VIA DNA CENTER API

**DNA Center** API example:

1. **Authenticate**: POST to `/dna/system/api/v1/auth/token`, get a Bearer token.
2. **GET** `/dna/intent/api/v1/network-device` with that token in the headers.
3. **JSON response** shows device ID, IP, software version, etc.

**Use Cases**:
- Creating dynamic inventory for Ansible.
- Custom dashboards with real-time device status or performance stats.

---

## 7. EXPLAIN VIRTUALIZATION TECHNOLOGIES: VRF, GRE, AND MPLS

### 7.1 VRF (Virtual Routing and Forwarding)

- Creates multiple **isolated routing tables** on the same physical device.
- Good for multi-tenant or departmental separation.
- Combined with subinterfaces or VLAN assignment.

### 7.2 GRE (Generic Routing Encapsulation)

- Encapsulates IP or other Layer 3 traffic in an IP header.
- Used for site-to-site overlays, bridging, or multi-tenant scenarios.
- Typically no encryption, often combined with IPsec.

### 7.3 MPLS (Multiprotocol Label Switching)

- Label-based forwarding, essential for service providers.
- L3VPN for multi-customer or multi-department separation.
- Large enterprises can also adopt it for advanced WAN solutions.

> 🔥 **Exam Tip**: In an enterprise, VRFs may be used for local segmentation even without full MPLS.

---

## 8. CONFIGURE VRFs ON CISCO DEVICES

### 8.1 VRF Setup

```
vrf definition BLUE
!
interface GigabitEthernet0/1
 ip vrf forwarding BLUE
 ip address 10.10.10.1 255.255.255.0
```

Check:
- `show ip route vrf BLUE`
- `ping vrf BLUE 10.10.10.2`

### 8.2 Use Cases

- **Multi-tenant** campus: Each department has a separate VRF.
- **Security**: Strict isolation, no route leakage.
- **Lab vs. Production** on a single router.

---

## 9. DEFINE DNA CENTER’S CAPABILITIES

**Cisco DNA Center** unifies:

- **Automation**: Templates, zero-touch provisioning.
- **Assurance**: Health scores, AI-driven root cause analysis.
- **Policy**: Identity-based micro-segmentation in the campus (SD-Access).
- **Open API**: Let external or custom-coded tools integrate seamlessly.

**Use Cases**:
- **Plug-and-play** branch device deployments.
- **Consistent QoS** or security policy across thousands of devices.
- **API-based** integration with custom dashboards or 3rd-party tools.

---

## 10. WALKTHROUGH: AUTOMATION, ASSURANCE, AND APIs

1. **Network Discovery**: DNA Center finds devices via IP, SNMP, or NetFlow.
2. **Define Policy**: Admin sets up VLANs, ACLs, or segmentation rules.
3. **Push Config**: DNA Center auto-generates the CLI or uses NETCONF to apply config.
4. **Assurance**: Real-time analytics gauge device/client health.
5. **APIs**: Teams can script or integrate external systems (ITSM or CI/CD) with DNA Center’s REST endpoints.

---

## 11. EXPLAIN CISCO TOOLS: EEM, TCL SCRIPTS, AND DNA CENTER AUTOMATION

### 11.1 EEM (Embedded Event Manager)

- Triggers device-local scripts or actions on event (CPU threshold, syslog pattern, interface down).
- Great for small or immediate responses, but not a full-scale solution.

### 11.2 Tcl Scripts

- Embedded in Cisco IOS for older models.
- Replaced by external solutions or DNA Center for large-scale orchestration.

### 11.3 DNA Center Automation

- **Central approach**: uniform policy and standard templates.
- **Day-0** or existing device integration.
- Minimizes partial or inconsistent config changes.

> 🔥 **Exam Tip**: EEM is local, DNA Center is centralized. Large enterprise deployments prefer a global approach.

---

## 12. CONFIGURE AUTOMATED NETWORK TASKS

### 12.1 Scheduling and Triggering

**Ansible** or **Python** can be scheduled with cron jobs or Jenkins pipelines:

- e.g., “nightly compliance check” that ensures each device’s config matches the baseline.
- e.g., “CI/CD for network”: merges new VLAN changes, runs tests in a lab, then pushes to production.

### 12.2 Real-World Example

**Automating VLAN creation** across dozens of switches:

1. Inventory all distribution switches in an Ansible group.
2. Playbook references VLAN variable.
3. Run the playbook; each switch ends with the new VLAN config.
4. Verify with “show vlan brief” or potential netmiko get commands.

---

## 🔥 EXAM TIPS

1. **Programmability** – Distinguish major tools (Python, Ansible) and the concept of REST calls.
2. **NETCONF/YANG** – Understand data model-driven config, how it’s structured and more consistent than CLI scraping.
3. **DNA Center** – Single pane for automation, uses templates, policy, and strong REST APIs.
4. **VRF, GRE, MPLS** – Tools for virtualizing or separating traffic, crucial in multi-tenant or overlay architectures.
5. **EEM vs. External Automation** – EEM triggers local scripts on single devices, external solutions orchestrate multi-node changes.
6. **Automation** – Always test in a lab or limited environment. Watch for partial changes that could disrupt the network.
7. **Security** – Protect your credentials in scripts or Ansible playbooks. Prefer environment variables or vault/encryption solutions for secrets.
