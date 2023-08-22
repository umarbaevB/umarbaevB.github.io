---
title: "[HTB] Machine: Agile"
description: "[HTB] Machine: Agile"
date: 2023-08-05
menu:
  sidebar:
    name: "[HTB] Machine: Agile"
    identifier: htb-machine-Agile
    parent: htb-machines-linux
    weight: 10
hero: images/Agile.png
tags: ["HTB"]
---

# Agile
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p22,80,7352,12249,13006,15172,26869,28151,34525,34972,37863,40486,49272,63692 -sC -sV 10.10.11.203 -T4
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-20 15:31 EDT
Nmap scan report for 10.10.11.203 (10.10.11.203)
Host is up (0.18s latency).

PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f4bcee21d71f1aa26572212d5ba6f700 (ECDSA)
|_  256 65c1480d88cbb975a02ca5e6377e5106 (ED25519)
80/tcp    open   http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://superpass.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
7352/tcp  closed swx
12249/tcp closed unknown
13006/tcp closed unknown
15172/tcp closed unknown
26869/tcp closed unknown
28151/tcp closed unknown
34525/tcp closed unknown
34972/tcp closed unknown
37863/tcp closed unknown
40486/tcp closed unknown
49272/tcp closed unknown
63692/tcp closed unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.44 seconds
```
- `gobuster`
```
└─$ gobuster dir -u http://superpass.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://superpass.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/05/20 15:33:55 Starting gobuster in directory enumeration mode
===============================================================
/download             (Status: 302) [Size: 249] [--> /account/login?next=%2Fdownload]
/static               (Status: 301) [Size: 178] [--> http://superpass.htb/static/]
/vault                (Status: 302) [Size: 243] [--> /account/login?next=%2Fvault]
```
## Foothold

## User


## Root