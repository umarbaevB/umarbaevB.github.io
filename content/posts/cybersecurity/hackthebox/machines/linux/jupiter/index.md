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
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV 10.10.11.216 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-05 15:10 BST
Nmap scan report for 10.10.11.216 (10.10.11.216)
Host is up (0.17s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 ac5bbe792dc97a00ed9ae62b2d0e9b32 (ECDSA)
|_  256 6001d7db927b13f0ba20c6c900a71b41 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://jupiter.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.19 seconds
```