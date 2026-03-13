# Virtualized Enterprise Infrastructure Lab (VMware ESXi)

Enterprise-style VMware ESXi 7 homelab built on physical hardware for hands-on virtualization, infrastructure, and systems administration practice.

This lab is designed to simulate a small business environment using real-world infrastructure components including Active Directory, file services, DNS, DHCP, web servers, print server, Windows clients, and Linux-based Apache.

---

## Lab Objectives

- Deploy VMware ESXi 7 on bare metal
- Design and manage VM storage architecture
- Deploy and manage Windows and Linux virtual machines
- Build a multi-server Active Directory environment
- Configure core infrastructure services (DNS, DHCP, File Services, Linux-based Apache, Web Server, Print Server, Windows Clients)
- Practice enterprise-style resource allocation and capacity planning
- Prepare for real-world systems administration and virtualization roles

---

## Project Phasing Model

This homelab is organized and documented in structured build phases.

Each phase represents a defined milestone in the infrastructure deployment process.  
Phases build on one another and simulate how real-world infrastructure projects are executed incrementally.

The repository will be progressively reorganized into clear phase-based documentation:

- [Phase 1 – ESXi Installation & Core VM Deployment](01-esxi-and-vm/esxi-and-vm-configuration.md)
- [Phase 2 – Core Network and Domain Infrstructure (DNS, DHCP, File Services, Windows Clients)](02-core-network-and-domain/README.md)
- [Phase 3 - Server Roles & Application Services](03-server-roles-and-application-services/README.md)
- [Phase 4 - Secure Web Infrastructure (PKI & SSL)](04-secure-web-infrastructure-pki-ssl/README.md)


Each phase will contain:

- Goals and objectives
- Infrastructure design decisions
- Step-by-step implementation notes
- Concept explanations
- Validation testing
- Issues encountered and resolutions

---

## Repository Structure

```
esxi-7-homelab/
│
├── 01-esxi-and-vm/
│   ├── esxi-and-vm-configuration.md
│   ├── bios-configuration.md
│   └── screenshots/
│
├── 02-core-network-and-domain/
│   ├── README.md
│   ├── ip-assignment.md
│   ├── ad-and-dns-configuration.md
│   ├── fs-and-dhcp-configuration.md
│   └── screenshots/
│
├── 03-server-roles-and-application-services/
│   ├── README.md
│   ├── ad-users-and-ntfs-permissions.md
│   ├── print-server.md
│   ├── ubuntu-apache-domain-join-ssh.md
│   ├── windows-web-service-iis.md
│   └── screenshots/
│
├── 04-server-roles-and-application-services/
│   ├── README.md
│   ├── ad-certificate-authority.md
│   ├── iis-ssl-certificate-deployment.md
│   ├── apache-ssl-certificate-deployment.md
│   └── screenshots/
│
├── troubleshooting.md
│
└── README.md
```

Each directory contains step-by-step documentation of the lab build process.

---

## Current Hardware Configuration

**Host Machine**
- Intel i5-8600K (6 cores / 6 threads)
- 32GB RAM
- 500GB Seagate HDD (ESXi installation)
- 256GB Samsung 850 EVO SSD (VM datastore)
- Additional SSD storage planned for future expansion

---

## Storage Design

- ESXi installed on dedicated 500GB HDD
- 256GB SSD used exclusively for VM storage (VMFS datastore)
- Future expansion: Additional 512GB–1TB SSD for increased VM capacity

---

## Planned Virtual Infrastructure

Planned VM deployment (phased build):

## Planned VM Deployment

| VM Name        | Roles / Services | OS | RAM |
|----------------|--------------------------------------------|-------------------------|------|
| WS2019-DC01    | Active Directory Domain Services, DNS | Windows Server 2019 | 6GB |
| WS2019-FS01    | File Services, DHCP, Print Services | Windows Server 2019 | 6GB |
| Ubuntu-SRV01   | Apache Web Server | Ubuntu Server 24.04 LTS | 4GB |
| Win11-Client01 | Domain-Joined Client | Windows 11 | 6GB |

Services will be deployed incrementally.

---

## Long-Term Goals

- Implement vCenter Server (future phase)
- Expand storage capacity
- Practice high-availability concepts
- Build a realistic enterprise-style virtual infrastructure
- Use lab as portfolio evidence for IT roles
