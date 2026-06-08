# 🎫 osTicket Helpdesk Lab

A hands-on helpdesk lab built to simulate a real-world IT support environment. osTicket is deployed on Ubuntu Server running in Hyper-V, configured with departments, SLA plans, and agents — extending the [Windows Server AD Lab](https://github.com/hallowseph/windows-server-lab).

---

## 🧭 Lab Overview

| Component | Details |
|---|---|
| **Ticketing System** | osTicket v1.18.2 |
| **Server OS** | Ubuntu Server 26.04 LTS |
| **Hypervisor** | Microsoft Hyper-V |
| **PHP Version** | 8.5.4 |
| **Web Server** | Apache 2.4.66 |
| **Database** | MySQL |
| **Lab Goal** | Simulate L1/L2 helpdesk workflows |

---

## 🗺️ Network Topology

```
Internet
    |
[pfSense 192.168.10.254] — Firewall, NAT
    |
[LabSwitch — 192.168.10.0/24]
    |
    ├── DC01 (192.168.10.1) — AD DS, DNS, DHCP
    ├── DC02 (192.168.10.2) — Secondary DC
    ├── FS01 (192.168.10.3) — File Server
    ├── WEB01 (192.168.10.4) — IIS
    ├── WSUS01 (192.168.10.5) — Patch Management
    └── osticket-server (192.168.10.6) — osTicket Helpdesk
```

---

## 🏗️ What Was Built

### Infrastructure
- Ubuntu Server 26.04 LTS VM deployed on Hyper-V with static IP
- LAMP stack installed and configured (Apache, MySQL, PHP 8.5)
- osTicket v1.18.2 installed and secured

### Helpdesk Configuration
- **3 Departments** — Support, IT Support, Systems Administration
- **4 SLA Plans** — Sev-1 Critical (1hr/24/7), Sev-2 High (4hr/24/7), Sev-3 Normal (8hr/business hours), Default
- **7 Help Topics** — Password Reset, Software Installation, Hardware Issue, Network Connectivity, New User Onboarding, System Outage, VPN Access
- **3 Agents** — Admin, L1 Support (Sarah Miller), L2 Support (James Carter)
- **2 Teams** — L1 Support, L2 Support
- **3 End Users** — John Smith, Emily Davis, Mike Johnson

### Ticket Scenarios

| Ticket | Type | SLA | Agent | Outcome |
|---|---|---|---|---|
| Password Reset | L1 | Sev-3 Normal | Sarah Miller | Resolved |
| Hardware Issue — laptop not turning on | L1 | Sev-2 High | Sarah Miller | Resolved |
| VPN Access — certificate expired | L2 | Sev-2 High | James Carter | Resolved |
| System Outage — file server unreachable | L2 | Sev-1 Critical | James Carter | Resolved in 13 min |

---

## 📁 Documentation

| File | Description |
|---|---|
| [setup.md](setup.md) | Full installation walkthrough — Ubuntu VM, LAMP stack, osTicket |
| [configuration.md](configuration.md) | osTicket configuration — departments, SLAs, help topics, agents |
| [ad-integration.md](ad-integration.md) | LDAP/AD integration attempt and PHP 8.5 compatibility findings |
| [ticket-scenarios.md](ticket-scenarios.md) | Four simulated L1/L2 ticket walkthroughs end-to-end |
| [troubleshooting.md](troubleshooting.md) | Issues encountered during the build and how they were resolved |

---

## 🔧 Troubleshooting Highlights

This lab was built on Ubuntu 26.04 LTS with PHP 8.5.4 — both very new releases at the time of building. Several compatibility issues were encountered and documented:

- `php-imap` not available on Ubuntu 26.04 (email-to-ticket feature unavailable)
- osTicket LDAP plugin incompatible with PHP 8.5 (abstract class enforcement changes)
- `Net_LDAP2` PEAR library deprecated in PHP 8.5

Full details in [troubleshooting.md](troubleshooting.md).

---

## 🔗 Related Projects

- [windows-server-lab](https://github.com/hallowseph/windows-server-lab) — The AD environment this lab extends

---

## ✅ Status

**Complete.** All phases built and documented.