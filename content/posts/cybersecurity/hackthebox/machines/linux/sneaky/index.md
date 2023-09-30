---
title: "[HTB] Machine: Sneaky"
description: "[HTB] Machine: Sneaky"
date: 2023-09-30
menu:
  sidebar:
    name: "[HTB] Machine: Sneaky"
    identifier: htb-machine-Sneaky
    parent: htb-machines-linux
    weight: 10
hero: images/sneaky.png
tags: ["HTB"]
---

# Sneaky
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.20 --min-rate 10000           
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:49 BST
Warning: 10.10.10.20 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.20 (10.10.10.20)
Host is up (0.10s latency).
Not shown: 54464 filtered tcp ports (no-response), 6 filtered tcp ports (host-unreach), 11064 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 70.54 seconds

```
```
└─$ nmap -Pn -p80 -sC -sV 10.10.10.20 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:50 BST
Nmap scan report for 10.10.10.20 (10.10.10.20)
Host is up (0.099s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Under Development!
|_http-server-header: Apache/2.4.7 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.51 seconds
```
```
└─$ sudo nmap -Pn -sU --top-ports 10 10.10.10.20 --min-rate 10000
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:52 BST
Nmap scan report for 10.10.10.20 (10.10.10.20)
Host is up (0.16s latency).

PORT     STATE         SERVICE
53/udp   closed        domain
67/udp   open|filtered dhcps
123/udp  open|filtered ntp
135/udp  closed        msrpc
137/udp  closed        netbios-ns
138/udp  open|filtered netbios-dgm
161/udp  open          snmp
445/udp  closed        microsoft-ds
631/udp  open|filtered ipp
1434/udp open|filtered ms-sql-m

Nmap done: 1 IP address (1 host up) scanned in 1.35 seconds
```