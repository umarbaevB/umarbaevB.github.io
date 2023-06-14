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
└─$ nmap -Pn -p- 10.10.10.111
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-14 16:52 BST
Nmap scan report for 10.10.10.111 (10.10.10.111)
Host is up (0.15s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
1880/tcp open  vsat-control
9999/tcp open  abyss

Nmap done: 1 IP address (1 host up) scanned in 1037.54 seconds
```
```
└─$ nmap -sV -sC -p22,139,445,1880,9999 10.10.10.111
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-14 17:10 BST
Nmap scan report for 10.10.10.111 (10.10.10.111)
Host is up (0.16s latency).

PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 877b912a0f11b6571ecb9f77cf35e221 (RSA)
|   256 b79b06ddc25e284478411e677d1eb762 (ECDSA)
|_  256 21cf166d82a430c3c69cd738bab502b0 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
1880/tcp open  http        Node.js (Express middleware)
|_http-title: Node-RED
9999/tcp open  http        nginx 1.10.3 (Ubuntu)
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
Service Info: Host: FROLIC; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1h50m26s, deviation: 3h10m30s, median: -27s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-06-14T16:09:55
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: frolic
|   NetBIOS computer name: FROLIC\x00
|   Domain name: \x00
|   FQDN: frolic
|_  System time: 2023-06-14T21:39:55+05:30
|_nbstat: NetBIOS name: FROLIC, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.51 seconds
```
- `smbmap`
```
└─$ smbmap -H 10.10.10.111
[+] Guest session       IP: 10.10.10.111:445    Name: 10.10.10.111                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        IPC$                                                    NO ACCESS       IPC Service (frolic server (Samba, Ubuntu))
```
```
└─$ smbclient -N -L //10.10.10.111

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (frolic server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            FROLIC
```
- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.111:9999 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x php,txt
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.111:9999
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2023/06/14 17:34:41 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/admin/]
/test                 (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/test/]
/dev                  (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/dev/]
/backup               (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/backup/]
/loop                 (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/loop/]
```
```
└─$ gobuster dir -u http://10.10.10.111:9999/dev -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x php,txt
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.111:9999/dev
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2023/06/14 17:45:50 Starting gobuster in directory enumeration mode
===============================================================
/test                 (Status: 200) [Size: 5]
/backup               (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/dev/backup/]
```