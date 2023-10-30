---
title: "[HTB] Machine: Jupiter"
description: "[HTB] Machine: Jupiter"
date: 2023-10-28
menu:
  sidebar:
    name: "[HTB] Machine: Jupiter"
    identifier: htb-machine-jupiter
    parent: htb-machines-linux
    weight: 10
hero: images/jupiter.png
tags: ["HTB"]
---

# Jupiter
## Enumeration
- ```Nmap```
```
└─$ nmap -Pn -p- 10.10.11.216 --min-rate 5000                               
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-30 17:44 GMT
Nmap scan report for 10.10.11.216 (10.10.11.216)
Host is up (0.25s latency).
Not shown: 65521 filtered tcp ports (no-response)
PORT     STATE  SERVICE
22/tcp   open   ssh
23/tcp   closed telnet
25/tcp   closed smtp
80/tcp   open   http
110/tcp  closed pop3
113/tcp  closed ident
139/tcp  closed netbios-ssn
143/tcp  closed imap
445/tcp  closed microsoft-ds
993/tcp  closed imaps
1025/tcp closed NFS-or-IIS
1720/tcp closed h323q931
1723/tcp closed pptp
3389/tcp closed ms-wbt-server
5900/tcp closed vnc

Nmap done: 1 IP address (1 host up) scanned in 36.41 seconds
```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.11.216 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-30 17:56 GMT
Nmap scan report for 10.10.11.216 (10.10.11.216)
Host is up (0.25s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 ac:5b:be:79:2d:c9:7a:00:ed:9a:e6:2b:2d:0e:9b:32 (ECDSA)
|_  256 60:01:d7:db:92:7b:13:f0:ba:20:c6:c9:00:a7:1b:41 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://jupiter.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.80 seconds

```

- Web server

![](./images/1.png)


- `vhosts`
```
└─$ wfuzz -u http://jupiter.htb/ -H 'Host: FUZZ.jupiter.htb' -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --hw 12
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://jupiter.htb/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000001955:   200        211 L    798 W      34390 Ch    "kiosk"  
```

- `gobuster`
```
└─$ gobuster dir -u http://jupiter.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js,php --no-error   
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://jupiter.htb/
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
/.html                (Status: 403) [Size: 162]
/index.html           (Status: 200) [Size: 19680]
/contact.html         (Status: 200) [Size: 10141]
/about.html           (Status: 200) [Size: 12613]
/img                  (Status: 301) [Size: 178] [--> http://jupiter.htb/img/]
/services.html        (Status: 200) [Size: 11969]
/css                  (Status: 301) [Size: 178] [--> http://jupiter.htb/css/]
/portfolio.html       (Status: 200) [Size: 11913]
/js                   (Status: 301) [Size: 178] [--> http://jupiter.htb/js/]

```
```
└─$ gobuster dir -u http://kiosk.jupiter.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js,php --no-error --exclude-length 178 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://kiosk.jupiter.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] Exclude Length:          178
[+] User Agent:              gobuster/3.6
[+] Extensions:              js,php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/login                (Status: 200) [Size: 34390]
/profile              (Status: 302) [Size: 29] [--> /login]
/signup               (Status: 200) [Size: 34390]
/public               (Status: 302) [Size: 31] [--> /public/]
/admin                (Status: 302) [Size: 24] [--> /]
/plugins              (Status: 302) [Size: 24] [--> /]
/live                 (Status: 302) [Size: 24] [--> /]

```

- Web Servers

![](./images/1.png)

![](./images/2.png)
