---
title: "[HTB] Machine: BroScience"
description: "[HTB] Machine: BroScience"
date: 2023-11-13
menu:
  sidebar:
    name: "[HTB] Machine: BroScience"
    identifier: htb-machine-BroScience
    parent: htb-machines-linux
    weight: 10
hero: images/broscience.png
tags: ["HTB"]
---

# BroScience
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.195 --min-rate 1000                                                                                                        
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-12 20:07 GMT
Warning: 10.10.11.195 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.195 (10.10.11.195)
Host is up (0.17s latency).
Not shown: 65488 closed tcp ports (conn-refused), 44 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 79.36 seconds

```
```
└─$ nmap -Pn -p22,80,443 -sC -sV 10.10.11.195 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-12 20:09 GMT
Nmap scan report for 10.10.11.195 (10.10.11.195)
Host is up (0.23s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 df:17:c6:ba:b1:82:22:d9:1d:b5:eb:ff:5d:3d:2c:b7 (RSA)
|   256 3f:8a:56:f8:95:8f:ae:af:e3:ae:7e:b8:80:f6:79:d2 (ECDSA)
|_  256 3c:65:75:27:4a:e2:ef:93:91:37:4c:fd:d9:d4:63:41 (ED25519)
80/tcp  open  http     Apache httpd 2.4.54
|_http-title: Did not follow redirect to https://broscience.htb/
|_http-server-header: Apache/2.4.54 (Debian)
443/tcp open  ssl/http Apache httpd 2.4.54 ((Debian))
| tls-alpn: 
|_  http/1.1
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| ssl-cert: Subject: commonName=broscience.htb/organizationName=BroScience/countryName=AT
| Not valid before: 2022-07-14T19:48:36
|_Not valid after:  2023-07-14T19:48:36
|_http-server-header: Apache/2.4.54 (Debian)
|_ssl-date: TLS randomness does not represent time
|_http-title: BroScience : Home
Service Info: Host: broscience.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.41 seconds

```

- Web Server

![](./images/1.png)

- `feroxbuster`
```
└─$ feroxbuster -u https://broscience.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -t 50 -k --depth 2

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ https://broscience.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔓  Insecure              │ true
 🔃  Recursion Depth       │ 2
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      280c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      321c https://broscience.htb/includes => https://broscience.htb/includes/
301      GET        9l       28w      319c https://broscience.htb/images => https://broscience.htb/images/
200      GET        3l        7w       44c https://broscience.htb/styles/light.css
200      GET       29l       70w     1309c https://broscience.htb/user.php
200      GET       28l       71w     1322c https://broscience.htb/exercise.php
200      GET        1l        4w       39c https://broscience.htb/includes/img.php
200      GET       42l       97w     1936c https://broscience.htb/login.php
200      GET      147l      510w     9295c https://broscience.htb/
200      GET        0l        0w        0c https://broscience.htb/includes/utils.php
200      GET      220l     1542w   123552c https://broscience.htb/images/reverse_butterfly.jpeg
200      GET      122l      678w    56054c https://broscience.htb/images/bench.png
500      GET        2l        4w       65c https://broscience.htb/includes/navbar.php
200      GET        5l       14w      369c https://broscience.htb/includes/header.php
200      GET        0l        0w        0c https://broscience.htb/includes/db_connect.php
301      GET        9l       28w      319c https://broscience.htb/styles => https://broscience.htb/styles/
200      GET        3l        7w       41c https://broscience.htb/styles/dark.css
200      GET      161l     1002w    83700c https://broscience.htb/images/seated_rows.png
301      GET        9l       28w      323c https://broscience.htb/javascript => https://broscience.htb/javascript/
200      GET      383l     2045w   205698c https://broscience.htb/images/barbell_squats.jpeg
200      GET        0l        0w   598012c https://broscience.htb/images/shoulder_press.jpeg
200      GET        0l        0w  1011236c https://broscience.htb/images/deadlift.png
200      GET        0l        0w   326860c https://broscience.htb/images/dumbell_curls.jpeg
200      GET        0l        0w   297898c https://broscience.htb/images/tricep_extensions.jpeg
301      GET        9l       28w      319c https://broscience.htb/manual => https://broscience.htb/manual/
301      GET        9l       28w      326c https://broscience.htb/manual/images => https://broscience.htb/manual/images/
301      GET        9l       28w      322c https://broscience.htb/manual/en => https://broscience.htb/manual/en/
301      GET        9l       28w      325c https://broscience.htb/manual/style => https://broscience.htb/manual/style/
301      GET        9l       28w      322c https://broscience.htb/manual/de => https://broscience.htb/manual/de/
301      GET        9l       28w      322c https://broscience.htb/manual/fr => https://broscience.htb/manual/fr/
301      GET        9l       28w      322c https://broscience.htb/manual/es => https://broscience.htb/manual/es/
301      GET        9l       28w      322c https://broscience.htb/manual/ru => https://broscience.htb/manual/ru/
200      GET      129l      566w    10124c https://broscience.htb/manual/ja/index.html
200      GET      127l      643w    10996c https://broscience.htb/manual/ru/index.html
200      GET      118l      578w     9683c https://broscience.htb/manual/ko/index.html
200      GET      130l      623w     9843c https://broscience.htb/manual/de/index.html
200      GET      127l      637w     9903c https://broscience.htb/manual/tr/index.html
200      GET      129l      701w    10325c https://broscience.htb/manual/es/index.html
200      GET      123l      648w     9828c https://broscience.htb/manual/pt-br/index.html
200      GET      130l      683w    10033c https://broscience.htb/manual/fr/index.html
200      GET      127l      636w     9666c https://broscience.htb/manual/en/index.html
200      GET      121l      605w     9416c https://broscience.htb/manual/da/index.html
200      GET      124l      553w     9400c https://broscience.htb/manual/zh-cn/index.html
200      GET       14l       28w      676c https://broscience.htb/manual/
301      GET        9l       28w      322c https://broscience.htb/manual/ja => https://broscience.htb/manual/ja/
301      GET        9l       28w      322c https://broscience.htb/manual/tr => https://broscience.htb/manual/tr/
301      GET        9l       28w      322c https://broscience.htb/manual/da => https://broscience.htb/manual/da/
301      GET        9l       28w      322c https://broscience.htb/manual/ko => https://broscience.htb/manual/ko/
301      GET        9l       28w      325c https://broscience.htb/manual/zh-cn => https://broscience.htb/manual/zh-cn/
[####################] - 71s   168946/168946  0s      found:48      errors:151508 
[####################] - 61s    56294/56294   919/s   https://broscience.htb/ 
[####################] - 2s     56294/56294   35765/s https://broscience.htb/includes/ => Directory listing
[####################] - 8s     56294/56294   7381/s  https://broscience.htb/images/ => Directory listing
[####################] - 1s     56294/56294   68235/s https://broscience.htb/styles/ => Directory listing
[####################] - 56s    56294/56294   1011/s  https://broscience.htb/javascript/ 
[####################] - 51s    56294/56294   1107/s  https://broscience.htb/manual/   
```

- Nothing on `vhosts`