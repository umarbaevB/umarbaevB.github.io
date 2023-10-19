---
title: "[HTB] Machine: Stratosphere"
description: "[HTB] Machine: Stratosphere"
date: 2023-10-17
menu:
  sidebar:
    name: "[HTB] Machine: Stratosphere"
    identifier: htb-machine-Stratosphere
    parent: htb-machines-linux
    weight: 10
hero: images/Stratosphere.png
tags: ["HTB"]
---

# Stratosphere
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.64              
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-19 19:33 BST
Nmap scan report for 10.10.10.64 (10.10.10.64)
Host is up (0.10s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 182.77 seconds
```
```
└─$ nmap -Pn -p22,80,8080 -sC -sV 10.10.10.64
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-19 19:39 BST
Nmap scan report for 10.10.10.64 (10.10.10.64)
Host is up (0.18s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.4p1 Debian 10+deb9u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 5b:16:37:d4:3c:18:04:15:c4:02:01:0d:db:07:ac:2d (RSA)
|   256 e3:77:7b:2c:23:b0:8d:df:38:35:6c:40:ab:f6:81:50 (ECDSA)
|_  256 d7:6b:66:9c:19:fc:aa:66:6c:18:7a:cc:b5:87:0e:40 (ED25519)
80/tcp   open  http
|_http-title: Stratosphere
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 1114
|     Date: Thu, 19 Oct 2023 18:38:28 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 404 
|     Found</title><style type="text/css">h1 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:22px;} h2 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:16px;} h3 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:14px;} body {font-family:Tahoma,Arial,sans-serif;color:black;background-color:white;} b {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;} p {font-family:Tahoma,Arial,sans-serif;background:white;color:black;font-size:12px;} a {color:black;} a.name {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body>
|   GetRequest: 
|     HTTP/1.1 200 
|     Accept-Ranges: bytes
|     ETag: W/"1708-1519762495000"
|     Last-Modified: Tue, 27 Feb 2018 20:14:55 GMT
|     Content-Type: text/html
|     Content-Length: 1708
|     Date: Thu, 19 Oct 2023 18:38:27 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta charset="utf-8"/>
|     <title>Stratosphere</title>
|     <link rel="stylesheet" type="text/css" href="main.css">
|     </head>
|     <body>
|     <div id="background"></div>
|     <header id="main-header" class="hidden">
|     <div class="container">
|     <div class="content-wrap">
|     <p><i class="fa fa-diamond"></i></p>
|     <nav>
|     class="btn" href="GettingStarted.html">Get started</a>
|     </nav>
|     </div>
|     </div>
|     </header>
|     <section id="greeting">
|     <div class="container">
|     <div class="content-wrap">
|     <h1>Stratosphere<br>We protect your credit.</h1>
|     class="btn" href="GettingStarted.html">Get started now</a>
|     <p><i class="ar
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET, HEAD, POST, PUT, DELETE, OPTIONS
|     Content-Length: 0
|     Date: Thu, 19 Oct 2023 18:38:27 GMT
|     Connection: close
|   RTSPRequest, X11Probe: 
|     HTTP/1.1 400 
|     Date: Thu, 19 Oct 2023 18:38:28 GMT
|_    Connection: close
| http-methods: 
|_  Potentially risky methods: PUT DELETE
8080/tcp open  http-proxy
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 1114
|     Date: Thu, 19 Oct 2023 18:38:28 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 404 
|     Found</title><style type="text/css">h1 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:22px;} h2 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:16px;} h3 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:14px;} body {font-family:Tahoma,Arial,sans-serif;color:black;background-color:white;} b {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;} p {font-family:Tahoma,Arial,sans-serif;background:white;color:black;font-size:12px;} a {color:black;} a.name {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body>
|   GetRequest: 
|     HTTP/1.1 200 
|     Accept-Ranges: bytes
|     ETag: W/"1708-1519762495000"
|     Last-Modified: Tue, 27 Feb 2018 20:14:55 GMT
|     Content-Type: text/html
|     Content-Length: 1708
|     Date: Thu, 19 Oct 2023 18:38:27 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta charset="utf-8"/>
|     <title>Stratosphere</title>
|     <link rel="stylesheet" type="text/css" href="main.css">
|     </head>
|     <body>
|     <div id="background"></div>
|     <header id="main-header" class="hidden">
|     <div class="container">
|     <div class="content-wrap">
|     <p><i class="fa fa-diamond"></i></p>
|     <nav>
|     class="btn" href="GettingStarted.html">Get started</a>
|     </nav>
|     </div>
|     </div>
|     </header>
|     <section id="greeting">
|     <div class="container">
|     <div class="content-wrap">
|     <h1>Stratosphere<br>We protect your credit.</h1>
|     class="btn" href="GettingStarted.html">Get started now</a>
|     <p><i class="ar
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET, HEAD, POST, PUT, DELETE, OPTIONS
|     Content-Length: 0
|     Date: Thu, 19 Oct 2023 18:38:27 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 400 
|     Date: Thu, 19 Oct 2023 18:38:28 GMT
|_    Connection: close
|_http-title: Stratosphere
| http-methods: 
|_  Potentially risky methods: PUT DELETE
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port80-TCP:V=7.94%I=7%D=10/19%Time=653177D2%P=x86_64-pc-linux-gnu%r(Get
SF:Request,786,"HTTP/1\.1\x20200\x20\r\nAccept-Ranges:\x20bytes\r\nETag:\x
SF:20W/\"1708-1519762495000\"\r\nLast-Modified:\x20Tue,\x2027\x20Feb\x2020
SF:18\x2020:14:55\x20GMT\r\nContent-Type:\x20text/html\r\nContent-Length:\
SF:x201708\r\nDate:\x20Thu,\x2019\x20Oct\x202023\x2018:38:27\x20GMT\r\nCon
SF:nection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html>\n<head>\n\x20\x20\x
SF:20\x20<meta\x20charset=\"utf-8\"/>\n\x20\x20\x20\x20<title>Stratosphere
SF:</title>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20type=\"text/cs
SF:s\"\x20href=\"main\.css\">\n</head>\n\n<body>\n<div\x20id=\"background\
SF:"></div>\n<header\x20id=\"main-header\"\x20class=\"hidden\">\n\x20\x20<
SF:div\x20class=\"container\">\n\x20\x20\x20\x20<div\x20class=\"content-wr
SF:ap\">\n\x20\x20\x20\x20\x20\x20<p><i\x20class=\"fa\x20fa-diamond\"></i>
SF:</p>\n\x20\x20\x20\x20\x20\x20<nav>\n\x20\x20\x20\x20\x20\x20\x20\x20<a
SF:\x20class=\"btn\"\x20href=\"GettingStarted\.html\">Get\x20started</a>\n
SF:\x20\x20\x20\x20\x20\x20</nav>\n\x20\x20\x20\x20</div>\n\x20\x20</div>\
SF:n</header>\n\n<section\x20id=\"greeting\">\n\x20\x20<div\x20class=\"con
SF:tainer\">\n\x20\x20\x20\x20<div\x20class=\"content-wrap\">\n\x20\x20\x2
SF:0\x20\x20\x20<h1>Stratosphere<br>We\x20protect\x20your\x20credit\.</h1>
SF:\n\x20\x20\x20\x20\x20\x20<a\x20class=\"btn\"\x20href=\"GettingStarted\
SF:.html\">Get\x20started\x20now</a>\n\x20\x20\x20\x20\x20\x20<p><i\x20cla
SF:ss=\"ar")%r(HTTPOptions,8A,"HTTP/1\.1\x20200\x20\r\nAllow:\x20GET,\x20H
SF:EAD,\x20POST,\x20PUT,\x20DELETE,\x20OPTIONS\r\nContent-Length:\x200\r\n
SF:Date:\x20Thu,\x2019\x20Oct\x202023\x2018:38:27\x20GMT\r\nConnection:\x2
SF:0close\r\n\r\n")%r(RTSPRequest,49,"HTTP/1\.1\x20400\x20\r\nDate:\x20Thu
SF:,\x2019\x20Oct\x202023\x2018:38:28\x20GMT\r\nConnection:\x20close\r\n\r
SF:\n")%r(X11Probe,49,"HTTP/1\.1\x20400\x20\r\nDate:\x20Thu,\x2019\x20Oct\
SF:x202023\x2018:38:28\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(FourOhFo
SF:urRequest,4F6,"HTTP/1\.1\x20404\x20\r\nContent-Type:\x20text/html;chars
SF:et=utf-8\r\nContent-Language:\x20en\r\nContent-Length:\x201114\r\nDate:
SF:\x20Thu,\x2019\x20Oct\x202023\x2018:38:28\x20GMT\r\nConnection:\x20clos
SF:e\r\n\r\n<!doctype\x20html><html\x20lang=\"en\"><head><title>HTTP\x20St
SF:atus\x20404\x20\xe2\x80\x93\x20Not\x20Found</title><style\x20type=\"tex
SF:t/css\">h1\x20{font-family:Tahoma,Arial,sans-serif;color:white;backgrou
SF:nd-color:#525D76;font-size:22px;}\x20h2\x20{font-family:Tahoma,Arial,sa
SF:ns-serif;color:white;background-color:#525D76;font-size:16px;}\x20h3\x2
SF:0{font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525
SF:D76;font-size:14px;}\x20body\x20{font-family:Tahoma,Arial,sans-serif;co
SF:lor:black;background-color:white;}\x20b\x20{font-family:Tahoma,Arial,sa
SF:ns-serif;color:white;background-color:#525D76;}\x20p\x20{font-family:Ta
SF:homa,Arial,sans-serif;background:white;color:black;font-size:12px;}\x20
SF:a\x20{color:black;}\x20a\.name\x20{color:black;}\x20\.line\x20{height:1
SF:px;background-color:#525D76;border:none;}</style></head><body>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8080-TCP:V=7.94%I=7%D=10/19%Time=653177D2%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,786,"HTTP/1\.1\x20200\x20\r\nAccept-Ranges:\x20bytes\r\nETag:
SF:\x20W/\"1708-1519762495000\"\r\nLast-Modified:\x20Tue,\x2027\x20Feb\x20
SF:2018\x2020:14:55\x20GMT\r\nContent-Type:\x20text/html\r\nContent-Length
SF::\x201708\r\nDate:\x20Thu,\x2019\x20Oct\x202023\x2018:38:27\x20GMT\r\nC
SF:onnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html>\n<head>\n\x20\x20
SF:\x20\x20<meta\x20charset=\"utf-8\"/>\n\x20\x20\x20\x20<title>Stratosphe
SF:re</title>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20type=\"text/
SF:css\"\x20href=\"main\.css\">\n</head>\n\n<body>\n<div\x20id=\"backgroun
SF:d\"></div>\n<header\x20id=\"main-header\"\x20class=\"hidden\">\n\x20\x2
SF:0<div\x20class=\"container\">\n\x20\x20\x20\x20<div\x20class=\"content-
SF:wrap\">\n\x20\x20\x20\x20\x20\x20<p><i\x20class=\"fa\x20fa-diamond\"></
SF:i></p>\n\x20\x20\x20\x20\x20\x20<nav>\n\x20\x20\x20\x20\x20\x20\x20\x20
SF:<a\x20class=\"btn\"\x20href=\"GettingStarted\.html\">Get\x20started</a>
SF:\n\x20\x20\x20\x20\x20\x20</nav>\n\x20\x20\x20\x20</div>\n\x20\x20</div
SF:>\n</header>\n\n<section\x20id=\"greeting\">\n\x20\x20<div\x20class=\"c
SF:ontainer\">\n\x20\x20\x20\x20<div\x20class=\"content-wrap\">\n\x20\x20\
SF:x20\x20\x20\x20<h1>Stratosphere<br>We\x20protect\x20your\x20credit\.</h
SF:1>\n\x20\x20\x20\x20\x20\x20<a\x20class=\"btn\"\x20href=\"GettingStarte
SF:d\.html\">Get\x20started\x20now</a>\n\x20\x20\x20\x20\x20\x20<p><i\x20c
SF:lass=\"ar")%r(HTTPOptions,8A,"HTTP/1\.1\x20200\x20\r\nAllow:\x20GET,\x2
SF:0HEAD,\x20POST,\x20PUT,\x20DELETE,\x20OPTIONS\r\nContent-Length:\x200\r
SF:\nDate:\x20Thu,\x2019\x20Oct\x202023\x2018:38:27\x20GMT\r\nConnection:\
SF:x20close\r\n\r\n")%r(RTSPRequest,49,"HTTP/1\.1\x20400\x20\r\nDate:\x20T
SF:hu,\x2019\x20Oct\x202023\x2018:38:28\x20GMT\r\nConnection:\x20close\r\n
SF:\r\n")%r(FourOhFourRequest,4F6,"HTTP/1\.1\x20404\x20\r\nContent-Type:\x
SF:20text/html;charset=utf-8\r\nContent-Language:\x20en\r\nContent-Length:
SF:\x201114\r\nDate:\x20Thu,\x2019\x20Oct\x202023\x2018:38:28\x20GMT\r\nCo
SF:nnection:\x20close\r\n\r\n<!doctype\x20html><html\x20lang=\"en\"><head>
SF:<title>HTTP\x20Status\x20404\x20\xe2\x80\x93\x20Not\x20Found</title><st
SF:yle\x20type=\"text/css\">h1\x20{font-family:Tahoma,Arial,sans-serif;col
SF:or:white;background-color:#525D76;font-size:22px;}\x20h2\x20{font-famil
SF:y:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-siz
SF:e:16px;}\x20h3\x20{font-family:Tahoma,Arial,sans-serif;color:white;back
SF:ground-color:#525D76;font-size:14px;}\x20body\x20{font-family:Tahoma,Ar
SF:ial,sans-serif;color:black;background-color:white;}\x20b\x20{font-famil
SF:y:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;}\x20p\x
SF:20{font-family:Tahoma,Arial,sans-serif;background:white;color:black;fon
SF:t-size:12px;}\x20a\x20{color:black;}\x20a\.name\x20{color:black;}\x20\.
SF:line\x20{height:1px;background-color:#525D76;border:none;}</style></hea
SF:d><body>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.07 seconds
                                                                         
```

- Web Server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.64 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50  -x txt,php,html -k 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.64
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
/index.html           (Status: 200) [Size: 1708]
/manager              (Status: 302) [Size: 0] [--> /manager/]
/GettingStarted.html  (Status: 200) [Size: 203]
/Monitoring           (Status: 302) [Size: 0] [--> /Monitoring/]
```