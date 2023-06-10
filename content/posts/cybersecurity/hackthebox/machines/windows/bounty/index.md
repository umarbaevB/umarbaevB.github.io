---
title: "[HTB] Machine: Bounty"
description: "[HTB] Machine: Bounty"
date: 2023-06-07
menu:
  sidebar:
    name: "[HTB] Machine: Bounty"
    identifier: htb-machine-bounty
    parent: htb-machines-windows
    weight: 10
hero: images/bounty.png
tags: ["HTB"]
---

# Bounty
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV 10.10.10.93                        
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-10 16:26 BST
Nmap scan report for 10.10.10.93 (10.10.10.93)
Host is up (0.098s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.31 seconds
```
- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.93 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x asp,aspx 
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.93
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              asp,apsx
[+] Timeout:                 10s
===============================================================
2023/06/10 16:39:26 Starting gobuster in directory enumeration mode
===============================================================
/transfer.aspx        (Status: 200) [Size: 941]
/UploadedFiles        (Status: 301) [Size: 156] [--> http://10.10.10.93/UploadedFiles/]
/uploadedFiles        (Status: 301) [Size: 156] [--> http://10.10.10.93/uploadedFiles/]
/uploadedfiles        (Status: 301) [Size: 156] [--> http://10.10.10.93/uploadedfiles/]
Progress: 661634 / 661683 (99.99%)
===============================================================
2023/06/10 17:15:00 Finished
===============================================================
```