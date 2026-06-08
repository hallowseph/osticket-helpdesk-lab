# osTicket Helpdesk Lab

A home lab simulating a real enterprise helpdesk environment — osTicket deployed on Ubuntu Server, authenticated against Active Directory.

---

## What This Lab Covers

- Installing and configuring osTicket on Ubuntu Server (Hyper-V)
- Integrating osTicket with Active Directory via LDAP
- Configuring departments, SLA plans, and help topics
- Simulating realistic L1 and L2 support tickets
- Documenting issues and resolutions like a real support environment

---

## Lab Environment

- **Ticketing System:** osTicket
- **Server OS:** Ubuntu Server 26.04 LTS
- **Hypervisor:** Microsoft Hyper-V
- **AD Integration:** LDAP → existing Windows Server domain

This lab extends my [Windows Server AD Lab](https://github.com/hallowseph/windows-server-lab), placing a helpdesk on top of an already-running domain.

---

## Pages

- [Setup Guide](setup) — Installing Ubuntu + osTicket
- [Configuration](configuration) — Departments, SLAs, help topics
- [AD Integration](ad-integration) — Connecting osTicket to Active Directory
- [Ticket Scenarios](ticket-scenarios) — L1/L2 simulated tickets
- [Troubleshooting Log](troubleshooting) — Problems solved during the build