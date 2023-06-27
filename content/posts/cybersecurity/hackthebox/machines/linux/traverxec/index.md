---
title: "[HTB] Machine: Traverxec"
description: "[HTB] Machine: Traverxec"
date: 2023-06-26
menu:
  sidebar:
    name: "[HTB] Machine: Traverxec"
    identifier: htb-machine-traverxec
    parent: htb-machines-linux
    weight: 10
hero: images/traverxec.png
tags: ["HTB"]
---

# Traverxec
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV -Pn 10.10.10.165
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-27 19:35 BST
Nmap scan report for 10.10.10.165 (10.10.10.165)
Host is up (0.096s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa99a81668cd41ccf96c8401c759095c (RSA)
|   256 93dd1a23eed71f086b58470973a388cc (ECDSA)
|_  256 9dd6621e7afb8f5692e637f110db9bce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.10 seconds
```
- Web server

![](./images/1.png)

- `gobuster`

```
└─$ gobuster dir -u http://10.10.10.165 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.165
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/06/27 19:39:09 Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 314] [--> http://10.10.10.165/img/]
/icons                (Status: 301) [Size: 314] [--> http://10.10.10.165/icons/]
/css                  (Status: 301) [Size: 314] [--> http://10.10.10.165/css/]
/lib                  (Status: 301) [Size: 314] [--> http://10.10.10.165/lib/]
/js                   (Status: 301) [Size: 314] [--> http://10.10.10.165/js/]
```

## Foothold
- From `nmap` results we see `nostromo 1.9.6`
  - Check `searchsploit`

![](./images/2.png)


## Root