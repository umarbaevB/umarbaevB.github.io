---
title: "[HTB] Machine: APT"
description: "[HTB] Machine: APT"
date: 2023-10-04
menu:
  sidebar:
    name: "[HTB] Machine: APT"
    identifier: htb-machine-APT
    parent: htb-machines-windows
    weight: 10
hero: images/apt.png
tags: ["HTB"]
---

# APT
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.213 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 17:44 BST
Nmap scan report for 10.10.10.213 (10.10.10.213)
Host is up (0.14s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT    STATE SERVICE
80/tcp  open  http
135/tcp open  msrpc

Nmap done: 1 IP address (1 host up) scanned in 61.36 seconds

```
```
└─$ nmap -Pn -p80,135 -sC -sV 10.10.10.213 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 17:47 BST
Nmap scan report for 10.10.10.213 (10.10.10.213)
Host is up (0.14s latency).

PORT    STATE SERVICE VERSION
80/tcp  open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Gigantic Hosting | Home
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.35 seconds

```

- Web Server

![](./images/1.png)

![](./images/2.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.213 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -t 50  -x asp,aspx,txt -k
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.213
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              asp,aspx,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 150] [--> http://10.10.10.213/images/]
/css                  (Status: 301) [Size: 147] [--> http://10.10.10.213/css/]
/js                   (Status: 301) [Size: 146] [--> http://10.10.10.213/js/]
/fonts                (Status: 301) [Size: 149] [--> http://10.10.10.213/fonts/]

```