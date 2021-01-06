---
title: "Writeup: Pathfinder"
date: 2021-01-06T11:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - Pathfinder
  - update
  - Windows
 
---

Here are notes from the named target:

**Enumeration**

NMAP

```yaml
ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.30 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
 nmap -sC -sV -p$ports 10.10.10.30
 ```
