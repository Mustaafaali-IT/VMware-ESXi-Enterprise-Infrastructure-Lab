# IIS SSL Certificate Deployment

## Summary

This document covers the deployment of an SSL certificate for the **LAB-Intranet** site hosted on `WS2019-FS01` using the internal Certificate Authority configured in the lab environment.

A certificate is requested from the Active Directory Certificate Authority and installed on the IIS server. The certificate is then bound to the **LAB-Intranet** website so the service can be accessed securely over HTTPS using `https://intranet.lab.local`.

This configuration demonstrates how internal web services are secured in enterprise environments using certificates issued by an internal PKI, allowing domain-joined clients to trust the certificate automatically without relying on external certificate providers.

---

## VMs In Use

| VM Name         | Operating System        | vCPU | RAM | Disk  | Provisioning | Assigned Roles                 |
|-----------------|------------------------|------|-----|-------|-------------|--------------------------------|
| WS2019-DC01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thick       | Active Directory, DNS, AD CS |
| WS2019-FS01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thin        | File Services, DHCP, Print, IIS(SSL) |
| W11-CL01     | Windows 11 Pro      | 2    | 6GB  | 60GB | Thin        | Client machine - (Lab User & Lab Guest)|

**Note:** WS2019-DC01 was provisioned as Thick by default. This does not negatively impact lab functionality and was intentionally left unchanged.

---

## Prerequisites

### Phase 3 - Server Roles and Application Services

Before configureing cewrtificate base trust within the lab environement you must first confrigure the IIS and Apache web services.

See the [Phase 3 - Server Roles & Application Services](../03-server-roles-and-application-services/README.md) for steps on completion.

---

## IIS SSL Certificate Deployment

## Certificate Template Permissions (DC01)

Before requesting a certificate, the Web Server certificate template needed to allow domain machines to enroll for it.

On **DC01**, I opened the Certificate Templates console:

```
certtmpl.msc
```

I located the **Web Server** template and opened its properties. Under the **Security** tab I added the following group:

```
Domain Computers
```

Then I enabled the permissions:

```
Read
Enroll
```

This allows domain servers such as FS01 to request a certificate using the Web Server template from the internal CA.

---

## Requesting the Certificate (FS01)

On the IIS server (**FS01**) I opened the local machine certificate store:

```
certlm.msc
```

Then navigated to:

```
Certificates (Local Computer)
Personal
Request New Certificate
```

Using the **Active Directory Enrollment Policy**, I selected the **Web Server** template and configured the certificate properties.

### Subject Name

```
Common Name: intranet.lab.local
```

This matches the internal DNS name of the IIS site.

### Subject Alternative Name (SAN)

```
DNS: intranet.lab.local
```

Modern browsers require a Subject Alternative Name field, so this was added to ensure the certificate would be trusted by the client browser.

### Friendly Name

```
IIS-Intranet-Cert
```

This simply helps identify the certificate later when selecting it inside IIS.

The certificate was then successfully enrolled and installed into the **Personal certificate store** of the FS01 server.

---

## Configuring HTTPS in IIS

After the certificate was issued, I configured IIS to use it for HTTPS.

In **IIS Manager** on FS01:

```
Sites
LAB-intranet
Bindings
```

I added an HTTPS binding with the following settings:

```
Type: https
Port: 443
Host Name: intranet.lab.local
IP Address: unassinged
SSL Certificate: IIS-Intranet-Cert
```

The binding was applied and IIS was restarted to ensure the configuration was active.

---

## Testing Secure Access

Testing was performed from the **Windows 11 domain client**.

The intranet site was opened using:

```
https://intranet.lab.local
```

The browser successfully established a secure connection and displayed the lock icon. The certificate was shown as being issued by the internal CA:

```
lab-WS2019-DC01-CA
```

Because the client machine is domain joined, it automatically trusts the Enterprise Root CA, which prevented any certificate trust warnings.

---

## Results

The IIS intranet website is now secured using HTTPS with a certificate issued by the internal Active Directory Certificate Authority.

Key outcomes of this configuration include:

- Internal SSL certificates are issued and managed by the domain CA
- Domain clients automatically trust the certificate authority
- The IIS intranet site now communicates securely over port 443
- Users accessing the site from domain computers do not receive browser security warnings

This setup simulates how internal web services are secured within enterprise networks using Active Directory PKI.

---

## Troubleshooting

During the process several issues were encountered and resolved.

Initially, the certificate request on FS01 failed because the **Web Server template did not allow enrollment for the requesting machine**. This was corrected by modifying the template permissions in the Certificate Templates console and allowing **Domain Computers** to enroll.

Another issue occurred where the browser displayed the error:

```
certificate does not specify Subject Alternative Names
```

This happened because modern browsers ignore the certificate's Common Name when SAN is missing. Reissuing the certificate and adding:

```
DNS: intranet.lab.local
```

to the Subject Alternative Name field resolved the problem.

There was also a temporary network connectivity issue where the Windows client lost communication with the domain controller. After restarting the server services and verifying DHCP authorization, the client regained proper network configuration and Group Policy updates resumed.

Once the correct certificate template permissions and SAN configuration were applied, the certificate enrollment and IIS HTTPS configuration worked as expected.

---

## Next Steps

With IIS now secured using HTTPS, the next step is to [configure SSL for the Apache web server running on the Ubuntu server.](apache-ssl-certificate-deployment.md)

The plan is to request another certificate from the internal CA for the Apache host and configure Apache to serve the intranet site over HTTPS. This will allow both the Windows IIS server and the Linux Apache server to use certificates issued by the same internal PKI infrastructure.

Completing this step will demonstrate how Active Directory Certificate Services can provide trusted certificates for both Windows and Linux web services inside the same enterprise environment.