---
title: "[HTB] Machine: Frolic"
description: "[HTB] Machine: Frolic"
date: 2023-06-12
menu:
  sidebar:
    name: "[HTB] Machine: Frolic"
    identifier: htb-machine-frolic
    parent: htb-machines-linux
    weight: 10
hero: images/frolic.png
tags: ["HTB"]
---

# Frolic
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV 10.10.10.111
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-13 19:21 BST
Nmap scan report for 10.10.10.111 (10.10.10.111)
Host is up (0.17s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 877b912a0f11b6571ecb9f77cf35e221 (RSA)
|   256 b79b06ddc25e284478411e677d1eb762 (ECDSA)
|_  256 21cf166d82a430c3c69cd738bab502b0 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
9999/tcp open  http        nginx 1.10.3 (Ubuntu)
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
Service Info: Host: FROLIC; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: FROLIC, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: frolic
|   NetBIOS computer name: FROLIC\x00
|   Domain name: \x00
|   FQDN: frolic
|_  System time: 2023-06-13T23:51:45+05:30
|_clock-skew: mean: -1h50m27s, deviation: 3h10m30s, median: -28s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2023-06-13T18:21:45
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.71 seconds
```

