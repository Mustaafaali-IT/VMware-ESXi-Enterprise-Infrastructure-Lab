# AD User Creation, File Share and NTFS Permissions

## Summary

This document covers creating domain users in Active Directory and setting up shared folders on the File Server with proper NTFS permissions.

The goal is to simulate a real-world setup where user accounts are centrally managed and access to files is controlled based on permissions. These configurations are then tested from the Windows 11 domain-joined client to confirm that authentication and access control are working correctly within the lab.

We will:

- Create an AD User
- Create an AD Security Group
- Create a Shared Folder on FS01
- Configure NTFS Permissions
- Configure Share Permissions
- Test access from Windows 11 client
- Verify permission behavior

---

## VMs In Use

| VM Name         | Operating System        | vCPU | RAM | Disk  | Provisioning | Assigned Roles                 |
|-----------------|------------------------|------|-----|-------|-------------|--------------------------------|
| WS2019-DC01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thick       | Active Directory, DNS |
| WS2019-FS01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thin        | File Services, DHCP |
| W11-CL01     | Windows 11 Pro      | 2    | 6GB  | 60GB | Thin        | Client machine|

**Note:** WS2019-DC01 was provisioned as Thick by default. This does not negatively impact lab functionality and was intentionally left unchanged.

---

## Prerequisites

### Phase 2 - Core Networking Infrastructure

Before user creation and file sharing you must first create the core networking infrastructure.

See the [Phase 2 – Core Network and Domain Infrstructure (DNS, DHCP, File Services, Windows Clients)](../02-core-network-and-domain/README.md) for steps on completion.

---

## Creating a Domain User and Security Group (WS2019-DC01)

To finish validating the domain setup, I created a standard domain user and a security group in **Active Directory Users and Computers (ADUC)** on `WS2019-DC01`. The goal was to test real authentication and permission-based access using a normal user account instead of relying on the built-in domain administrator.

### Domain User: `LabUser`

A user account named `LabUser` was created under the default **Users** container.

Key decisions:
- **Password never expires** was enabled purely for lab simplicity. This prevents password expiry policies from interrupting testing or breaking access unexpectedly while building and documenting the environment.
- I also disabled “User must change password at next logon” for the same reason—keeping the account immediately usable for repeat testing.

This account will be used to sign into the Windows 11 client and verify domain login + access to shared resources.

---

### Security Group: `FileShare-Users`

Instead of assigning permissions directly to a single user, I created a group called `FileShare-Users` and added `LabUser` as a member. This mirrors how access is usually managed in real environments (permissions are assigned to groups, not individual accounts).

Key decisions:
- **Group scope: Global**  
  Global groups are the standard choice for grouping users within the same domain. Since this lab is a single-domain setup, a Global group is the clean and correct scope to use for controlling access to file shares.
- **Group type: Security**  
  Security groups are used for access control (NTFS permissions, share permissions, etc.). Distribution groups are only for email lists, so Security is the correct type for file share access.

---

## File Share Configuration and Permission Validation

A centralized shared folder was configured on `WS2019-FS01` and secured using domain-based access control. This step validates authentication, group membership enforcement, and NTFS permission layering within the isolated lab network.

### Folder Structure

A dedicated directory was created:

C:\Shares\DepartmentData

The `C:\Shares` root was intentionally used to centralize shared resources rather than sharing random directories. This mirrors common enterprise file server design where shared data is organized under a controlled parent structure.

---

### NTFS Permissions

Access was not assigned directly to individual users. Instead, the previously created security group `FileShare-Users` was granted **Modify** permissions at the NTFS level.

Key decisions:

- **Modify** was chosen instead of Full Control to allow users to create, edit, and delete files without granting unnecessary administrative rights.
- Permissions were assigned to a **security group**, not an individual user, to follow enterprise best practices for scalable access control.
- Built-in accounts such as `SYSTEM` and `Administrators` were preserved to maintain proper server operation and administrative oversight.

NTFS permissions act as the primary security layer and ultimately determine effective access to the resource.

---

### Share Permissions

The folder was shared as:

\\WS2019-FS01\DepartmentData

Share-level permissions were also assigned to `FileShare-Users` with **Change + Read** access.

Share permissions were kept simple and aligned with NTFS permissions. In enterprise environments, NTFS is typically used for granular control while share permissions remain straightforward.

---

### Access Validation

Access testing was performed from the Windows 11 domain-joined client.

- Logging in as `LAB\LabUser` (a member of `FileShare-Users`) allowed full modify access to the share.
- A second domain user not included in the security group was denied access, confirming group-based enforcement was functioning correctly.
- Running `whoami /groups` on the client confirmed that the user’s security token included membership in `FileShare-Users`.

---

### What This Validated

This configuration successfully validated:

- Domain authentication via Kerberos
- DNS-based name resolution to `WS2019-FS01`
- DHCP scope functionality within the isolated subnet
- Group-based permission management
- Proper NTFS and share permission layering
- Controlled access using enterprise design principles

## Results

The domain user and security group were successfully created and integrated into the file access workflow. The shared folder on WS2019-FS01 was properly secured using group-based NTFS and share permissions.

Access validation from the Windows 11 domain-joined client confirmed:

- Successful domain authentication
- Proper group membership enforcement
- Correct NTFS and share permission behavior
- Denial of access for unauthorized users

This confirms that identity-based access control is functioning correctly within the lab and that file services are operating as expected in a domain-managed environment.

## Next Steps

With user management and file access control validated, the next step is to expand server functionality by implementing additional application services.

This includes deploying Print Services through the domain and introducing web services using IIS on Windows Server and Apache on Ubuntu. These additions will further test role-based server design, DNS integration, service publishing, and cross-platform hosting within the isolated lab network.

To view next steps, view the [print server documentation](print-server.md)
