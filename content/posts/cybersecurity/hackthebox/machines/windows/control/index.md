---
title: "[HTB] Machine: Control"
description: "[HTB] Machine: Control"
date: 2023-09-30
menu:
  sidebar:
    name: "[HTB] Machine: Control"
    identifier: htb-machine-Control
    parent: htb-machines-windows
    weight: 10
hero: images/control.png
tags: ["HTB"]
---

# Control
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.167 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 20:22 BST
Nmap scan report for 10.10.10.167 (10.10.10.167)
Host is up (0.11s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
3306/tcp  open  mysql
49666/tcp open  unknown
49667/tcp open  unknown

```
```
└─$ nmap -Pn -p80,135,3306 -sC -sV 10.10.10.167 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 20:23 BST
Nmap scan report for 10.10.10.167 (10.10.10.167)
Host is up (0.12s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Fidelity
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp  open  msrpc   Microsoft Windows RPC
3306/tcp open  mysql?
| fingerprint-strings: 
|   NULL, SIPOptions: 
|_    Host '10.10.16.9' is not allowed to connect to this MariaDB server
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.94%I=7%D=9/30%Time=6518759C%P=x86_64-pc-linux-gnu%r(NU
SF:LL,49,"E\0\0\x01\xffj\x04Host\x20'10\.10\.16\.9'\x20is\x20not\x20allowe
SF:d\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(SIPOptions,49
SF:,"E\0\0\x01\xffj\x04Host\x20'10\.10\.16\.9'\x20is\x20not\x20allowed\x20
SF:to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.22 seconds

```

- `mysql`
```
└─$ mysql -h 10.10.10.167
ERROR 1130 (HY000): Host '10.10.16.9' is not allowed to connect to this MariaDB server

```

- Web Server

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.167/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50  -x php --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.167/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
2023/09/30 20:24:57 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 3145]
/images               (Status: 301) [Size: 150] [--> http://10.10.10.167/images/]
/about.php            (Status: 200) [Size: 7867]
/uploads              (Status: 301) [Size: 151] [--> http://10.10.10.167/uploads/]
/Images               (Status: 301) [Size: 150] [--> http://10.10.10.167/Images/]
/admin.php            (Status: 200) [Size: 89]
/assets               (Status: 301) [Size: 150] [--> http://10.10.10.167/assets/]
/About.php            (Status: 200) [Size: 7867]
/Index.php            (Status: 200) [Size: 3145]
/database.php         (Status: 200) [Size: 0]
```