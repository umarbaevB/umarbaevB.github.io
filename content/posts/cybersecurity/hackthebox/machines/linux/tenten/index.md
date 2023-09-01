---
title: "[HTB] Machine: Tenten"
description: "[HTB] Machine: Tenten"
date: 2023-09-01
menu:
  sidebar:
    name: "[HTB] Machine: Tenten"
    identifier: htb-machine-Tenten
    parent: htb-machines-linux
    weight: 10
hero: images/tenten.png
tags: ["HTB"]
---

# Tenten
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.10 -T4                            
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-01 17:38 BST
Nmap scan report for 10.10.10.10 (10.10.10.10)
Host is up (0.094s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

```
```
└─$ nmap -Pn -p22,80 -sC -sV  10.10.10.10 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-01 17:54 BST
Nmap scan report for 10.10.10.10 (10.10.10.10)
Host is up (0.10s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ec:f7:9d:38:0c:47:6f:f0:13:0f:b9:3b:d4:d6:e3:11 (RSA)
|   256 cc:fe:2d:e2:7f:ef:4d:41:ae:39:0e:91:ed:7e:9d:e7 (ECDSA)
|_  256 8d:b5:83:18:c0:7c:5d:3d:38:df:4b:e1:a4:82:8a:07 (ED25519)
80/tcp open  http    Apache httpd 2.4.18
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://tenten.htb/
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
- 