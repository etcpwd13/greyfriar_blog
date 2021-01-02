---
title: "Writeup: Shield
date: 2021-01-02T03:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - shield
  - update
  - Windows
  - nmap
  - nikto
  
---

Here are notes from the named target:

**Enumeration**

NMAP

```yaml
ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.29 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
 nmap -sC -sV -p$ports 10.10.10.29
 ```
 vuln scan the SQL service:
 
 ```yaml
 sudo nmap -sU --script=ms-sql-info 10.10.10.29
 ```
 
 ```yaml
 PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
3306/tcp open  mysql   MySQL (unauthorized)
```

Take a look at the web server:

Shows only the IIS start page so maybe scan it:

```yaml
nikto -h 10.10.10.29
```

nikto gave no results

