---
title: "[HTB] Machine: Cache"
description: "[HTB] Machine: Cache"
date: 2023-10-28
menu:
  sidebar:
    name: "[HTB] Machine: Cache"
    identifier: htb-machine-Cache
    parent: htb-machines-linux
    weight: 10
hero: images/cache.png
tags: ["HTB"]
---

# Cache
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.188 --min-rate 5000 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-30 16:53 GMT
Nmap scan report for 10.10.10.188 (10.10.10.188)
Host is up (0.17s latency).
Not shown: 64028 filtered tcp ports (no-response), 1505 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
2201/tcp open  ats

Nmap done: 1 IP address (1 host up) scanned in 91.28 seconds

```
```

```