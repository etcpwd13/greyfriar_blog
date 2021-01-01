---
title: "Writeup: Vaccine"
date: 2021-01-01T11:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - vaccine
  - update
  - Linux
  - fcrackzip
  - ftp
  - crackstation
---

Here are notes from the named target:

**Enumeration**

NMAP

```yaml
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
```

So Linux server target has 3 services 

1. FTP
2. SSH
3. HTTP

First I want to activally connect to the 3 services:

*HTTP*
browser > 10.10.10.46 > MegaCorp Login Page - try previous accounts - 
admin - MEGACORP_4dm1n!!
robert - M3g4C0rpUs3r!

Neither worked!!

See if any info on versions of FTP and SSH:

*FTP*

Tried anonymous login but that failed - )-:
Tried nmap scan for FTP backdoor

```yaml
nmap --script ftp-vsftpd-backdoor -p 21 10.10.10.46                                      1 ⨯ 1 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-01 16:25 EST
Nmap scan report for 10.10.10.46
Host is up (0.065s latency).

PORT   STATE SERVICE
21/tcp open  ftp

```

No known vulns for this version of vsFTP 3.0.3, SSH or Apache versions listed - use searchsploit - also no results

```yaml
┌──(kali㉿kali)-[~]
└─$ searchsploit vsftpd 3.0.3 -w                                                                 1 ⚙
Exploits: No Results
Shellcodes: No Results
Papers: No Results
                                                                                                     
┌──(kali㉿kali)-[~]
└─$ searchsploit vsftpd -w                                                                       1 ⚙
-------------------------------------------------------- --------------------------------------------
 Exploit Title                                          |  URL
-------------------------------------------------------- --------------------------------------------
vsftpd 2.0.5 - 'CWD' (Authenticated) Remote Memory Cons | https://www.exploit-db.com/exploits/5814
vsftpd 2.0.5 - 'deny_file' Option Remote Denial of Serv | https://www.exploit-db.com/exploits/31818
vsftpd 2.0.5 - 'deny_file' Option Remote Denial of Serv | https://www.exploit-db.com/exploits/31819
vsftpd 2.3.2 - Denial of Service                        | https://www.exploit-db.com/exploits/16270
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)  | https://www.exploit-db.com/exploits/17491
-------------------------------------------------------- --------------------------------------------
Shellcodes: No Results
Papers: No Results
                                                                                                     
┌──(kali㉿kali)-[~]
└─$ searchsploit OpenSSH 8.0p1 -w                                                                1 ⚙

Exploits: No Results
Shellcodes: No Results
Papers: No Results
                                                                                                     
┌──(kali㉿kali)-[~]
└─$ searchsploit Apache httpd 2.4.41 -w                                                          1 ⚙

Exploits: No Results
Shellcodes: No Results
Papers: No Results
                                                                                                     
┌──(kali㉿kali)-[~]
└─$                 
```

Take a look at the writeup and see that it references an account and password from Target "Oopsie" the filezilla account mentioned in bugreport 2 or 3 anyway i did not follow that so did not have it in my list of accounts so peeked and here it is:

```yaml
The credentials ftpuser / mc@F1l3ZilL4 can be used to login to the FTP server.
```

***lessons learned*** is do post exploit recon for user names and passwords:

Logged in and used FTP GET command to download the file that was there "backup.zip" and use find command to see where it was downloaded to:

```yaml
find / -type f -name backup.zip 2>/dev/null  
```

cat the file to show it is PK zip and contains a file "index.php" in it abut may hav other files

```yaml
                                                                                                     
┌──(kali㉿kali)-[~]
└─$ cat backup.zip                                                                               3 ⚙
PK     "WCP�A:�"
        index.phpUT     ��7^��7^ux
```

Use "unzip" to decompress but file is pass protected:

```yaml
                                                                        
┌──(kali㉿kali)-[~]
└─$ unzip backup.zip                                                                             3 ⚙
Archive:  backup.zip
[backup.zip] index.php password: 

```

Going to use "fcrackzip"  -> https://installlion.com/kali/kali/main/f/fcrackzip/install/index.html

```yaml
(kali㉿kali)-[~]
└─$ fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt backup.zip                               3 ⚙


PASSWORD FOUND!!!!: pw == 741852963

```

Note: Rockyoy may have to be unpacked on kali as it is a gz by default - This took less than one second to find the password.

Using that password back to unzip the backup.zip file and we get two files:

Cat the index.php and see a MD5 hash for a Admin password:

```yaml
PASSWORD FOUND!!!!: pw == 741852963
                                                                                                     
┌──(kali㉿kali)-[~]
└─$ unzip backup.zip                                                                             3 ⚙
Archive:  backup.zip
[backup.zip] index.php password: 
  inflating: index.php               
  inflating: style.css               
                                                                                                     
┌──(kali㉿kali)-[~]
└─$ cat index.php                                                                                3 ⚙
<!DOCTYPE html>
<?php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
```

used [CrackStation][https://crackstation.net/] on the MD5 hash
```yaml

Hash	Type	Result
2cb42f8734ea607eefed3b70af13bbd3	md5	qwerty789
Color Codes: Green: Exact match, Yellow: Partial match, Red: Not found.

Download CrackStation's Wordlist
```

