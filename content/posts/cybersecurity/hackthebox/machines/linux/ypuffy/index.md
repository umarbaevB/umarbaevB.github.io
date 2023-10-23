---
title: "[HTB] Machine: Ypuffy"
description: "[HTB] Machine: Ypuffy"
date: 2023-10-22
menu:
  sidebar:
    name: "[HTB] Machine: Ypuffy"
    identifier: htb-machine-Ypuffy
    parent: htb-machines-linux
    weight: 10
hero: images/ypuffy.png
tags: ["HTB"]
---

# Ypuffy
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.107                
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-23 19:54 BST
Stats: 0:06:07 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 41.38% done; ETC: 20:09 (0:08:34 remaining)
Nmap scan report for 10.10.10.107 (10.10.10.107)
Host is up (0.17s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
389/tcp open  ldap
445/tcp open  microsoft-ds

```
```
└─$ nmap -Pn -p22,80,139,389,445 -sC -sV 10.10.10.107 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-23 20:22 BST
Nmap scan report for 10.10.10.107 (10.10.10.107)
Host is up (0.15s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 2e:19:e6:af:1b:a7:b0:e8:07:2a:2b:11:5d:7b:c6:04 (RSA)
|   256 dd:0f:6a:2a:53:ee:19:50:d9:e5:e7:81:04:8d:91:b6 (ECDSA)
|_  256 21:9e:db:bd:e1:78:4d:72:b0:ea:b4:97:fb:7f:af:91 (ED25519)
80/tcp  open  http        OpenBSD httpd
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: YPUFFY)
389/tcp open  ldap        (Anonymous bind OK)
445/tcp open  �hv^�U      Samba smbd 4.7.6 (workgroup: YPUFFY)
Service Info: Host: YPUFFY

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6)
|   Computer name: ypuffy
|   NetBIOS computer name: YPUFFY\x00
|   Domain name: hackthebox.htb
|   FQDN: ypuffy.hackthebox.htb
|_  System time: 2023-10-23T15:22:21-04:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2023-10-23T19:22:22
|_  start_date: N/A
|_clock-skew: mean: 1h19m11s, deviation: 2h18m34s, median: -48s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.85 seconds

```