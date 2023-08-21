---
title: "[HTB] Machine: GoodGames"
description: "[HTB] Machine: GoodGames"
date: 2023-08-18
menu:
  sidebar:
    name: "[HTB] Machine: GoodGames"
    identifier: htb-machine-GoodGames
    parent: htb-machines-linux
    weight: 10
hero: images/goodgames.png
tags: ["HTB"]
---

# GoodGames
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV -Pn 10.10.11.130 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-21 19:43 BST
Nmap scan report for 10.10.11.130 (10.10.11.130)
Host is up (0.14s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.51
|_http-title: GoodGames | Community and Store
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
Service Info: Host: goodgames.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.73 seconds
```
- `gobuster`
```
└─$ gobuster dir -u http://goodgames.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt --exclude-length 9265
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://goodgames.htb
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] Exclude Length:          9265
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt
[+] Timeout:                 10s
===============================================================
2023/08/21 19:51:04 Starting gobuster in directory enumeration mode
===============================================================
/blog                 (Status: 200) [Size: 44212]
/login                (Status: 200) [Size: 9294]
/profile              (Status: 200) [Size: 9267]
/signup               (Status: 200) [Size: 33387]
/logout               (Status: 302) [Size: 208] [--> http://goodgames.htb/]
/forgot-password      (Status: 200) [Size: 32744]

```
- No `vhosts` were found by `ffuf`

