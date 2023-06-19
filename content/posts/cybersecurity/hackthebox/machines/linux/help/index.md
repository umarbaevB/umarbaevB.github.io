---
title: "[HTB] Machine: Help"
description: "[HTB] Machine: Help"
date: 2023-06-16
menu:
  sidebar:
    name: "[HTB] Machine: Help"
    identifier: htb-machine-help
    parent: htb-machines-linux
    weight: 10
hero: images/help.png
tags: ["HTB"]
---

# Help
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV -Pn 10.10.10.121
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-19 18:37 BST
Nmap scan report for 10.10.10.121 (10.10.10.121)
Host is up (0.16s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5bb4d9cdeaf6bbfba8c227ad8d74328 (RSA)
|   256 d5b010507486a39fc5536f3b4a246119 (ECDSA)
|_  256 e21b88d37621d41e38154a8111b79907 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18
|_http-title: Did not follow redirect to http://help.htb/
|_http-server-header: Apache/2.4.18 (Ubuntu)
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.49 seconds

```