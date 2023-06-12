---
title: "[HTB] Machine: Curling"
description: "[HTB] Machine: Curling"
date: 2023-06-12
menu:
  sidebar:
    name: "[HTB] Machine: Curling"
    identifier: htb-machine-curling
    parent: htb-machines-linux
    weight: 10
hero: images/curling.png
tags: ["HTB"]
---

# Curling
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV 10.10.10.150
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-12 17:50 BST
Nmap scan report for 10.10.10.150 (10.10.10.150)
Host is up (0.18s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8ad169b490203ea7b65401eb68303aca (RSA)
|   256 9f0bc2b20bad8fa14e0bf63379effb43 (ECDSA)
|_  256 c12a3544300c5b566a3fa5cc6466d9a9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Home
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-generator: Joomla! - Open Source Content Management
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.29 seconds
```
- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.150 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50        
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.150
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/06/12 18:07:47 Starting gobuster in directory enumeration mode
===============================================================
/templates            (Status: 301) [Size: 316] [--> http://10.10.10.150/templates/]
/media                (Status: 301) [Size: 312] [--> http://10.10.10.150/media/]
/modules              (Status: 301) [Size: 314] [--> http://10.10.10.150/modules/]
/bin                  (Status: 301) [Size: 310] [--> http://10.10.10.150/bin/]
/plugins              (Status: 301) [Size: 314] [--> http://10.10.10.150/plugins/]
/includes             (Status: 301) [Size: 315] [--> http://10.10.10.150/includes/]
/images               (Status: 301) [Size: 313] [--> http://10.10.10.150/images/]
/language             (Status: 301) [Size: 315] [--> http://10.10.10.150/language/]
/components           (Status: 301) [Size: 317] [--> http://10.10.10.150/components/]
/cache                (Status: 301) [Size: 312] [--> http://10.10.10.150/cache/]
/libraries            (Status: 301) [Size: 316] [--> http://10.10.10.150/libraries/]
/tmp                  (Status: 301) [Size: 310] [--> http://10.10.10.150/tmp/]
/layouts              (Status: 301) [Size: 314] [--> http://10.10.10.150/layouts/]
/administrator        (Status: 301) [Size: 320] [--> http://10.10.10.150/administrator/]
/cli                  (Status: 301) [Size: 310] [--> http://10.10.10.150/cli/]
/server-status        (Status: 403) [Size: 277]
Progress: 220515 / 220561 (99.98%)
===============================================================
2023/06/12 18:16:06 Finished
===============================================================

```