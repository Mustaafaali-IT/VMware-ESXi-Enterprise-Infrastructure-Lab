# Windows Web Service Configuration (IIS) (WS2019-FS01)

## Summary

This document covers the installation and configuration of Internet Information Services (IIS) on WS2019-FS01 within the isolated lab network.

The goal is to deploy a Windows-based web service that operates alongside the existing Linux Apache server. This demonstrates how both Windows and Linux systems can host web applications within the same Active Directory environment.

This configuration includes installing the IIS role, validating the default web service, and confirming that the server can host and serve web content across the internal lab network. Client-side access from the Windows 11 domain-joined machine is also tested to verify proper connectivity, name resolution, and service availability.

---

## VMs In Use

| VM Name         | Operating System        | vCPU | RAM | Disk  | Provisioning | Assigned Roles                 |
|-----------------|------------------------|------|-----|-------|-------------|--------------------------------|
| WS2019-DC01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thick       | Active Directory, DNS |
| WS2019-FS01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thin        | File Services, DHCP, Print Server |
| W11-CL01     | Windows 11 Pro      | 2    | 6GB  | 60GB | Thin        | Client machine|

**Note:** WS2019-DC01 was provisioned as Thick by default. This does not negatively impact lab functionality and was intentionally left unchanged.

---

## Prerequisites

### Phase 2 - Core Networking Infrastructure

Before user creation and file sharing you must first create the core networking infrastructure.

See the [Phase 2 – Core Network and Domain Infrstructure (DNS, DHCP, File Services, Windows Clients)](../02-core-network-and-domain/README.md) for steps on completion.