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
  - domain
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

Having to dig up more info on enumerating a DC looked to [Active Directory Reconnaissence][AD-Recon[

As their were a bunch of ports open I probed thos ports with the (-sV)

```yaml
┌──(kali㉿kali)-[~]
└─$ nmap -sT -Pn -n --open 10.10.10.30 -sV -p53,88,135,139,389,445,464,593,636,3268,3269,3389
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-07 11:45 EST
Nmap scan report for 10.10.10.30
Host is up (0.065s latency).
Not shown: 1 filtered port
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-01-08 00:53:46Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: PATHFINDER; OS: Windows; CPE: cpe:/o:microsoft:windows

```

The LDAP service is running on a couple of ports. the spec for LDAP say it has to provide some info unauthenticated so run this nmap query:

```yaml
nmap -sT -Pn -n --open 10.10.10.30 -p389 --script ldap-rootdse
```








[AD-Recon]: https://exploit.ph/active-directory-recon-1.html

