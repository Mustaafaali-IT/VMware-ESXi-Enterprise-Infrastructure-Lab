# Print Server and Virtual Printer

## Summary

This document covers the installation and configuration of the Print Services role on WS2019-FS01 within the domain environment.

A virtual printer is created and published through Active Directory to simulate centralized printer deployment in a business network. The printer is then accessed from the Windows 11 domain-joined client to validate authentication, resource discovery, and client-side installation.

The goal of this section is to demonstrate how shared resources can be centrally managed and distributed through the domain, reinforcing role-based server design and domain-integrated service deployment within the isolated lab network.

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

## Print Server Role Installation and Initial Printer Configuration

The **Print and Document Services** role was installed on `WS2019-FS01`.

### Role Installation

The **Print Server** role service was added to `WS2019-FS01` using Server Manager. Only the core Print Server component was installed. Optional components such as LPD Service and Internet Printing were intentionally excluded to keep the environment aligned with a modern internal domain-based deployment.

The goal at this stage is a centralized domain-managed print services within the isolated lab subnet.

---

### Virtual Printer Configuration 

Because this is a fully isolated virtual lab with no physical printer hardware, a virtual printer was created instead of configuring a real TCP/IP device.

A **Local Port** was used during setup to eliminate any dependency on external network devices. This keeps the print server completely self-contained inside the lab subnet.

During initial configuration, the built-in Microsoft PDF/XPS drivers were tested. However, those drivers do not support printer sharing in a Windows Server environment. To properly simulate enterprise print deployment, a share-capable driver was required.

The **Generic / Text Only** driver was manually installed from the built-in Windows Server driver catalog and used for the final configuration. This driver:

- Does not require internet access
- Does not require physical hardware
- Supports printer sharing
- Works cleanly in isolated environments

The printer was named `Lab-Printer01` to align with the structured naming convention used across all lab systems.

---

## Printer Sharing, Client Deployment, and Validation

### Sharing and Publishing the Printer

Once `Lab-Printer01` was created using the Generic / Text Only driver, it was configured as a shared domain resource.

The printer was shared with the name:

`Lab-Printer01`

The option **“List in the directory”** was enabled so the printer object would be published into Active Directory. This allows the printer to exist as a domain resource rather than just a locally shared device.

Publishing to AD aligns with enterprise practice where printers are centrally managed and discoverable through directory services.

---

### Security Configuration

Default permissions were adjusted to follow the same access model used for file shares earlier in the lab.

Changes made:

- Removed `Everyone`
- Added `FileShare-Users` security group
- Granted **Print** permission only
- Retained `Administrators` and `SYSTEM` with management rights

A **Global Security Group** was used intentionally so access can scale cleanly in larger environments. Instead of assigning permissions to individual users, access is controlled at the group level. This reinforces centralized identity and permission management inside Active Directory.

This design keeps resource access consistent across file services and print services.

---

## Client Connection (Windows 11)

On the Windows 11 domain-joined client, the printer was connected using the shared UNC path:

```
\\WS2019-FS01\Lab-Printer01
```

The client was intentionally **not** configured using a local port. Instead, it connects directly to the server's shared printer object. This ensures:

- Authentication is handled through Active Directory
- Jobs are processed by the server’s Print Spooler service
- Permissions are enforced centrally
- No duplicate or independent printer configurations exist on the client

---

## Print Job Validation

A test print was initiated from the Windows 11 client.

On the File Server (WS2019-FS01), the job appeared in the printer queue immediately. The job status showed **Error**, which is expected in this lab environment because:

- No physical printer device exists
- The printer uses a local test port
- There is no hardware endpoint to receive the data

Despite the error state, this confirms successful:

- Client-to-server communication
- SMB connectivity
- DNS resolution
- Active Directory authentication
- Group-based authorization
- Print Spooler functionality
- Centralized job processing

---

## Results

The print server is now fully integrated into the domain environment and functioning as a centralized resource.

The printer was successfully:

- Shared from the File Server
- Published to Active Directory
- Secured using a domain security group
- Connected to from a Windows 11 domain client via UNC path

Print jobs submitted from the client reached the server’s Print Spooler service and appeared in the server queue. Although the jobs show an error state due to the absence of physical printer hardware, this confirms that:

- Domain authentication is working correctly
- Group-based permissions are enforced
- Client-to-server communication is functioning
- DNS resolution and SMB connectivity are operational
- Centralized print management is active

---

## Next Steps

The next step is to begin deploying application services.

The next build will focus on configuring an Apache web server on the Ubuntu Server VM, joining it to the lab.local domain and configuriong SSH. This will include:

- Installing and enabling Apache
- Verifying HTTP service availability
- Creating an internal DNS record for name-based access
- Testing web access from the Windows domain 
- Joining Ubuntu to the Domain ontrollers centralized AD

To view next steps, view the [Ubuntu Apache Web Server, Domain Join and SSH documentation](ubuntu-apache-domain-join-ssh.md)

