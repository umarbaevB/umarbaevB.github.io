---
title: "[HTB] Machine: Inject"
description: "[HTB] Machine: Inject"
date: 2023-08-28
menu:
  sidebar:
    name: "[HTB] Machine: Inject"
    identifier: htb-machine-Inject
    parent: htb-machines-linux
    weight: 10
hero: images/inject.png
tags: ["HTB"]
---

# Inject
## Enumeration
- `nmap`
```
┌──(kali㉿kali)-[~/Documents/tasks]
└─$ nmap -Pn -p- -T4 10.10.11.204                                     
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-17 14:17 EDT
Nmap scan report for 10.10.11.204 (10.10.11.204)
Host is up (0.16s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
22/tcp    open     ssh
5427/tcp  filtered sco-peer-tta
8080/tcp  open     http-proxy
62318/tcp filtered unknown

```
```
┌──(kali㉿kali)-[~/Documents/tasks]
└─$ nmap -sC -sV -p22,5427,8080,62318 10.10.11.204 -Pn -T4            
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-17 14:27 EDT
Nmap scan report for 10.10.11.204 (10.10.11.204)
Host is up (0.12s latency).

PORT      STATE  SERVICE      VERSION
22/tcp    open   ssh          OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 caf10c515a596277f0a80c5c7c8ddaf8 (RSA)
|   256 d51c81c97b076b1cc1b429254b52219f (ECDSA)
|_  256 db1d8ceb9472b0d3ed44b96c93a7f91d (ED25519)
5427/tcp  closed sco-peer-tta
8080/tcp  open   nagios-nsca  Nagios NSCA
|_http-title: Home
62318/tcp closed unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.26 seconds

```