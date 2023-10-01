---
title: "[HTB] Machine: Sharp"
description: "[HTB] Machine: Sharp"
date: 2023-09-30
menu:
  sidebar:
    name: "[HTB] Machine: Sharp"
    identifier: htb-machine-Sharp
    parent: htb-machines-windows
    weight: 10
hero: images/sharp.png
tags: ["HTB"]
---

# Sharp
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.219 --min-rate 10000                                                                       
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:15 BST
Nmap scan report for 10.10.10.219 (10.10.10.219)
Host is up (0.11s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
5985/tcp open  wsman
8888/tcp open  sun-answerbook
8889/tcp open  ddi-tcp-2

Nmap done: 1 IP address (1 host up) scanned in 29.11 seconds
```
```
└─$ nmap -Pn -p135,139,445,5985,8888,8889 -sC -sV 10.10.10.219 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:16 BST
Nmap scan report for 10.10.10.219 (10.10.10.219)
Host is up (0.15s latency).

PORT     STATE SERVICE            VERSION
135/tcp  open  msrpc              Microsoft Windows RPC
139/tcp  open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8888/tcp open  storagecraft-image StorageCraft Image Manager
8889/tcp open  mc-nmf             .NET Message Framing
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-09-30T20:17:13
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: -3s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 107.93 seconds

```
- `smb`
```
└─$ smbmap -H 10.10.10.219 
[+] IP: 10.10.10.219:445        Name: 10.10.10.219                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        dev                                                     NO ACCESS
        IPC$                                                    NO ACCESS       Remote IPC
        kanban                                                  READ ONLY

```

