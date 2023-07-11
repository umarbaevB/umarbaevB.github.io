---
title: "[HTB] Machine: Luanne"
description: "[HTB] Machine: Luanne"
date: 2023-07-09
menu:
  sidebar:
    name: "[HTB] Machine: Luanne"
    identifier: htb-machine-luanne
    parent: htb-machines-linux
    weight: 10
hero: images/luanne.png
tags: ["HTB"]
---

# Luanne
## Enumeration
- ```Nmap```
```
└─$ nmap -sC -sV -Pn 10.10.10.218 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-11 17:33 BST
Nmap scan report for 10.10.10.218 (10.10.10.218)
Host is up (0.15s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.0 (NetBSD 20190418-hpn13v14-lpk; protocol 2.0)
| ssh-hostkey: 
|   3072 20977f6c4a6e5d20cffda3aaa90d37db (RSA)
|   521 35c329e187706d7374b2a9a204a96669 (ECDSA)
|_  256 b3bd316dcc226b18ed2766b4a72ae4a5 (ED25519)
80/tcp   open  http    nginx 1.19.0
| http-robots.txt: 1 disallowed entry 
|_/weather
|_http-server-header: nginx/1.19.0
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=.
|_http-title: 401 Unauthorized
9001/tcp open  http    Medusa httpd 1.12 (Supervisor process manager)
|_http-title: Error response
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=default
|_http-server-header: Medusa/1.12
Service Info: OS: NetBSD; CPE: cpe:/o:netbsd:netbsd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 204.16 seconds
```
- Web server

![](./images/1.png)
![](./images/3.png)


## Foothold/User
- If we visit port `9001`, we have login page but it shows nothing else
  - `nmap` tells us some information about `Supervisor process manager` and `realm`
  - Googling reveals the [configuration](http://supervisord.org/configuration.html)
  - The creds work `user:123`

![](./images/2.png)
![](./images/4.png)



## Root