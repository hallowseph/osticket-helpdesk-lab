# 🎫 osTicket Helpdesk Lab

A hands-on helpdesk lab built to simulate a real-world IT support environment. osTicket is deployed on Ubuntu Server running in Hyper-V, integrated with an Active Directory domain for user authentication — extending the [Windows Server AD Lab](https://github.com/hallowseph/windows-server-lab).

---

## 🧭 Lab Overview

| Component | Details |
|---|---|
| **Ticketing System** | osTicket (open source) |
| **Server OS** | Ubuntu Server 24.04 LTS |
| **Hypervisor** | Microsoft Hyper-V |
| **AD Integration** | LDAP authentication via existing AD domain |
| **Lab Goal** | Simulate L1/L2 helpdesk workflows |

---

## 🗺️ Network Topology

> *Diagram coming soon — to be added during setup phase*

All VMs reside on the same Hyper-V internal virtual switch, allowing the Ubuntu/osTicket VM to communicate directly with the domain controllers from the Windows Server lab.

---

## 📁 Documentation

| File | Description |
|---|---|
| [setup.md](setup.md) | Full installation walkthrough — Ubuntu VM + osTicket |
| [configuration.md](configuration.md) | osTicket configuration — departments, SLAs, help topics |
| [ad-integration.md](ad-integration.md) | Connecting osTicket to Active Directory via LDAP |
| [ticket-scenarios.md](ticket-scenarios.md) | Simulated L1/L2 ticket walkthroughs |
| [troubleshooting.md](troubleshooting.md) | Issues encountered and how they were resolved |

---

## 🔗 Related Projects

- [windows-server-lab](https://github.com/hallowseph/windows-server-lab) — The AD environment this lab extends

---

## 🚧 Status

> **In progress** — Lab is being built and documented. Pages will be updated as each phase is completed.