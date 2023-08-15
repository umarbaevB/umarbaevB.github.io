---
title: "[HTB] Machine: Driver"
description: "[HTB] Machine: Driver"
date: 2023-08-15
menu:
  sidebar:
    name: "[HTB] Machine: Driver"
    identifier: htb-machine-Driver
    parent: htb-machines-windows
    weight: 10
hero: images/driver.png
tags: ["HTB"]
---

# Driver
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV -Pn 10.10.11.104 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-15 20:03 BST
Nmap scan report for 10.10.11.104 (10.10.11.104)
Host is up (0.14s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 53:ed:44:40:11:6e:8b:da:69:85:79:c0:81:f2:3a:12 (RSA)
|   256 bc:54:20:ac:17:23:bb:50:20:f4:e1:6e:62:0f:01:b5 (ECDSA)
|_  256 33:c1:89:ea:59:73:b1:78:84:38:a4:21:10:0c:91:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-title: Previse Login
|_Requested resource was login.php
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.90 seconds

```