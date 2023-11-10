---
title: "[HTB] Machine: Unicode"
description: "[HTB] Machine: Unicode"
date: 2023-11-06
menu:
  sidebar:
    name: "[HTB] Machine: Unicode"
    identifier: htb-machine-Unicode
    parent: htb-machines-linux
    weight: 10
hero: images/unicode.png
tags: ["HTB"]
---

# Unicode
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.126 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 18:30 GMT
Warning: 10.10.11.126 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.126 (10.10.11.126)
Host is up (0.17s latency).
Not shown: 65167 closed tcp ports (conn-refused), 366 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 92.75 seconds

```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.11.126 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 18:32 GMT
Nmap scan report for 10.10.11.126 (10.10.11.126)
Host is up (0.19s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fd:a0:f7:93:9e:d3:cc:bd:c2:3c:7f:92:35:70:d7:77 (RSA)
|   256 8b:b6:98:2d:fa:00:e5:e2:9c:8f:af:0f:44:99:03:b1 (ECDSA)
|_  256 c9:89:27:3e:91:cb:51:27:6f:39:89:36:10:41:df:7c (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: 503
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.04 seconds

```

- Web Server

![](./images/1.png)

- `gobuster`

```
└─$ gobuster dir -u http://10.10.11.126/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -x txt,html,js,php --no-error --exclude-length 9294 -b 404,503
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.126/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404,503
[+] Exclude Length:          9294
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,js,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/register             (Status: 308) [Size: 264] [--> http://10.10.11.126/register/]
/logout               (Status: 308) [Size: 260] [--> http://10.10.11.126/logout/]
/error                (Status: 308) [Size: 258] [--> http://10.10.11.126/error/]
/checkout             (Status: 308) [Size: 264] [--> http://10.10.11.126/checkout/]
/dashboard            (Status: 308) [Size: 266] [--> http://10.10.11.126/dashboard/]
/pricing              (Status: 308) [Size: 262] [--> http://10.10.11.126/pricing/]
/filenotfound         (Status: 308) [Size: 272] [--> http://10.10.11.126/filenotfound/]
```
```

```