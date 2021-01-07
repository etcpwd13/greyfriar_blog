---
title: "Writeup: pathfinder"
date: 2021-01-02T03:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - spathfinder
  - update
  - Windows
  - nmap
---

Here are notes from the named target:

**Enumeration**

NMAP

```yaml
ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.30 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
 nmap -sC -sV -p$ports 10.10.10.30
 ```

Above nmap scan did not work and got an error on ports. Tried to do standard all ports "nmap -p- 10.10.10.30" but that said ports may be blocked by firewall 
as suggested started scan with host discovery disabled (-Pn) and it returnes results

```yaml
──(kali㉿kali)-[~]
└─$ nmap -Pn 10.10.10.30           
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-07 11:28 EST
Nmap scan report for 10.10.10.30
Host is up (0.063s latency).
Not shown: 989 filtered ports
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl

Nmap done: 1 IP address (1 host up) scanned in 4.80 seconds
```

Based on port 53 DNS, 88 Kerbos, and 389 ldap, it looks like this is a DC. I want to do a scan with a different tool to verify.



