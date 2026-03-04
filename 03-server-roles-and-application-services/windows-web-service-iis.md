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

---

## IIS Installation

The IIS web server role was installed on `WS2019-FS01` using **Server Manager**.

From **Manage → Add Roles and Features**, the **Web Server (IIS)** role was selected and installed with the default role services. These components include the core HTTP service, static content handling, logging, and IIS management tools.

No additional modules were required for this lab environment since the objective was simply to deploy and validate a functioning Windows-based web service within the internal network.

Once the installation completed, the IIS management console became available under: `Server Manager → Tools → Internet Information Services (IIS) Manager`

---

## Web Server Verification

After installation, the IIS service was verified to ensure the web server was running correctly.

Testing was performed locally on the server by opening a browser and navigating to: `http://localhost`

The **default IIS welcome page** loaded successfully, confirming that:

- The IIS service was installed correctly
- The HTTP service was listening on port 80
- The web server was able to serve content from the default web root

---

## Network Access Validation

After confirming the service locally, the web server was tested from the Windows 11 domain-joined client.

A browser on the client machine was used to access the server through the network: `192.168.11.20`

The IIS default page loaded successfully from the client machine, confirming:

- Proper DNS resolution through the Domain Controller
- Successful network communication between the client and file server
- HTTP traffic allowed across the internal network
- The IIS service responding to requests from remote systems

This validation confirms that the Windows server is successfully hosting web content accessible from other machines in the lab environment.

---

## Basic Web Content Deployment

To further validate that IIS was actively serving content, the default web page was modified.

A simple HTML file was placed inside the a newly created directory: `C:\inetpub\intranet`

Rather than modifying the default IIS website, a separate internal site was created to simulate a dedicated intranet service within the lab environment.

## Internal Website Deployment (LAB-Intranet)

Rather than modifying the default IIS website, a separate internal site was created to simulate a dedicated intranet service within the lab environment.

A new IIS site named `LAB-intranet` was created and configured to serve content from a dedicated web root directory. This keeps the default IIS site unchanged while allowing the server to host additional internal web services.

The site was configured with a hostname binding: `intranet.lab.local`

An A record for `intranet.lab.local` was then created in the **Active Directory DNS zone** on the Domain Controller, pointing to the IP address of `WS2019-FS01`.

This allows domain clients to locate the web service using centralized DNS rather than relying on IP addresses.

After the DNS record was created, the site was tested from the Windows 11 domain client by navigating to: `http://intranet.lab.local`

The custom intranet page loaded successfully, confirming that:

- IIS is hosting the new website correctly
- DNS resolution through the Domain Controller is functioning
- Hostname bindings are properly configured
- HTTP traffic is accessible across the internal lab network

--- 

## Results

The IIS web server was successfully installed and deployed on `WS2019-FS01`.

Local testing confirmed that the service was running and listening on the default HTTP port. Network validation from the Windows 11 domain client confirmed proper DNS resolution, client-to-server connectivity, and successful delivery of web content.

## Next steps

