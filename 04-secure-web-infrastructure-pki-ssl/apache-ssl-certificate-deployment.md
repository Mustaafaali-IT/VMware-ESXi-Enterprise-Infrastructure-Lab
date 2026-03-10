# Ubuntu Apache SSL Certificate Configuration

## Summary

This document covers the configuration of SSL encryption for the Apache web server running on `Ubuntu-SRV01` using a certificate issued by the internal Active Directory Certificate Authority.

A certificate will be requested from the domain PKI and installed on the Ubuntu server, allowing the Apache service to serve content over HTTPS instead of standard HTTP. The Apache configuration will then be updated to enable SSL and bind the certificate to the web service.

Once configured, the site will be accessible securely from domain clients using `https://ubuntu-srv01.lab.local`. Because the certificate is issued by the internal Certificate Authority, domain-joined machines will automatically trust the connection.

---

## VMs In Use

| VM Name         | Operating System        | vCPU | RAM | Disk  | Provisioning | Assigned Roles                 |
|-----------------|------------------------|------|-----|-------|-------------|--------------------------------|
| WS2019-DC01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thick       | Active Directory, DNS, AD CS |
| WS2019-FS01     | Windows Server 2019    | 2    | 6GB | 40GB  | Thin        | File Services, DHCP, Print, IIS |
| Ubuntu-SRV01    | Ubuntu Server LTS      | 2    | 4GB | 30GB  | Thin        | Apache Web Server, OpenSSH |
| W11-CL01     | Windows 11 Pro      | 2    | 6GB  | 60GB | Thin        | Client machine - (Lab User & Lab Guest)|

**Note:** WS2019-DC01 was provisioned as Thick by default. This does not negatively impact lab functionality and was intentionally left unchanged.

---

## Prerequisites

### Phase 3 - Server Roles and Application Services

Before configureing cewrtificate base trust within the lab environement you must first confrigure the IIS and Apache web services.

See the [Phase 3 - Server Roles & Application Services](../03-server-roles-and-application-services/README.md) for steps on completion.

---

### DNS Record for Apache Server

Before configuring SSL on the Apache server, a DNS record was created so the service could be accessed using a clean hostname. On **DC01**, I opened the DNS Manager and added a new **A record** in the `lab.local` forward lookup zone.

```
Name: apache
IP Address: 192.168.11.30
```

This created the DNS entry:

```
apache.lab.local
```

which points to the Ubuntu Apache server. After creating the record, name resolution was verified from the Windows client to confirm the hostname correctly resolved to the Ubuntu server's IP address.

---

### Preparing SSH Access

Earlier SSH hardening restricted access to members of the `labsshaccess` Active Directory group. To allow administrative access from the domain controller in the future, the domain **Administrator** account was added to this group in Active Directory.

This allowed SSH authentication using a domain account: `administrator@lab.local`

Once the account was added to the group, the administrator was able to SSH into the Ubuntu server from DC01.

---
## Ubuntu Apache SSL Certificate Configuration

### Generating the Apache Certificate Request

On the Ubuntu server, a directory was created to store SSL materials: `/etc/apache2/ssl`

A new private key and certificate signing request were generated using OpenSSL. This request included a Subject Alternative Name (SAN) so that modern browsers would accept the certificate.

```
sudo openssl req -new -newkey rsa:2048 -nodes -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.csr -subj "/C=CA/ST=Ontario/L=Toronto/O=Lab/OU=IT/CN=apache.lab.local" -addext "subjectAltName=DNS:apache.lab.local"
```

This command generated two files:

- apache.key
- apache.csr

The private key (`apache.key`) remains on the server and is used by Apache to encrypt HTTPS traffic.

The certificate signing request (`apache.csr`) contains the public key and identifying information that will be submitted to the internal certificate authority.

The Common Name and Subject Alternative Name used in the request matched the DNS record created earlier for the server: `apache.lab.local`

---

### Installing the CA Web Enrollment Service

When attempting to submit the certificate request, the CA web portal was initially unavailable. This was because the **Certification Authority Web Enrollment** role service had not yet been installed on the domain controller.

The role service was installed through:

- Server Manager
- Add Roles and Features
- Active Directory Certificate Services
- Certification Authority Web Enrollment

After installation, the web enrollment interface became available at: `http://ws2019-dc01/certsrv`

---

### Submitting the Certificate Request

Because the Ubuntu server was accessed through the ESXi console, copying the CSR directly was difficult. To simplify this, the server was accessed remotely through SSH using the domain administrator account: `administrator@lab.local`

The CSR was displayed using: `cat /etc/apache2/ssl/apache.csr`

The contents of the request were then copied and submitted to the CA web portal.

Within the enrollment page the following options were selected:

**Request Type:**
 
- Advanced Certificate Request
- Submit a certificate request using a base-64 encoded certificate request

**Certificate Template:** Web server

Because the Subject Alternative Name was already embedded in the CSR, no additional attributes were required.

The certificate was then issued and downloaded in **Base64 encoded format**.

---

### Granting Administrative Privileges to Domain Admins

Although the domain administrator account was able to authenticate through SSH, it did not initially have administrative privileges on the Linux system.

To allow domain administrators to perform system administration tasks, the account was granted sudo privileges by modifying the sudoers configuration.

This allows domain administrators to elevate privileges when managing the server remotely.

---

### Installing the Issued Certificate on Ubuntu

The certificate downloaded from the CA was saved on the domain controller as: `apache.crt`

Because file transfer through SCP was problematic due to domain-based authentication syntax, the certificate was manually copied.

Using the SSH session, a new certificate file was created on the server: `/etc/apache2/ssl/apache.crt`

The contents of the downloaded `.crt` file were copied from the Windows system and pasted into this file.
---

## Apache SSL Configuration

With the certificate and private key now present on the Ubuntu server, Apache was configured to serve the website over HTTPS.

First, the Apache SSL module was enabled: `sudo a2enmod ssl`

This module adds TLS/SSL support to the Apache web server, allowing it to handle encrypted HTTPS traffic.

Next, the default SSL virtual host configuration was enabled: `sudo a2ensite default-ssl`

This activates Apache’s preconfigured HTTPS site definition located at: `/etc/apache2/sites-available/default-ssl.conf`

The configuration file was then edited to reference the certificate and private key issued by the internal certificate authority: `sudo nano /etc/apache2/sites-available/default-ssl.conf`

Within the configuration file, the certificate paths were updated to:

```
SSLCertificateFile /etc/apache2/ssl/apache.crt
SSLCertificateKeyFile /etc/apache2/ssl/apache.key
```

These directives instruct Apache to use the certificate issued by the internal CA along with the private key generated earlier.

Once the configuration was updated, Apache was restarted to apply the changes.

---

## Verification

After configuring Apache, several checks were performed to confirm that HTTPS was functioning correctly.

First, the Apache service was verified to ensure it was running without errors.

Next, the server was checked to confirm that Apache was actively listening on the HTTPS port 443.

Finally, HTTPS connectivity was tested from the Windows client by navigating to: `https://apache.lab.local`

The website loaded successfully and the browser displayed a secure connection indicator. The certificate details confirmed that the site was secured using a certificate issued by the internal certificate authority.

---

## Results

At this stage of the lab environment, a fully functional internal Public Key Infrastructure has been implemented.

The following components are now operational:

- Active Directory Certificate Authority issuing certificates
- Windows IIS web server secured with HTTPS
- Ubuntu Apache web server secured with HTTPS
- Domain clients automatically trusting the internal CA
- Secure encrypted web communication within the lab environment

This demonstrates the complete lifecycle of internal certificate management, from certificate request generation to issuance and deployment across both Windows and Linux systems.

---

## Next Steps

Further enhancements to the lab environment will continue to build on the existing infrastructure. They are to be determined.















 











