---
title: "[HTB] Machine: Chaos"
description: "[HTB] Machine: Chaos"
date: 2023-10-25
menu:
  sidebar:
    name: "[HTB] Machine: Chaos"
    identifier: htb-machine-Chaos
    parent: htb-machines-linux
    weight: 10
hero: images/chaos.png
tags: ["HTB"]
---

# Chaos
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.120 --min-rate 5000                                     
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-27 17:02 BST
Nmap scan report for 10.10.10.120 (10.10.10.120)
Host is up (0.17s latency).
Not shown: 65486 filtered tcp ports (no-response), 45 closed tcp ports (conn-refused)
PORT    STATE SERVICE
80/tcp  open  http
110/tcp open  pop3
993/tcp open  imaps
995/tcp open  pop3s

Nmap done: 1 IP address (1 host up) scanned in 26.45 seconds

```
```
└─$ nmap -Pn -p- 10.10.10.120                
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-27 17:06 BST
Stats: 0:11:43 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 60.89% done; ETC: 17:25 (0:07:32 remaining)
Nmap scan report for 10.10.10.120 (10.10.10.120)
Host is up (0.28s latency).
Not shown: 65528 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
80/tcp    open     http
110/tcp   open     pop3
143/tcp   open     imap
993/tcp   open     imaps
995/tcp   open     pop3s
10000/tcp open     snet-sensor-mgmt
63322/tcp filtered unknown

```