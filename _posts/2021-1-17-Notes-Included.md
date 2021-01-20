---
title: "Writeup: Included"
date: 2021-01-17T19:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - included
  - Linux
  - update
  - nmap
  - passwords
  - lxc
  - dirsearch
  - webshell
  - LFI
  - TFTP
  - nc
  - strings
  - chmod
  - upgrade shell
  - nmap
---

#Here are notes from the named target:
Target is Linux
Host name Included
IP 10.10.10.55

#Enumeration
Start with simple nmap

```yaml
┌──(root💀kali)-[~]
└─# nmap -A -v 10.10.10.55
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-19 20:42 EST
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 20:42
Completed NSE at 20:42, 0.00s elapsed
Initiating NSE at 20:42
Completed NSE at 20:42, 0.00s elapsed
Initiating NSE at 20:42
Completed NSE at 20:42, 0.00s elapsed
Initiating Ping Scan at 20:42
Scanning 10.10.10.55 [4 ports]
Completed Ping Scan at 20:42, 0.10s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 20:42
Completed Parallel DNS resolution of 1 host. at 20:42, 0.02s elapsed
Initiating SYN Stealth Scan at 20:42
Scanning 10.10.10.55 [1000 ports]
*Discovered open port 80/tcp on 10.10.10.55

```

only port found open was port 80 http so going to try UDP also


