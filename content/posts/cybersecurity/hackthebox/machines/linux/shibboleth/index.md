---
title: "[HTB] Machine: Shibboleth"
description: "[HTB] Machine: Shibboleth"
date: 2023-11-06
menu:
  sidebar:
    name: "[HTB] Machine: Shibboleth"
    identifier: htb-machine-Shibboleth
    parent: htb-machines-linux
    weight: 10
hero: images/shibboleth.png
tags: ["HTB"]
---

# Shibboleth
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.124 --min-rate 1000            
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-07 16:58 GMT
Warning: 10.10.11.124 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.124 (10.10.11.124)
Host is up (0.17s latency).
Not shown: 65181 closed tcp ports (conn-refused), 353 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 87.95 seconds

```
```
└─$ nmap -Pn -p80 -sC -sV 10.10.11.124 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-07 16:58 GMT
Nmap scan report for 10.10.11.124 (10.10.11.124)
Host is up (0.17s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41
|_http-title: Did not follow redirect to http://shibboleth.htb/
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: Host: shibboleth.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.06 seconds
                                                              
```

- Web server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://shibboleth.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js,php --no-error

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://shibboleth.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,js,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 279]
/.html                (Status: 403) [Size: 279]
/index.html           (Status: 200) [Size: 59474]
/blog.html            (Status: 200) [Size: 19196]
/assets               (Status: 301) [Size: 317] [--> http://shibboleth.htb/assets/]
/forms                (Status: 301) [Size: 316] [--> http://shibboleth.htb/forms/]
/changelog.txt        (Status: 200) [Size: 499]

```

- `vhosts`
```
└─$ wfuzz -u http://shibboleth.htb/ -H 'Host: FUZZ.shibboleth.htb' -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --hw 26
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://shibboleth.htb/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000099:   200        29 L     219 W      3684 Ch     "monitor"
000000346:   200        29 L     219 W      3684 Ch     "monitoring"
000000390:   200        29 L     219 W      3684 Ch     "zabbix"
```