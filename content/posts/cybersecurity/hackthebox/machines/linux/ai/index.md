---
title: "[HTB] Machine: AI"
description: "[HTB] Machine: AI"
date: 2023-10-25
menu:
  sidebar:
    name: "[HTB] Machine: AI"
    identifier: htb-machine-AI
    parent: htb-machines-linux
    weight: 10
hero: images/ai.png
tags: ["HTB"]
---

# AI
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.163 --min-rate 10000                                                                                         
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-25 19:19 BST
Nmap scan report for 10.10.10.163 (10.10.10.163)
Host is up (0.18s latency).
Not shown: 65443 filtered tcp ports (no-response), 90 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 24.11 seconds

```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.10.163 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-25 19:29 BST
Nmap scan report for 10.10.10.163 (10.10.10.163)
Host is up (0.77s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6d:16:f4:32:eb:46:ca:37:04:d2:a5:aa:74:ed:ab:fc (RSA)
|   256 78:29:78:d9:f5:43:d1:cf:a0:03:55:b1:da:9e:51:b6 (ECDSA)
|_  256 85:2e:7d:66:30:a6:6e:30:04:82:c1:ae:ba:a4:99:bd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Hello AI!
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.03 seconds
                                                                 
```

- Web Server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.163/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,html -t 50 -k 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.163/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 277]
/.php                 (Status: 403) [Size: 277]
/images               (Status: 301) [Size: 313] [--> http://10.10.10.163/images/]
/contact.php          (Status: 200) [Size: 37371]
/index.php            (Status: 200) [Size: 37347]
/about.php            (Status: 200) [Size: 37503]
/uploads              (Status: 301) [Size: 314] [--> http://10.10.10.163/uploads/]
/db.php               (Status: 200) [Size: 0]
/intelligence.php     (Status: 200) [Size: 38674]
/ai.php               (Status: 200) [Size: 37569]

```

## Foothold