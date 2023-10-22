---
title: "[HTB] Machine: Waldo"
description: "[HTB] Machine: Waldo"
date: 2023-10-22
menu:
  sidebar:
    name: "[HTB] Machine: Waldo"
    identifier: htb-machine-Waldo
    parent: htb-machines-linux
    weight: 10
hero: images/waldo.png
tags: ["HTB"]
---

# Waldo
## Enumeration
- `nmap`
```
└─$ nmap -Pn -sC -sV 10.10.10.87
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-22 20:00 BST
Stats: 0:03:49 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 72.30% done; ETC: 20:05 (0:01:28 remaining)
Stats: 0:09:53 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 99.99% done; ETC: 20:10 (0:00:00 remaining)
Nmap scan report for 10.10.10.87 (10.10.10.87)
Host is up (0.100s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE    SERVICE        VERSION
22/tcp   open     ssh            OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey: 
|   2048 c4:ff:81:aa:ac:df:66:9e:da:e1:c8:78:00:ab:32:9e (RSA)
|   256 b3:e7:54:6a:16:bd:c9:29:1f:4a:8c:cd:4c:01:24:27 (ECDSA)
|_  256 38:64:ac:57:56:44:d5:69:de:74:a8:88:dc:a0:b4:fd (ED25519)
80/tcp   open     http           nginx 1.12.2
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: nginx/1.12.2
| http-title: List Manager
|_Requested resource was /list.html
8888/tcp filtered sun-answerbook

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 620.98 seconds
```

