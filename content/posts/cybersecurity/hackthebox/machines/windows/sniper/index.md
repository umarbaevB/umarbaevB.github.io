---
title: "[HTB] Machine: Sniper"
description: "[HTB] Machine: Sniper"
date: 2023-09-11
menu:
  sidebar:
    name: "[HTB] Machine: Sniper"
    identifier: htb-machine-Sniper
    parent: htb-machines-windows
    weight: 10
hero: images/sniper.png
tags: ["HTB"]
---

# Sniper
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.151 -T4                   
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-11 19:59 BST
Stats: 0:17:07 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Nmap scan report for 10.10.10.151
Host is up (0.024s latency).
Not shown: 65530 filtered ports
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
49667/tcp open  unknown
```
```
└─$ nmap -Pn -p80,135,139,445 -sC -sV 10.10.10.151 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-11 20:17 BST
Nmap scan report for 10.10.10.151 (10.10.10.151)
Host is up (0.13s latency).

PORT    STATE SERVICE       VERSION
80/tcp  open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Sniper Co.
|_http-server-header: Microsoft-IIS/10.0
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 6h59m22s
| smb2-time: 
|   date: 2023-09-12T02:17:13
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required


```
- `smb`
```
└─$ smbclient -N -L //10.10.10.151                                                    
session setup failed: NT_STATUS_ACCESS_DENIED                                            
```

- Web Server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.151/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt,asp,aspx --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.151/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,asp,aspx
[+] Timeout:                 10s
===============================================================
2023/09/12 15:57:15 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 150] [--> http://10.10.10.151/images/]
/blog                 (Status: 301) [Size: 148] [--> http://10.10.10.151/blog/]
/user                 (Status: 301) [Size: 148] [--> http://10.10.10.151/user/]
/Images               (Status: 301) [Size: 150] [--> http://10.10.10.151/Images/]
/css                  (Status: 301) [Size: 147] [--> http://10.10.10.151/css/]
/js                   (Status: 301) [Size: 146] [--> http://10.10.10.151/js/]

```

## Foothold
- `/blog`

![](./images/2.png)




## User







## Root