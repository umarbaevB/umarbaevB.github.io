---
title: "[HTB] Machine: Optimum"
date: 2023-05-20
description: "[HTB] Machine: Optimum"
menu:
  sidebar:
    name: "[HTB] Machine: Optimum"
    identifier: htb-machine-optimum
    parent: htb-machines-windows
    weight: 10
hero: images/optimum.png
tags: ["HTB"]
---

# Optimum
## Enumeration
- ```Nmap```
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV 10.10.10.8
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-01 19:02 BST
Nmap scan report for 10.10.10.8 (10.10.10.8)
Host is up (0.091s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.60 seconds
```
## Foothold
## User
## Root
