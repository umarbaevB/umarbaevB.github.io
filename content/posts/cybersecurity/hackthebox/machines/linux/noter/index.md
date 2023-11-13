---
title: "[HTB] Machine: Noter"
description: "[HTB] Machine: Noter"
date: 2023-11-13
menu:
  sidebar:
    name: "[HTB] Machine: Noter"
    identifier: htb-machine-Noter
    parent: htb-machines-linux
    weight: 10
hero: images/noter.png
tags: ["HTB"]
---

# Noter
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.160 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-13 17:08 GMT
Warning: 10.10.11.160 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.160 (10.10.11.160)
Host is up (0.17s latency).
Not shown: 60517 closed tcp ports (conn-refused), 5015 filtered tcp ports (no-response)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
5000/tcp open  upnp

Nmap done: 1 IP address (1 host up) scanned in 164.93 seconds

```
```
└─$ nmap -Pn -p21,22,5000 -sC -sV 10.10.11.160 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-13 17:14 GMT
Nmap scan report for 10.10.11.160 (10.10.11.160)
Host is up (0.21s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c6:53:c6:2a:e9:28:90:50:4d:0c:8d:64:88:e0:08:4d (RSA)
|   256 5f:12:58:5f:49:7d:f3:6c:bd:9b:25:49:ba:09:cc:43 (ECDSA)
|_  256 f1:6b:00:16:f7:88:ab:00:ce:96:af:a6:7e:b5:a8:39 (ED25519)
5000/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.8.10)
|_http-title: Noter
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.86 seconds

```

- `ftp`
```
└─$ ftp anonymous@10.10.11.160                                                                                                              
Connected to 10.10.11.160.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
530 Login incorrect.
ftp: Login failed
ftp> 

```

- Port `5000`

![](./images/1.png)

