---
title: "[HTB] Machine: Wall"
description: "[HTB] Machine: Wall"
date: 2023-10-25
menu:
  sidebar:
    name: "[HTB] Machine: Wall"
    identifier: htb-machine-Wall
    parent: htb-machines-linux
    weight: 10
hero: images/wall.png
tags: ["HTB"]
---

# Wall
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.157 --min-rate 5000                                     
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-26 19:21 BST
Warning: 10.10.10.157 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.157 (10.10.10.157)
Host is up (0.17s latency).
Not shown: 46442 filtered tcp ports (no-response), 19091 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 119.61 seconds

```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.10.157 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-26 19:27 BST
Nmap scan report for 10.10.10.157 (10.10.10.157)
Host is up (0.19s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 2e:93:41:04:23:ed:30:50:8d:0d:58:23:de:7f:2c:15 (RSA)
|   256 4f:d5:d3:29:40:52:9e:62:58:36:11:06:72:85:1b:df (ECDSA)
|_  256 21:64:d0:c0:ff:1a:b4:29:0b:49:e1:11:81:b6:73:66 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.57 seconds
```

- Web server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.157 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,html -t 50
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.157
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 292]
/index.html           (Status: 200) [Size: 10918]
/.php                 (Status: 403) [Size: 291]
/aa.php               (Status: 200) [Size: 1]
/monitoring           (Status: 401) [Size: 459]
/panel.php            (Status: 200) [Size: 26]

```

## Foothold
- `aa.php` and `panel.php` don't have anything interesting
```
└─$ curl 10.10.10.157/panel.php
Just a test for php file !
```
```
└─$ curl 10.10.10.157/aa.php
1   
```

- We could `fuzz` parameters and `http` methods
  - Nothing interesting except `403` for a few params
```
└─$ wfuzz -z list,GET-HEAD-POST-TRACE-OPTIONS -X FUZZ -u http://10.10.10.157/panel.php
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.157/panel.php
Total requests: 5

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000001:   200        0 L      7 W        26 Ch       "GET - GET"
000000003:   200        0 L      7 W        26 Ch       "POST - POST"
000000002:   200        0 L      0 W        0 Ch        "HEAD - HEAD"
000000004:   405        9 L      35 W       310 Ch      "TRACE - TRACE"
000000005:   200        0 L      7 W        26 Ch       "OPTIONS - OPTIONS"

Total time: 0.380631
Processed Requests: 5
Filtered Requests: 0
Requests/sec.: 13.13606
```
```
└─$ wfuzz -z list,GET-HEAD-POST-TRACE-OPTIONS -X FUZZ -u http://10.10.10.157/aa.php   
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.157/aa.php
Total requests: 5

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000001:   200        0 L      1 W        1 Ch        "GET - GET"
000000002:   200        0 L      0 W        0 Ch        "HEAD - HEAD"
000000004:   405        9 L      35 W       307 Ch      "TRACE - TRACE"
000000003:   200        0 L      1 W        1 Ch        "POST - POST"
000000005:   200        0 L      1 W        1 Ch        "OPTIONS - OPTIONS"

Total time: 0
Processed Requests: 5
Filtered Requests: 0
Requests/sec.: 0
```
```
└─$ wfuzz -c -u http://10.10.10.157/panel.php -d 'FUZZ=test' -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt --hw 7
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.157/panel.php
Total requests: 6453

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000002631:   403        11 L     32 W       296 Ch      "hostname"`
000003612:   403        11 L     32 W       296 Ch      "nc"`
000004040:   403        11 L     32 W       296 Ch      "passwd" 
```
```

```
- Each endpoint returns `403` when trying each param
```
└─$ curl http://10.10.10.157 -d "nc"    
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access /
on this server.<br />
</p>
<hr>
<address>Apache/2.4.29 (Ubuntu) Server at 10.10.10.157 Port 80</address>
</body></html>
```

- `monitoring` returns `http basic auth`
  - Tried default passwords, but nothing works

![](./images/1.png)

- But when I tried fuzzing, we receive `200` on `POST`
```
└─$ wfuzz -z list,GET-HEAD-POST-TRACE-OPTIONS -X FUZZ -u http://10.10.10.157/monitoring/
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.157/monitoring/
Total requests: 5

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000001:   401        14 L     54 W       459 Ch      "GET - GET"`
000000002:   401        0 L      0 W        0 Ch        "HEAD - HEAD"`
000000003:   200        5 L      21 W       154 Ch      "POST - POST"`
000000005:   200        0 L      0 W        0 Ch        "OPTIONS - OPTIONS"`
000000004:   405        9 L      35 W       312 Ch      "TRACE - TRACE"`

Total time: 0.695538
Processed Requests: 5
Filtered Requests: 0
Requests/sec.: 7.188679

```

- If we `curl`, we see another endpoint
```
└─$ curl -XPOST http://10.10.10.157/monitoring/
<h1>This page is not ready yet !</h1>
<h2>We should redirect you to the required page !</h2>

<meta http-equiv="refresh" content="0; URL='/centreon'" />
```

- `Centreon v. 19.04.0`

![](./images/2.png)

- We have an [exploit](https://www.exploit-db.com/exploits/47069)
  - But we need creds
    - Default creds didn't work
  - Let's try to bruteforce
```

```
## User

## Root