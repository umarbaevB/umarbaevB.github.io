---
title: "[HTB] Machine: Laboratory"
description: "[HTB] Machine: Laboratory"
date: 2023-07-05
menu:
  sidebar:
    name: "[HTB] Machine: Laboratory"
    identifier: htb-machine-laboratory
    parent: htb-machines-linux
    weight: 10
hero: images/laboratory.png
tags: ["HTB"]
---

# Laboratory
## Enumeration
- ```Nmap```
```
└─$ nmap -Pn -sC -sV 10.10.10.209 -T4
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-06 19:08 BST
Nmap scan report for 10.10.10.209 (10.10.10.209)
Host is up (0.11s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 594d4ec2d8cfda9da8c8d0fd99a84617 (RSA)
|   256 7ff3dcfb2dafcbff9934ace0f8001e47 (ECDSA)
|_  256 530e966b9ce9c1a170516c2dce7b43e8 (ED25519)
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Doctor
8089/tcp open  ssl/http Splunkd httpd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Not valid before: 2020-09-06T15:57:27
|_Not valid after:  2023-09-06T15:57:27
|_http-server-header: Splunkd
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: splunkd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.13 seconds

```
- Web Server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.209 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 --no-error -x php,txt 
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.209
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2023/07/06 19:11:54 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 277]
/images               (Status: 301) [Size: 313] [--> http://10.10.10.209/images/]
/css                  (Status: 301) [Size: 310] [--> http://10.10.10.209/css/]
/js                   (Status: 301) [Size: 309] [--> http://10.10.10.209/js/]
/fonts                (Status: 301) [Size: 312] [--> http://10.10.10.209/fonts/]
/.php                 (Status: 403) [Size: 277]
/server-status        (Status: 403) [Size: 277]

```

- We have `Splunk` on port `8089`

![](./images/2.png)


## Foothold


## User

## Root