---
title: "[HTB] Machine: Haircut"
description: "[HTB] Machine: Haircut"
date: 2023-09-30
menu:
  sidebar:
    name: "[HTB] Machine: Haircut"
    identifier: htb-machine-Haircut
    parent: htb-machines-linux
    weight: 10
hero: images/haircut.png
tags: ["HTB"]
---

# Haircut
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.24 --min-rate 10000           
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:40 BST
Nmap scan report for 10.10.10.24 (10.10.10.24)
Host is up (0.11s latency).
Not shown: 65507 filtered tcp ports (no-response), 26 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 14.92 seconds

```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.10.24 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:41 BST
Nmap scan report for 10.10.10.24 (10.10.10.24)
Host is up (0.14s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e9:75:c1:e4:b3:63:3c:93:f2:c6:18:08:36:48:ce:36 (RSA)
|   256 87:00:ab:a9:8f:6f:4b:ba:fb:c6:7a:55:a8:60:b2:68 (ECDSA)
|_  256 b6:1b:5c:a9:26:5c:dc:61:b7:75:90:6c:88:51:6e:54 (ED25519)
80/tcp open  http    nginx 1.10.0 (Ubuntu)
|_http-title:  HTB Hairdresser 
|_http-server-header: nginx/1.10.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.13 seconds
```