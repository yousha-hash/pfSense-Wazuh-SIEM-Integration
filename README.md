# 🔐 Enterprise Security Monitoring Lab — pfSense + Wazuh SIEM Integration

![Security](https://img.shields.io/badge/Security-Enterprise%20Grade-green)
![pfSense](https://img.shields.io/badge/pfSense-Community%20Edition-blue)
![Wazuh](https://img.shields.io/badge/Wazuh-4.8-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![ITSolera](https://img.shields.io/badge/ITSolera-SOC%20Internship%202026-red)

> A comprehensive security monitoring infrastructure integrating pfSense Firewall with Wazuh SIEM for centralized threat detection, GeoIP blocking, DNS blacklisting, and File Integrity Monitoring.

**🏢 Project completed as part of SOC Winter Internship Program 2026 at ITSolera under the guidance of Dr. Hafeez (CEO)**

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Features Implemented](#-features-implemented)
- [Technologies Used](#-technologies-used)
- [Network Configuration](#-network-configuration)
- [Results Achieved](#-results-achieved)
- [Key Learnings](#-key-learnings)
- [Documentation](#-documentation)
- [Author](#-author)
- [Acknowledgments](#-acknowledgments)

---

## 🎯 Overview

This project demonstrates the design and implementation of a complete enterprise-grade security monitoring infrastructure from scratch. The lab environment simulates a real-world corporate network with:

- **Perimeter Security** — pfSense firewall protecting the network boundary
- **Threat Prevention** — GeoIP blocking and DNS blacklisting to prevent malicious traffic
- **Centralized Monitoring** — Wazuh SIEM collecting and correlating security events
- **Endpoint Protection** — File Integrity Monitoring detecting unauthorized changes
- **Access Control** — Administrator privilege rules restricting management access

---

## 🏗️ Architecture
      ┌──────────────────┐
                     │     INTERNET     │
                     └────────┬─────────┘
                              │
                              │ (NAT - WAN)
                              │
                     ┌────────▼─────────┐
                     │     pfSense      │
                     │    10.10.10.1    │
                     │  ┌─────────────┐ │
                     │  │ pfBlockerNG │ │
                     │  │  • GeoIP    │ │
                     │  │  • DNSBL    │ │
                     │  └─────────────┘ │
                     └────────┬─────────┘
                              │
                              │ (LAN - 10.10.10.0/24)
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌────────▼────────┐ ┌────────▼────────┐ ┌────────▼────────┐
│ Wazuh Server │ │ Windows 10 │ │ Future VMs │
│ 10.10.10.101 │ │ 10.10.10.100 │ │ (Expandable) │
│ ┌───────────┐ │ │ ┌───────────┐ │ │ │
│ │ Manager │ │◄──┤ │Wazuh Agent│ │ │ │
│ │ Dashboard │ │ │ │ FIM │ │ │ │
│ │ Indexer │ │ │ └───────────┘ │ │ │
│ └───────────┘ │ └─────────────────┘ └─────────────────┘
└─────────────────┘
▲
│ Syslog (UDP 514)
│
┌────────┴────────┐
│ pfSense │
│ Remote Logging │
└─────────────────┘


---

## ⚡ Features Implemented

### 🌍 Task A: GeoIP Blocking (Block Specific Countries)

Configured pfBlockerNG to block all inbound and outbound traffic from high-risk countries:

| Country | IP Zone Source | Action | Networks Blocked |
|---------|----------------|--------|------------------|
| China | ipdeny.com/cn.zone | Deny Both | ~8,000+ |
| Russia | ipdeny.com/ru.zone | Deny Both | ~6,000+ |

**Verification:**
```cmd
ping 223.5.5.5    # China (Alibaba DNS) → Request timed out ✓
ping 77.88.8.8    # Russia (Yandex DNS) → Request timed out ✓
ping 1.1.1.1      # USA (Cloudflare)    → Reply received ✓

🚫 Task B: DNS Blacklisting (Restrict Website Access)
Implemented DNSBL to block access to social media and non-work websites:

Domain	Status	Sinkhole IP
facebook.com	⛔ Blocked	172.16.0.1
youtube.com	⛔ Blocked	172.16.0.1
tiktok.com	⛔ Blocked	172.16.0.1
instagram.com	⛔ Blocked	172.16.0.1
twitter.com	⛔ Blocked	172.16.0.1
x.com	⛔ Blocked	172.16.0.1
Configuration:

Wildcard Blocking (TLD): Enabled
DNSBL VIP: 172.16.0.1
Web Server Interface: LAN
DNSBL IPs Action: Deny Both
🔒 Task C: Administrator Privilege Rules
Implemented IP-based access control for firewall management:

Rule	Source	Destination	Action
1	AdminIPs Alias	pfSense:443	✅ Allow
2	Any	pfSense:443	⛔ Block
3	LAN net	Any	✅ Allow
📊 Wazuh SIEM Integration
Configured centralized log collection and security monitoring:

Remote Syslog: pfSense → Wazuh (UDP 514)
Windows Agent: Real-time event forwarding
Log Sources: Firewall, DHCP, System, Authentication
Dashboards: Security events, FIM alerts, Agent status
📁 File Integrity Monitoring (FIM)
Deployed real-time file change detection on Windows endpoints:

Real-Time Monitoring:

C:\Program Files
C:\Program Files (x86)
C:\Windows\System32
C:\Users\Public
Periodic Scanning:

C:\Users (every 12 hours)
Exclusions (Noise Reduction):

C:\Windows\Temp
C:\Users*\AppData\Local\Temp
*.log, *.tmp, *.cache files

FIM Configuration:

<syscheck>
  <disabled>no</disabled>
  <frequency>43200</frequency>
  <directories realtime="yes" recursion_level="2">C:\Program Files</directories>
  <directories realtime="yes" recursion_level="2">C:\Program Files (x86)</directories>
  <directories realtime="yes" recursion_level="2">C:\Windows\System32</directories>
  <directories realtime="yes" recursion_level="2">C:\Users\Public</directories>
  <directories realtime="no" recursion_level="2">C:\Users</directories>
  <ignore>C:\Windows\Temp</ignore>
  <ignore>C:\Users\*\AppData\Local\Temp</ignore>
  <ignore type="sregex">\.log$|\.tmp$|\.cache$</ignore>
  <process_priority>10</process_priority>
  <max_eps>200</max_eps>
  <synchronization>
    <enabled>yes</enabled>
    <interval>5m</interval>
  </synchronization>
</syscheck>

🛠️ Technologies Used
Component	Version	Purpose
pfSense	Community Edition	Firewall / Router / Gateway
pfBlockerNG	devel	GeoIP Blocking + DNSBL
Wazuh	4.8	SIEM + FIM + Agent Management
Windows 10	10.0.19045	Endpoint with Wazuh Agent
Oracle VirtualBox	7.x	Lab Virtualization
Ubuntu Server	22.04 LTS	Wazuh Server OS
🌐 Network Configuration
Device	IP Address	Role
pfSense (LAN)	10.10.10.1	Gateway / Firewall / DNS
Wazuh Server	10.10.10.101	SIEM / Log Collector
Windows 10	10.10.10.100	Endpoint / Wazuh Agent
VM Resource Allocation:

VM	CPU	RAM	Storage	Network Adapters
pfSense	1 Core	2 GB	20 GB	NAT + Internal
Wazuh Server	2 Cores	4 GB	50 GB	Internal
Windows 10	2 Cores	4 GB	50 GB	Internal
📈 Results Achieved
Metric	Value
🌍 GeoIP Networks Blocked	14,000+ (China + Russia)
🚫 DNSBL Domains Blocked	12+ social media sites
🔥 Firewall Blocks Logged	2,500+
📊 Wazuh Alerts Generated	500+
🔒 Admin Access Attempts Blocked	Multiple
📁 FIM Events Captured	Real-time detection active
Wazuh Rules Triggered:

Rule ID	Description
100002	pfBlockerNG IP block
100172	pfBlockerNG DNSBL block
5710	DNS query to blocked domain
554	File added to system
553	File deleted from system
550	File integrity checksum changed
📚 Key Learnings
Through this project, I gained hands-on experience in:

✅ Network Security Architecture — Designing secure network topologies
✅ Firewall Configuration — pfSense rules, aliases, and NAT
✅ Threat Prevention — GeoIP blocking and DNS blacklisting
✅ SIEM Deployment — Wazuh installation, configuration, and log analysis
✅ Endpoint Security — Agent deployment and File Integrity Monitoring
✅ Log Analysis — Security event correlation and investigation
✅ Incident Detection — Real-time alerting and threat identification
✅ Documentation — Professional security reporting

🙏 Acknowledgments
🏢 ITSolera
Thank you to ITSolera for providing this incredible learning opportunity through the SOC Winter Internship Program 2026.

👨‍💼 Dr. Hafeez (CEO, ITSolera)
Special thanks to Dr. Hafeez for his guidance, mentorship, and vision in creating industry-focused cybersecurity training programs.

🌐 Open Source Community
pfSense Team — For the powerful open-source firewall platform
Wazuh Team — For the comprehensive open-source SIEM solution
pfBlockerNG Developers — For the excellent GeoIP and DNSBL package
