---
title: "[HTB] Machine: OnlyForYou"
description: "[HTB] Machine: OnlyForYou"
date: 2023-08-26
menu:
  sidebar:
    name: "[HTB] Machine: OnlyForYou"
    identifier: htb-machine-OnlyForYou
    parent: htb-machines-linux
    weight: 10
hero: images/onlyforyou.png
tags: ["HTB"]
---

# OnlyForYou
## Enumeration
- `nmap`
```
┌──(kali㉿kali)-[~]
└─$ nmap -p- -Pn 10.10.11.210 -T4                                                               
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-25 16:45 BST
Nmap scan report for 10.10.11.210 (10.10.11.210)
Host is up (0.15s latency).
Not shown: 65519 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http

Nmap done: 1 IP address (1 host up) scanned in 1249.23 seconds
```
```
┌──(kali㉿kali)-[~]
└─$ nmap -p22,80 -sC -sV -Pn 10.10.11.210 -T4
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-25 17:08 BST
Nmap scan report for 10.10.11.210 (10.10.11.210)
Host is up (0.19s latency).

PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e883e0a9fd43df38198aaa35438411ec (RSA)
|   256 83f235229b03860c16cfb3fa9f5acd08 (ECDSA)
|_  256 445f7aa377690a77789b04e09f11db80 (ED25519)
80/tcp    open   http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://only4you.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.46 seconds
```
- `gobuster`
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://beta.only4you.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt              
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://beta.only4you.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/05/25 17:33:53 Starting gobuster in directory enumeration mode
===============================================================
/download             (Status: 405) [Size: 683]
/source               (Status: 200) [Size: 12127]
/list                 (Status: 200) [Size: 5934]
/convert              (Status: 200) [Size: 2760]
/resize               (Status: 200) [Size: 2984]
Progress: 20734 / 26585 (77.99%)[ERROR] 2023/05/25 17:38:42 [!] parse "http://beta.only4you.htb/error\x1f_log": net/url: invalid control character in URL
Progress: 26584 / 26585 (100.00%)
===============================================================
2023/05/25 17:40:03 Finished
===============================================================

```

## Foothold


## User


## Root