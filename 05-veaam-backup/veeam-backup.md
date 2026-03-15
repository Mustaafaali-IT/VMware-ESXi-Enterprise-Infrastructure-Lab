# Phase 5 - Veaam Backup Soloutions

## Goal

The objective of Phase 5 is to implement a backup and recovery strategy for the lab environment using **Veeam Backup & Replication**. This phase focuses on protecting the virtual infrastructure by creating VM-level backups of the servers running within the ESXi host.

In this phase, Veeam will be deployed and connected to the ESXi environment to allow centralized backup management. Backup jobs will be configured for the domain controller, file server, Ubuntu server, and Windows client to simulate how organizations protect critical systems in production environments.

This setup will demonstrate how virtual machine backups, restore points, and recovery capabilities are managed in enterprise environments. It will also allow testing of backup validation and restore scenarios to ensure infrastructure services can be recovered in the event of system failure or data loss.

---

## What Was Built 

In progress

---

## Architecture snapshot

### Host

- Hypervisor: VMware ESXi 7.0U3
- Boot disk: 500GB Seagate HDD (ESXi installation)
- VM datastore: 256GB Samsung EVO SSD (`datastore-ssd01`, VMFS6)
- Primary NIC: Intel PRO/1000 GT Desktop Adapter
- Onboard Realtek NIC: Disabled in BIOS

---

### VMs

| VM Name         | Operating System        | vCPU | RAM | Disk  | Provisioning | Assigned Roles                 |
|-----------------|------------------------|------|-----|-------|-------------|--------------------------------|
| WS2019-DC01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thick       | Active Directory, DNS         |
| WS2019-FS01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thin        | File Services, DHCP, Web Service, Print, IIS |
| WS2019-VEEAM     | Windows Server 2019    | 2    | 6GB | 60GB + 150GB (Backup Repo)  | Thin        | Veeam Backup & Replication |
| Ubuntu-SRV01    | Ubuntu Server LTS      | 2    | 4GB | 30GB  | Thin        | Apache Web Server, OpenSSH    |
| W11-CL01     | Windows 11 Pro      | 2    | 6GB  | 60GB | Thin        | Client machine - (Lab User & Lab Guest)|

**Note:** WS2019-DC01 was provisioned as Thick by default. This does not negatively impact lab functionality and was intentionally left unchanged.

---

## Veeam Installation, Configuration and testing

### Veaam VM Provisioning (WS2019-VEEAM)

A new virtual machine named `WS2019-VEEAM` was created to run the **Veeam Backup & Replication** server inside the lab environment. The VM was given enough resources to run the backup software and process backup jobs without using unnecessary resources in the homelab.

The following hardware was assigned when creating the VM:

- **vCPU:** 2  
- **Memory:** 8 GB RAM  
- **OS Disk:** 60 GB (Windows Server and Veeam installation)  
- **Backup Repository Disk:** 150 GB (Dedicated backup storage)  
- **Network:** Connected to the isolated lab network (`LAB-ISOLATED` vSwitch)

The 60 GB disk is used for the Windows Server operating system and the Veeam application. A second disk of **150 GB** was added and configured as the backup repository where VM backups will be stored.

Keeping the backup repository on a separate disk helps keep the system organized and mirrors how backup storage is typically separated from the operating system in real environments.
