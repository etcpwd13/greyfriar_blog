---
title: "Tool: Known Passwords cheatsheet"
date: 2021-01-02T03:34:30-04:00
categories:
  - Blog
  - Tool
tags:
  - tool
  - cheatsheet
  - password

---

```yaml
admin : P@s5w0rd!
sql_svc : M3g4c0rp123
administrator : MEGACORP_4dm1n!!
robert : Writeup: Vaccine
 8 minute read
Here are notes from the named target:

Enumeration

NMAP

ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.46 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
 nmap -sC -sV -p$ports 10.10.10.46
 
 tarting Nmap 7.91 ( https://nmap.org ) at 2021-01-01 15:57 EST
Nmap scan report for 10.10.10.46
Host is up (0.13s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6build1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)
|   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)
|_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: MegaCorp Login
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.60 seconds
So Linux server target has 3 services

FTP
SSH (We will use this for shell once we get a user and pass)
HTTP
First I want to activally connect to the 3 services:

HTTP browser > 10.10.10.46 > MegaCorp Login Page - try previous accounts - admin - MEGACORP_4dm1n!! robert - M3g4C0rpUs3r!

