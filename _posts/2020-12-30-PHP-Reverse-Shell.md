---
title: "Tool: PHP Reverse Shell"
date: 2020-12-30T10:34:30-04:00
categories:
  - Blog
  - Tool
tags:
  - tool
  - PHP
  - webshell
  - link
link: http://pentestmonkey.net/tools/web-shells/php-reverse-shell
---

This tool is designed for those situations during a pentest where you have upload access to a webserver that’s running PHP.  Upload this script to somewhere in the web root then run it by accessing the appropriate URL in your browser.  The script will open an outbound TCP connection from the webserver to a host and port of your choice.  Bound to this TCP connection will be a shell.

This will be a proper interactive shell in which you can run interective programs like telnet, ssh and su.  It differs from web form-based shell which allow you to send a single command, then return you the output.
