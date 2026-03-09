# Phase 4 — Secure Web Infrastructure (PKI & SSL)

## Goal

The objective of Phase 4 is to introduce secure communication and certificate-based trust within the lab environment by implementing a Public Key Infrastructure (PKI) and enabling SSL/TLS encryption for internal web services.

In this phase, Active Directory Certificate Services (AD CS) will be deployed to create an internal enterprise Certificate Authority. This allows domain machines to automatically trust certificates issued within the lab environment, simulating how organizations manage secure communications internally.

Certificates issued by the domain PKI will then be used to secure the web services deployed in Phase 3. Both the Windows IIS intranet site and the Apache web server on Ubuntu will be configured to support HTTPS using certificates issued by the internal CA.

This phase focuses on demonstrating how organizations secure internal services through centralized certificate management, encrypted web traffic, and trusted identity infrastructure.

---

## What was built

In progress

---

## Order of Documentation

1. [Active Directory Certificate Authority](ad-certificate-services.md)  
2. [IIS HTTPS Configuration](iis-https-configuration.md)  
3. [Apache HTTPS Configuration](apache-https-configuration.md)

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
| WS2019-DC01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thick       | Active Directory, DNS, AD CS |
| WS2019-FS01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thin        | File Services, DHCP, Web Service, Print, IIS |
| Ubuntu-SRV01    | Ubuntu Server LTS      | 2    | 4GB | 30GB  | Thin        | Apache Web Server, OpenSSH |
| W11-CL01     | Windows 11 Pro      | 2    | 6GB  | 60GB | Thin        | Client machine - (Lab User & Lab Guest)|

**Note:** WS2019-DC01 was provisioned as Thick by default. This does not negatively impact lab functionality and was intentionally left unchanged.

---

## Active Directory Certificate Authority (WS2019-DC01)

For actual steps, view the [Active Directory Certificate Authority documentation](ad-certificate-services.md).

In this section, **Active Directory Certificate Services (AD CS)** is deployed to establish an internal Enterprise Certificate Authority for the `lab.local` domain. This allows the lab environment to issue trusted certificates to internal services and domain machines.

Implementing a domain-integrated Certificate Authority simulates how organizations manage internal certificates for secure services such as HTTPS, authentication, and encrypted communication.

---

## IIS SSL Certificate Deployment

For detailed steps, view the [IIS SSL Certificate Deployment documentation](iis-ssl-certificate-deployment.md).

In this section, a certificate issued by the internal Certificate Authority is deployed to the IIS server hosted on `WS2019-FS01`. The certificate is installed and bound to the **LAB-Intranet** website so the service can be accessed securely using HTTPS through `https://intranet.lab.local`.

Because the certificate is issued by the domain Certificate Authority, domain-joined machines automatically trust the connection. This simulates how organizations secure internal web services using centrally managed certificates instead of relying on external providers.

---

## Apache SSL Certificate Deployment

For actual steps, view the [Apache SSL Certificate Deployment documentation](apache-ssl-certificate-deployment.md).

In progress.