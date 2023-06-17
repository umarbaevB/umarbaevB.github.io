---
title: "[HTB] Machine: Bastion"
description: "[HTB] Machine: Bastion"
date: 2023-06-16
menu:
  sidebar:
    name: "[HTB] Machine: Bastion"
    identifier: htb-machine-bastion
    parent: htb-machines-windows
    weight: 10
hero: images/bastion.png
tags: ["HTB"]
---

# Bastion
## Enumeration
- `nmap`
```
└─$ nmap -Pn -sC -sV 10.10.10.134 -T4
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-17 21:20 BST
Nmap scan report for 10.10.10.134 (10.10.10.134)
Host is up (0.13s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT    STATE SERVICE      VERSION
22/tcp  open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a56ae753c780ec8564dcb1c22bf458a (RSA)
|   256 cc2e56ab1997d5bb03fb82cd63da6801 (ECDSA)
|_  256 935f5daaca9f53e7f282e664a8a3a018 (ED25519)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-06-17T20:20:50
|_  start_date: 2023-06-17T18:27:22
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: -40m29s, deviation: 1h09m15s, median: -30s
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-06-17T22:20:49+02:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.14 seconds

```
