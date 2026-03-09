# Troubleshooting Log – ESXi 7 Homelab

This document records major issues encountered during the ESXi deployment process and how they were resolved.

---

## Unable to Wipe Datastore (Device Busy)

### Issue
Originally tried to use usb as boot device and SSD as storage for temp storage solution. SSD already contained install of ESXi and I planned to wipe it inside ESXi. Attempting to wipe SSD via ESXi failed with “device busy” errors.

### Root Cause
ESXi system partitions were mounted and in use.

### Resolution
- Confirmed active boot disk
- Unmounted mounted filesystems
- Ultimately wiped disk externally using Windows DiskPart

### Lesson Learned
- It's easier to wipe drive with duplicated OS externally rather than innteranlly via same OS

---

## ESXi Web UI – Connection Refused After Moving Behind Switch

### Issue
After moving the ESXi host from a direct router connection to an unmanaged switch, the Web UI became inaccessible.

Symptoms:
- Ping to ESXi IP worked
- rhttpproxy service was running
- ESXi firewall disabled
- Browser error: ERR_CONNECTION_REFUSED
- vmk0 showed Enabled: true with correct static IP and gateway

Despite Layer 3 connectivity working (ping), HTTPS (port 443) connections were being refused.

---

### Root Cause
After the physical network topology changed (router -> switch -> router), the ESXi management network stack did not fully rebind to the new path.

Likely contributing factors:
- Stale ARP entries on router
- MAC address table refresh delay
- Management services not properly binding after static IP was previously configured

The management interface appeared healthy but was not properly accepting TCP 443 connections.

---

### Resolution
1. Changed management network from Static -> DHCP
2. Confirmed Web UI was accessible via new DHCP IP
3. Reconfigured management network back to Static
4. Web UI began functioning normally

Switching to DHCP forced:
- Full network stack refresh
- ARP table renewal
- Proper service binding to vmk0

---

### Lesson Learned
If ESXi Web UI:
- Responds to ping
- rhttpproxy is running
- Firewall is not blocking
- But browser shows “connection refused”

Try restarting the management network or temporarily switching to DHCP to force a clean network rebind after physical network changes.

---

## DHCP Conflict Risk Identified

During DHCP configuration on the Windows Server, it was identified that the lab environment was operating within the same subnet as the home router.

This introduced the risk of dual DHCP servers operating within the same broadcast domain, which can lead to unpredictable IP assignment behavior.

### Resolution

The network architecture was redesigned to use a dedicated vSwitch with no physical uplink, creating a fully isolated internal lab network.

This eliminated any possibility of DHCP conflicts with the home router and provided a controlled testing environment.

---

## Troubleshooting SSH Access with Active Directory Groups

During testing of SSH access control using Active Directory groups, the authorized domain user was initially denied access even though the user had been added to the correct group.

SSH logs showed messages similar to:

```
User labuser@lab.local from <client-ip> not allowed because none of user's groups are listed in AllowGroups
```

This indicated that SSH was not recognizing the user's group membership.

---

## Investigating Group Membership

Group membership was verified directly on the Ubuntu server using:

```
id "labuser@lab.local"
```

and

```
getent group "labsshaccess@lab.local"
```

Both commands confirmed that the user was correctly assigned to the `LabSSHAccess` Active Directory group.

---

## Identifying the Root Cause

The issue was caused by **SSSD caching outdated identity information** from Active Directory.

Since SSSD handles domain identity resolution on the Linux system, group membership changes may not be immediately recognized if cached data is still being used.

---

## Clearing Cached Identity Data

To resolve this issue, the SSSD cache was cleared and the service restarted:

```
sudo systemctl restart sssd
sudo sss_cache -E
sudo systemctl restart ssh
```

After clearing the cache, the SSH login test was repeated and the authorized user was able to successfully authenticate.

---

## Lessons Learned

This troubleshooting process demonstrates a common scenario when integrating Linux systems with Active Directory.

Because SSSD caches identity data, group membership updates in Active Directory may not be immediately recognized by the Linux system. Clearing the cache forces the server to retrieve updated identity information from the domain controller.

Understanding this behavior is important when implementing group-based access controls in enterprise environments.

---

# Troubleshooting

## Windows Client Not Receiving Domain CA Certificate

While verifying that the Windows 11 client trusted the newly configured Active Directory Certificate Authority, the CA certificate did not appear in the **Trusted Root Certification Authorities** store.

---

### Symptoms

Several issues were observed on the Windows 11 client:

- The CA certificate was missing from the trusted root store
- `gpupdate /force` failed with an error indicating the domain controller could not be contacted
- The client could not ping the domain controller
- Running `ipconfig /all` showed an IP address in the **169.254.x.x range**

Example output:

```
Autoconfiguration IPv4 Address : 169.254.x.x
Subnet Mask                    : 255.255.0.0
```

This indicated that the client was not receiving a DHCP lease.

---

### Root Cause

The Windows client was unable to reach the DHCP server. Because no DHCP response was received, Windows automatically assigned itself an **APIPA (Automatic Private IP Address)** in the 169.254.0.0/16 range.

With this address the client had:

- No default gateway
- No valid DNS configuration
- No connectivity to the domain controller

Since Group Policy relies on domain connectivity, the enterprise CA certificate could not be distributed to the client machine.

---

### Resolution

The DHCP service was checked on the file server (FS01). Restarting the DHCP service restored normal DHCP operation.

After restarting the service, the Windows client successfully obtained an IP address from the DHCP scope.

The following commands were then used on the client:

```
ipconfig /release
ipconfig /renew
```

Once the client received a valid IP configuration from DHCP, it was able to communicate with the domain controller again.

Running:

```
gpupdate /force
certutil -pulse
```

allowed the system to refresh Group Policy and pull the enterprise root certificate from Active Directory. The domain CA certificate then appeared in the **Trusted Root Certification Authorities** store as expected.

---

### Result

After restoring DHCP functionality, domain connectivity was re-established and the enterprise certificate authority trust chain propagated correctly to the Windows 11 client.