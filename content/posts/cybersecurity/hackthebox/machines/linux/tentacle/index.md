---
title: "[HTB] Machine: Tentacle"
description: "[HTB] Machine: Tentacle"
date: 2023-09-27
menu:
  sidebar:
    name: "[HTB] Machine: Tentacle"
    identifier: htb-machine-Tentacle
    parent: htb-machines-linux
    weight: 10
hero: images/tentacle.png
tags: ["HTB"]
---

# Tentacle
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.224 -T4 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-26 20:09 BST
Nmap scan report for 10.10.10.224 (10.10.10.224)
Host is up (0.093s latency).
Not shown: 65495 filtered tcp ports (no-response), 36 filtered tcp ports (host-unreach)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
88/tcp   open  kerberos-sec
3128/tcp open  squid-http

Nmap done: 1 IP address (1 host up) scanned in 39.58 seconds

```
```
└─$ nmap -Pn -p22,53,88,3128 -sC -sV 10.10.10.224 -T4 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-26 20:12 BST
Nmap scan report for 10.10.10.224 (10.10.10.224)
Host is up (0.12s latency).

PORT     STATE SERVICE      VERSION
22/tcp   open  ssh          OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 8d:dd:18:10:e5:7b:b0:da:a3:fa:14:37:a7:52:7a:9c (RSA)
|   256 f6:a9:2e:57:f8:18:b6:f4:ee:03:41:27:1e:1f:93:99 (ECDSA)
|_  256 04:74:dd:68:79:f4:22:78:d8:ce:dd:8b:3e:8c:76:3b (ED25519)
53/tcp   open  domain       ISC BIND 9.11.20 (RedHat Enterprise Linux 8)
| dns-nsid: 
|_  bind.version: 9.11.20-RedHat-9.11.20-5.el8
88/tcp   open  kerberos-sec MIT Kerberos (server time: 2023-09-26 19:11:56Z)
3128/tcp open  http-proxy   Squid http proxy 4.11
|_http-server-header: squid/4.11
|_http-title: ERROR: The requested URL could not be retrieved
Service Info: Host: REALCORP.HTB; OS: Linux; CPE: cpe:/o:redhat:enterprise_linux:8

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.44 seconds

```
```
└─$ sudo nmap -Pn -sU 10.10.10.224 -T4 --min-rate 5000
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-26 20:13 BST
Nmap scan report for 10.10.10.224 (10.10.10.224)
Host is up (0.10s latency).
Not shown: 991 open|filtered udp ports (no-response)
PORT      STATE    SERVICE
53/udp    open     domain
88/udp    open     kerberos-sec
123/udp   open     ntp
17787/udp filtered unknown
20560/udp filtered unknown
21016/udp filtered unknown
40019/udp filtered unknown
44185/udp filtered unknown
49179/udp filtered unknown

```