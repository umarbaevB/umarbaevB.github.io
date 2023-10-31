---
title: "[HTB] Machine: Cache"
description: "[HTB] Machine: Cache"
date: 2023-10-28
menu:
  sidebar:
    name: "[HTB] Machine: Cache"
    identifier: htb-machine-Cache
    parent: htb-machines-linux
    weight: 10
hero: images/cache.png
tags: ["HTB"]
---

# Cache
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.188 --min-rate 5000 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-30 16:53 GMT
Nmap scan report for 10.10.10.188 (10.10.10.188)
Host is up (0.17s latency).
Not shown: 64028 filtered tcp ports (no-response), 1505 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 91.28 seconds

```
```
└─$ nmap -Pn -p22,80,2201 -sC -sV 10.10.10.188
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-31 15:59 GMT
Nmap scan report for 10.10.10.188 (10.10.10.188)
Host is up (0.22s latency).

PORT     STATE  SERVICE VERSION
22/tcp   open   ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a9:2d:b2:a0:c4:57:e7:7c:35:2d:45:4d:db:80:8c:f1 (RSA)
|   256 bc:e4:16:3d:2a:59:a1:3a:6a:09:28:dd:36:10:38:08 (ECDSA)
|_  256 57:d5:47:ee:07:ca:3a:c0:fd:9b:a8:7f:6b:4c:9d:7c (ED25519)
80/tcp   open   http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Cache
|_http-server-header: Apache/2.4.29 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.02 seconds
                                                              
```