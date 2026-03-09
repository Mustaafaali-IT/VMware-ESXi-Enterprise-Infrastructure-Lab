# AD Certificate Authority Configuration

## Summary

This document covers the deployment and configuration of **Active Directory Certificate Services (AD CS)** within the lab environment to establish an internal Public Key Infrastructure (PKI).

An **Enterprise Root Certificate Authority** will be installed on the domain controller to allow trusted certificates to be issued to machines and services inside the `lab.local` domain. This mirrors how organizations manage internal certificates used to secure web services, authenticate devices, and enable encrypted communication across their infrastructure.

Once the Certificate Authority is operational, certificates can be requested and deployed to services such as IIS and Apache. Because the CA is integrated with Active Directory, domain-joined machines will automatically trust certificates issued by this authority.

This setup prepares the lab environment for enabling **HTTPS and SSL/TLS encryption** on internal web services in the next steps of Phase 4.

---

## VMs In Use

| VM Name         | Operating System        | vCPU | RAM | Disk  | Provisioning | Assigned Roles                 |
|-----------------|------------------------|------|-----|-------|-------------|--------------------------------|
| WS2019-DC01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thick       | Active Directory, DNS, AD CS |
| WS2019-FS01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thin        | File Services, DHCP, Print, IIS |
| W11-CL01     | Windows 11 Pro      | 2    | 6GB  | 60GB | Thin        | Client machine - (Lab User & Lab Guest)|

**Note:** WS2019-DC01 was provisioned as Thick by default. This does not negatively impact lab functionality and was intentionally left unchanged.

---

## Prerequisites

### Phase 3 - Server Roles and Application Services

Before configureing cewrtificate base trust within the lab environement you must first confrigure the IIS and Apache web services.

See the [Phase 3 - Server Roles & Application Services](../03-server-roles-and-application-services/README.md) for steps on completion.

---

## AD CS 

### Configuration

The AD CS role was installed on **DC01** using Server Manager through the standard *Add Roles and Features* process. Only the **Certification Authority** role service was required for this environment.

After the role installation completed, the certificate authority was configured as an **Enterprise Root CA**. Choosing an Enterprise CA integrates the service with Active Directory so domain machines automatically trust certificates issued by it. DC01 was configured as the **Root CA**, meaning it acts as the top-level certificate authority for the entire lab environment.

A new private key was generated for the CA and the default cryptographic settings were kept. The automatically generated CA name and default database locations were also left unchanged since they are sufficient for a lab setup.

Once the configuration finished, the Certification Authority management console confirmed the CA was running. The standard CA containers such as *Issued Certificates*, *Pending Requests*, *Failed Requests*, and *Certificate Templates* were present.

---

### Verification

Because the certificate authority is integrated with Active Directory, trust is automatically distributed to domain joined machines. To verify this, I checked the certificate store on the Windows 11 client and confirmed that the root certificate for the domain CA appeared in the **Trusted Root Certification Authorities** store.

---

### Result

The environment now has a functioning **internal certificate authority** running on DC01. This allows the lab to issue trusted certificates for internal services such as IIS and Apache. Domain computers trust the CA automatically, which will allow internal websites to be accessed over HTTPS without browser warnings.

---

## Troubleshooting

During verification of the certificate authority from the Windows 11 client, the CA certificate initially did not appear in the **Trusted Root Certification Authorities** store. This turned out to be a network issue rather than a PKI problem.

Running `ipconfig /all` on the client showed an address in the **169.254.x.x range**, which indicates Windows could not reach a DHCP server and automatically assigned itself an APIPA address. Because the client was not receiving a valid IP configuration, it could not reach the domain controller, causing Group Policy updates to fail and preventing the enterprise CA certificate from being distributed.

After checking the DHCP server on the file server (FS01), the DHCP service was restarted. Once DHCP began responding again, the Windows 11 client successfully obtained a valid IP address from the scope. After renewing the lease and refreshing Group Policy, the domain CA certificate appeared in the trusted root store as expected.

---

## Next Steps

With the internal Certificate Authority operational, the next step is to begin [securing web services using certificates issued by the domain PKI.](iis-ssl-certificate-deployment.md)

The first service to be secured will be the internal IIS web server hosted on `WS2019-FS01`. A certificate will be requested from the Certificate Authority and bound to the **LAB-Intranet** site so that the service can be accessed securely using HTTPS through `https://intranet.lab.local`.

After IIS is secured, a certificate will also be issued for the Apache web server running on the Ubuntu machine to enable encrypted HTTPS communication across both Windows and Linux hosted services within the lab environment.