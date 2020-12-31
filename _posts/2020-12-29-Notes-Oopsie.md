---
title: "Writeup: Oopsie"
date: 2020-12-29T9:34:30-04:00
categories:
  - Blog
  - Notes
tags:
  - oopsie
  - update
  - nmap
  - pwd
  - BURP
---

Here are notes from the named target:

*Enumeration* - Ports 445 and 1433 are open, which are associated with file sharing (SMB) and SQL Server.
Linux - 10.10.10.28
```yaml
ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.27 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
 nmap -sC -sV -p$ports 10.10.10.27 
```

Results:
```yaml
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Welcome
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Two Ports open SSH and Web

*Enumerate* the Web port 80 - MegaCorpAutomotive

Using a web browser connect to the IP and look around. None of the menu links work but at the bottom of the home page it speaks about login but no page for that. Use BURP proxy to take a look at web traffic.

**BURP**

Using BURP and its uilt in browser no need to configure external browser.
  open web site and in BURP click :Forward" button on the Proxy tab until the page opens up:
  Switch to the :Target" tab and you see a folder tree under the Web IP for "CDN-CGI\login" add that to url path in browser and hit enter:
  back in BURP on the Proxy Tab click Forward to get to the Login Screen:
  
From the "ARCHTYPE" box we learned that their was an Administrator/Admin account that used the Password: MEGACORP_4dm1n!!

Using that Admin user name and Password from ARCHETYPE works and we get the admin page

```yaml
http://10.10.10.28/cdn-cgi/login/admin.php
```

Look aroun that page:

  

