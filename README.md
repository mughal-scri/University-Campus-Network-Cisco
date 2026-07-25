# 🏫 University Campus Network Design & Implementation

> A fully functional university campus network simulated in **Cisco Packet Tracer**, featuring fault-tolerant routing, DNS/HTTP services, email, VLANs, ACL-based security, DHCP, and wireless connectivity.

---

## 📌 Project Overview

This project simulates a real-world university building network spanning **10 distinct zones** with complete inter-zone communication, security policies, and network services. Designed and implemented as a **Computer Networks Lab Semester Project** at Air University, Kamra.

---

## 🗂️ Network Zones

| Zone | Subnet | Gateway |
|------|--------|---------|
| Lab 1 (Left) | 192.168.10.0/24 | 192.168.10.1 |
| Lab 2 (Left) | 192.168.20.0/24 | 192.168.20.1 |
| Lab 3 (Right) | 192.168.30.0/24 | 192.168.30.1 |
| Lab 4 (Right) | 192.168.40.0/24 | 192.168.40.1 |
| Faculty Office | 192.168.50.0/24 | 192.168.50.1 |
| Top Classrooms | 192.168.60.0/24 | 192.168.60.1 |
| Exam Office | 192.168.70.0/24 | 192.168.70.1 |
| Bottom Classrooms | 192.168.80.0/24 | 192.168.80.1 |
| HOD Office | 192.168.90.0/24 | 192.168.90.1 |
| Server Room | 192.168.100.0/24 | 192.168.100.1 |

---

## 🔧 Network Topology

```
                    [Central Router]
                   /   |   |   |   \
                  /    |   |   |    \
              Left  Mid-U  Top  Mid-L  Right
              /  \   |  |   |   |  |   /  \
           Lab1 Lab2 Fac HOD  CR  SR Exam Lab3 Lab4
                              |
                           Bottom
                              |
                          Bottom CR
```

**Topology Type:** Hybrid Star-Ring

- **Core Layer** — Central Router (2911) backbone
- **Distribution Layer** — 6x Zone Routers (2911 + HWIC-2T) in a ring
- **Access Layer** — Cisco 2960 Switches per zone

---

## 🖥️ Devices Used

| Device | Model | Count |
|--------|-------|-------|
| Central Router | Cisco 2911 | 1 |
| Zone Routers | Cisco 2911 + HWIC-2T | 6 |
| Switches | Cisco 2960 | 8+ |
| End Devices | PC-PT / Laptop | 100+ |
| Servers | Server-PT | 8 |
| Wireless AP | WRT300N | 2 |

---

## ⚙️ Features Implemented

### 1. 🔁 Routing — RIP v2 (Dynamic)
- RIP version 2 with `no auto-summary`
- Auto-discovers all routes across all zones
- Replaces manual static routing for scalability

### 2. 📡 DHCP
- Configured on each zone router
- Auto-assigns IP, subnet mask, gateway, DNS to all end devices
- Server Room kept static to preserve server IPs

### 3. 🌐 DNS & HTTP Services
| Domain | IP |
|--------|----|
| www.chatgpt.com | 192.168.100.60 |
| www.claude.ai | 192.168.100.70 |
| www.yahoo.com | 192.168.100.20 |
| google.com | 192.168.100.30 |
| www.classroom.google.com | 192.168.100.50 |
| www.aack.au.edu.pk | 192.168.100.40 |

### 4. 📧 Email Service (SMTP/POP3)
- Domain: `aack.au.edu.pk`
- SMTP Port: 25 | POP3 Port: 110
- Server IP: 192.168.100.10
- All PCs configured with email client

### 5. 🔒 Access Control Lists (ACLs)
Access policy based on real university hierarchy:

| Source | Server Room | Faculty | HOD | Exam |
|--------|-------------|---------|-----|------|
| Labs / Classrooms | ✅ | ✅ | ❌ | ❌ |
| Faculty Office | ✅ | — | ✅ | ❌ |
| HOD Office | ✅ | ✅ | — | ✅ |
| Exam Office | ❌ | ❌ | ✅ | — |
| Server Room | — | ✅ | ✅ | ✅ |

> **Design Note:** HOD can initiate contact with Labs (one-way). Labs cannot reply — models real university hierarchy.

### 6. 📶 Wireless Access Points
- Lab 01 Area → 192.168.10.0/24
- Exam Office → 192.168.70.0/24

### 7. 🔄 Fault Tolerance (Original Static Version)
- 3-level floating static routes with Administrative Distance
- AD=1 → Primary via Central
- AD=10 → Backup via Ring Neighbor A
- AD=20 → Backup via Ring Neighbor B

---

## 🔗 WAN Links (Router-to-Router)

All serial links use `/30` subnets (`255.255.255.252`):

| Link | Subnet |
|------|--------|
| Central ↔ Left | 10.1.1.0/30 |
| Central ↔ Mid-Upper | 10.1.2.0/30 |
| Central ↔ Upper | 10.1.3.0/30 |
| Central ↔ Right | 10.1.4.0/30 |
| Central ↔ Mid-Lower | 10.1.5.0/30 |
| Central ↔ Lower | 10.1.6.0/30 |
| Ring: Left ↔ Mid-Upper | 10.0.1.0/30 |
| Ring: Mid-Upper ↔ Top | 10.0.2.0/30 |
| Ring: Top ↔ Right | 10.0.3.0/30 |
| Ring: Right ↔ Mid-Lower | 10.0.4.0/30 |
| Ring: Mid-Lower ↔ Bottom | 10.0.5.0/30 |
| Ring: Bottom ↔ Left | 10.0.6.0/30 |

---

## ✅ Testing Results

| Test | Source | Destination | Result |
|------|--------|-------------|--------|
| Cross-zone ping | Lab 1 PC | Faculty PC | ✅ PASS |
| Server access | Lab 1 PC | IT Manager | ✅ PASS |
| DNS resolution | Any PC | www.yahoo.com | ✅ PASS |
| HTTP access | Any PC | google.com | ✅ PASS |
| ACL: Labs → HOD | Lab PC | HOD PC | ✅ BLOCKED |
| ACL: Faculty → HOD | Faculty PC | HOD PC | ✅ PASS |
| Failover test | Lab PC | Faculty PC (Central DOWN) | ✅ PASS via Ring |

---

## 🚧 Challenges Faced

| Challenge | Solution |
|-----------|----------|
| HWIC-1GE-SFP not linking up | Switched to HWIC-2T serial ports |
| IR8340 had Layer 2 switch ports | Replaced with standard 2911 |
| Serial link IP overlap | Corrected to /30 subnet mask |
| Ring backup causing routing loop | Rebuilt routes with correct ring order |
| ACL blocking HOD reply | Made intentional one-way policy |
| Teammate IP addressing errors | Manually corrected all zones |

---

## 📁 Repository Structure

```
University-Campus-Network-Cisco/
│
├── GroupName_Project_CNLab.pkt   # Cisco Packet Tracer file (Static version)
├── GroupName_Project_RIP.pkt     # Cisco Packet Tracer file (RIP + DHCP version)
├── CN_Project_Report.pdf         # Full project report
├── CN_Project_Presentation.pptx  # Presentation slides
└── README.md                     # This file
```

---

## 🛠️ Tools Used

- **Cisco Packet Tracer** — Network simulation
- **Cisco IOS CLI** — Router and switch configuration
- **MS Word / PDF** — Documentation

---

## 👨‍💻 Authors

| Name | GitHub |
|------|--------|
| Abdullah Mughal | [@mughal-scri](https://github.com/mughal-scri) |
| Huzaifa Abdur Rahman | — |

---

## 🤝 Co-Authors

```
Co-authored-by: Abdullah Mughal <@mughal-scri>
Co-authored-by: Huzaifa Abdur Rahman <>
```

---

## 🏫 Institution

**Air University, Aerospace & Aviation Campus, Kamra**
BS Artificial Intelligence — 2nd Semester
Computer Networks Lab Project — 2026

---

## 📄 License

This project is for educational purposes only.
