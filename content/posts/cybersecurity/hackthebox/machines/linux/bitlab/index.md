---
title: "[HTB] Machine: Bitlab"
description: "[HTB] Machine: Bitlab"
date: 2023-10-25
menu:
  sidebar:
    name: "[HTB] Machine: Bitlab"
    identifier: htb-machine-Bitlab
    parent: htb-machines-linux
    weight: 10
hero: images/bitlab.png
tags: ["HTB"]
---

# Bitlab
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.114 --min-rate 5000                 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-25 18:31 BST
Nmap scan report for 10.10.10.114 (10.10.10.114)
Host is up (0.27s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 127.78 seconds
```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.10.114 --min-rate 5000 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-25 18:33 BST
Nmap scan report for 10.10.10.114 (10.10.10.114)
Host is up (0.34s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a2:3b:b0:dd:28:91:bf:e8:f9:30:82:31:23:2f:92:18 (RSA)
|   256 e6:3b:fb:b3:7f:9a:35:a8:bd:d0:27:7b:25:d4:ed:dc (ECDSA)
|_  256 c9:54:3d:91:01:78:03:ab:16:14:6b:cc:f0:b7:3a:55 (ED25519)
80/tcp open  http    nginx
| http-robots.txt: 55 disallowed entries (15 shown)
| / /autocomplete/users /search /api /admin /profile 
| /dashboard /projects/new /groups/new /groups/*/edit /users /help 
|_/s/ /snippets/new /snippets/*/edit
|_http-trane-info: Problem with XML parsing of /evox/about
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://10.10.10.114/users/sign_in
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.12 seconds
```

- Web server

![](./images/1.png)