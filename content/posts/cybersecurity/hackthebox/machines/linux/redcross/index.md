---
title: "[HTB] Machine: RedCross"
description: "[HTB] Machine: RedCross"
date: 2023-10-22
menu:
  sidebar:
    name: "[HTB] Machine: RedCross"
    identifier: htb-machine-RedCross
    parent: htb-machines-linux
    weight: 10
hero: images/redcross.png
tags: ["HTB"]
---

# RedCross
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.113 --min-rate 5000              
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-23 19:12 BST
Nmap scan report for 10.10.10.113 (10.10.10.113)
Host is up (0.12s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 36.85 seconds
```
```
└─$ nmap -Pn -p22,80,443 -sC -sV 10.10.10.113 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-23 19:13 BST
Nmap scan report for 10.10.10.113 (10.10.10.113)
Host is up (0.12s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.4p1 Debian 10+deb9u3 (protocol 2.0)
| ssh-hostkey: 
|   2048 67:d3:85:f8:ee:b8:06:23:59:d7:75:8e:a2:37:d0:a6 (RSA)
|   256 89:b4:65:27:1f:93:72:1a:bc:e3:22:70:90:db:35:96 (ECDSA)
|_  256 66:bd:a1:1c:32:74:32:e2:e6:64:e8:a5:25:1b:4d:67 (ED25519)
80/tcp  open  http     Apache httpd 2.4.25
|_http-title: Did not follow redirect to https://intra.redcross.htb/
|_http-server-header: Apache/2.4.25 (Debian)
443/tcp open  ssl/http Apache httpd 2.4.25
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=intra.redcross.htb/organizationName=Red Cross International/stateOrProvinceName=NY/countryName=US
| Not valid before: 2018-06-03T19:46:58
|_Not valid after:  2021-02-27T19:46:58
|_http-title: Did not follow redirect to https://intra.redcross.htb/
|_http-server-header: Apache/2.4.25 (Debian)
Service Info: Host: redcross.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.13 seconds

```