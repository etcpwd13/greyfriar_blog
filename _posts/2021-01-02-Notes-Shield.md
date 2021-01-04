---
title: "Writeup: Shield"
date: 2021-01-02T03:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - shield
  - update
  - Windows
  - nmap
  - nikto
  - wpscan
  - sqlmap
---

Here are notes from the named target:

**Enumeration**

NMAP

```yaml
ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.29 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
 nmap -sC -sV -p$ports 10.10.10.29
 ```
 vuln scan the SQL service:
 
 ```yaml
 sudo nmap -sU --script=ms-sql-info 10.10.10.29
 ```
 
 ```yaml
 PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
3306/tcp open  mysql   MySQL (unauthorized)
```

Take a look at the web server:

Shows only the IIS start page so maybe scan it:

```yaml
nikto -h 10.10.10.29
```

nikto gave no results

*start dirsearch and it finds a site in the wordpress directory*


```yaml
┌──(kali㉿kali)-[~/dirsearch]
└─$ python3 dirsearch.py -e php -u http://10.10.10.29 --exclude-status 403,401

  _|. _ _  _  _  _ _|_    v0.4.1
 (_||| _) (/_(_|| (_| )                                                             
                                                                                    
Extensions: php | HTTP method: GET | Threads: 30 | Wordlist size: 8853

Error Log: /home/kali/dirsearch/logs/errors-21-01-02_04-14-59.log

Target: http://10.10.10.29/                                                         
                                                                                    
Output File: /home/kali/dirsearch/reports/10.10.10.29/_21-01-02_04-15-00.txt

[04:15:00] Starting: 
[04:16:03] 301 -    0B  - /Wordpress/  ->  http://10.10.10.29/wordpress/
[04:18:32] 200 -   24KB - /wordpress/                   
[04:18:32] 200 -    3KB - /wordpress/wp-login.php
                                                             
Task Completed   

```

*Wordpress Site with Login page*

Scan with wpscan:

```yaml
wpscan --url http://10.10.10.29/wordpress --enumerate p
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.10
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Updating the Database ...
[i] Update completed.

[+] URL: http://10.10.10.29/wordpress/ [10.10.10.29]
[+] Started: Sat Jan  2 04:26:20 2021

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Microsoft-IIS/10.0
 |  - X-Powered-By: PHP/7.1.29
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.10.29/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] WordPress readme found: http://10.10.10.29/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.10.29/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.2.1 identified (Insecure, released on 2019-05-21).
 | Found By: Rss Generator (Passive Detection)
 |  - http://10.10.10.29/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=5.2.1</generator>
 |  - http://10.10.10.29/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.2.1</generator>

[i] The main theme could not be detected.

[+] Enumerating Most Popular Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] mesmerize-companion
 | Location: http://10.10.10.29/wordpress/wp-content/plugins/mesmerize-companion/
 | Latest Version: 1.6.120
 | Last Updated: 2020-12-14T17:38:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | The version could not be determined.

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpscan.com/register

```
Password attack with WPScan is possible but time consuming:

```yaml
┌──(kali㉿kali)-[~]
└─$ wpscan --url http://10.10.10.29/wordpress/ --passwords /usr/share/wordlists/rockyou.txt --usernames admin --max-threads 20   
```
Searchsploit  for WordPress version 5.2.1 found no exploits looking in google - nothing in google:

Running SQLMAP:

```yaml
──(kali㉿kali)-[~]
└─$ sqlmap -u ‘http://10.10.10.29/wordpress/index.php/2020/02/10/run-flat-tires/#comment-2’ --crawl=2

```











