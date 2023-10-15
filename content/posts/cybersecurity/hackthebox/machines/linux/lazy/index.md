---
title: "[HTB] Machine: Lazy"
description: "[HTB] Machine: Lazy"
date: 2023-09-30
menu:
  sidebar:
    name: "[HTB] Machine: Lazy"
    identifier: htb-machine-Lazy
    parent: htb-machines-linux
    weight: 10
hero: images/lazy.png
tags: ["HTB"]
---

# Lazy
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.18 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:57 BST
Nmap scan report for 10.10.10.18 (10.10.10.18)
Host is up (0.10s latency).
Not shown: 64514 filtered tcp ports (no-response), 1019 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 47.01 seconds
```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.10.18 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:54 BST
Nmap scan report for 10.10.10.18 (10.10.10.18)
Host is up (0.53s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 e1:92:1b:48:f8:9b:63:96:d4:e5:7a:40:5f:a4:c8:33 (DSA)
|   2048 af:a0:0f:26:cd:1a:b5:1f:a7:ec:40:94:ef:3c:81:5f (RSA)
|   256 11:a3:2f:25:73:67:af:70:18:56:fe:a2:e3:54:81:e8 (ECDSA)
|_  256 96:81:9c:f4:b7:bc:1a:73:05:ea:ba:41:35:a4:66:b7 (ED25519)
80/tcp open  http    Apache/2.4.7 (Ubuntu)
|_http-title: CompanyDev
|_http-server-header: Apache/2.4.7 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.72 seconds

```
- `feroxbuster`
```
└─$ feroxbuster -u http://10.10.10.18 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -k

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.10.18
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔓  Insecure              │ true
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET       10l       30w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      310c http://10.10.10.18/images => http://10.10.10.18/images/
301      GET        9l       28w      307c http://10.10.10.18/css => http://10.10.10.18/css/
200      GET       37l      233w    22523c http://10.10.10.18/images/banner.png
301      GET        9l       28w      311c http://10.10.10.18/classes => http://10.10.10.18/classes/
200      GET        0l        0w        0c http://10.10.10.18/classes/phpfix.php
200      GET        0l        0w        0c http://10.10.10.18/classes/user.php
200      GET        0l        0w        0c http://10.10.10.18/classes/db.php
302      GET        0l        0w        0c http://10.10.10.18/classes/auth.php => http://10.10.10.18/admin/login.php
200      GET        4l        7w       57c http://10.10.10.18/css/companydev.css
200      GET       60l       95w     1592c http://10.10.10.18/register.php
200      GET       58l       97w     1548c http://10.10.10.18/login.php
200      GET        5l       16w     1027c http://10.10.10.18/images/key.png
200      GET       92l      538w    68572c http://10.10.10.18/images/maxresdefault.jpg
200      GET     4656l     9502w    81933c http://10.10.10.18/css/bootstrap.css
200      GET       41l       77w     1117c http://10.10.10.18/

```