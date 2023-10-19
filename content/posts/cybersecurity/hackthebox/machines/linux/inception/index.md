---
title: "[HTB] Machine: Inception"
description: "[HTB] Machine: Inception"
date: 2023-10-17
menu:
  sidebar:
    name: "[HTB] Machine: Inception"
    identifier: htb-machine-Inception
    parent: htb-machines-linux
    weight: 10
hero: images/inception.png
tags: ["HTB"]
---

# Inception
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.67
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-19 19:11 BST
Stats: 0:05:40 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 81.05% done; ETC: 19:18 (0:01:20 remaining)
Nmap scan report for 10.10.10.67 (10.10.10.67)
Host is up (0.10s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
3128/tcp open  squid-http

Nmap done: 1 IP address (1 host up) scanned in 416.34 seconds
```
```
└─$ nmap -Pn -p80,3128 -sC -sV 10.10.10.67
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-19 19:19 BST
Nmap scan report for 10.10.10.67 (10.10.10.67)
Host is up (0.11s latency).

PORT     STATE SERVICE    VERSION
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Inception
3128/tcp open  http-proxy Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.26 seconds

```

- Web server

![](./images/1.png)

- `feroxbuster`
```
└─$ feroxbuster -u http://10.10.10.67 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,php --depth 2   

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.10.67
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, php]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 2
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET       11l       32w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      311c http://10.10.10.67/images => http://10.10.10.67/images/
200      GET        8l       71w     2380c http://10.10.10.67/assets/js/ie/html5shiv.js
200      GET      174l      401w     5284c http://10.10.10.67/assets/js/main.js
200      GET     1187l     2417w    22010c http://10.10.10.67/assets/css/main.css
200      GET       12l       34w      228c http://10.10.10.67/assets/css/ie9.css
200      GET       27l       54w      422c http://10.10.10.67/assets/css/ie8.css
200      GET     1051l      169w     2877c http://10.10.10.67/
200      GET      397l     4810w   190069c http://10.10.10.67/images/bg03.jpg
301      GET        9l       28w      311c http://10.10.10.67/assets => http://10.10.10.67/assets/
200      GET       69l      328w     2307c http://10.10.10.67/README.txt
200      GET       63l     2733w    17128c http://10.10.10.67/LICENSE.txt

```