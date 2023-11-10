---
title: "[HTB] Machine: Backend"
description: "[HTB] Machine: Backend"
date: 2023-11-09
menu:
  sidebar:
    name: "[HTB] Machine: Backend"
    identifier: htb-machine-Backend
    parent: htb-machines-linux
    weight: 10
hero: images/backend.png
tags: ["HTB"]
---

# Backend
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.161 --min-rate 1000                                                                                                             
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 18:46 GMT
Nmap scan report for 10.10.11.161 (10.10.11.161)
Host is up (0.18s latency).
Not shown: 65529 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
8527/tcp  filtered unknown
20524/tcp filtered unknown
30552/tcp filtered unknown
51149/tcp filtered unknown

```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.11.161 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 18:51 GMT
Nmap scan report for 10.10.11.161 (10.10.11.161)
Host is up (0.19s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea:84:21:a3:22:4a:7d:f9:b5:25:51:79:83:a4:f5:f2 (RSA)
|   256 b8:39:9e:f4:88:be:aa:01:73:2d:10:fb:44:7f:84:61 (ECDSA)
|_  256 22:21:e9:f4:85:90:87:45:16:1f:73:36:41:ee:3b:32 (ED25519)
80/tcp open  http    uvicorn
|_http-server-header: uvicorn
|_http-title: Site doesn't have a title (application/json).
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     content-type: text/plain; charset=utf-8
|     Connection: close
|     Invalid HTTP request received.
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     date: Fri, 10 Nov 2023 23:02:09 GMT
|     server: uvicorn
|     content-length: 22
|     content-type: application/json
|     Connection: close
|     {"detail":"Not Found"}
|   GetRequest: 
|     HTTP/1.1 200 OK
|     date: Fri, 10 Nov 2023 23:01:55 GMT
|     server: uvicorn
|     content-length: 29
|     content-type: application/json
|     Connection: close
|     {"msg":"UHC API Version 1.0"}
|   HTTPOptions: 
|     HTTP/1.1 405 Method Not Allowed
|     date: Fri, 10 Nov 2023 23:02:02 GMT
|     server: uvicorn
|     content-length: 31
|     content-type: application/json
|     Connection: close
|_    {"detail":"Method Not Allowed"}
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.94%I=7%D=11/10%Time=654E7BD1%P=x86_64-pc-linux-gnu%r(Get
SF:Request,AD,"HTTP/1\.1\x20200\x20OK\r\ndate:\x20Fri,\x2010\x20Nov\x20202
SF:3\x2023:01:55\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:\x2029\r\n
SF:content-type:\x20application/json\r\nConnection:\x20close\r\n\r\n{\"msg
SF:\":\"UHC\x20API\x20Version\x201\.0\"}")%r(HTTPOptions,BF,"HTTP/1\.1\x20
SF:405\x20Method\x20Not\x20Allowed\r\ndate:\x20Fri,\x2010\x20Nov\x202023\x
SF:2023:02:02\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:\x2031\r\ncon
SF:tent-type:\x20application/json\r\nConnection:\x20close\r\n\r\n{\"detail
SF:\":\"Method\x20Not\x20Allowed\"}")%r(RTSPRequest,76,"HTTP/1\.1\x20400\x
SF:20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nCo
SF:nnection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(
SF:FourOhFourRequest,AD,"HTTP/1\.1\x20404\x20Not\x20Found\r\ndate:\x20Fri,
SF:\x2010\x20Nov\x202023\x2023:02:09\x20GMT\r\nserver:\x20uvicorn\r\nconte
SF:nt-length:\x2022\r\ncontent-type:\x20application/json\r\nConnection:\x2
SF:0close\r\n\r\n{\"detail\":\"Not\x20Found\"}")%r(GenericLines,76,"HTTP/1
SF:\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20charset
SF:=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20rec
SF:eived\.")%r(DNSVersionBindReqTCP,76,"HTTP/1\.1\x20400\x20Bad\x20Request
SF:\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20clo
SF:se\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(DNSStatusRequest
SF:TCP,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20text/pla
SF:in;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20
SF:request\x20received\.")%r(SSLSessionReq,76,"HTTP/1\.1\x20400\x20Bad\x20
SF:Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:
SF:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(TerminalS
SF:erverCookie,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20
SF:text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20
SF:HTTP\x20request\x20received\.")%r(TLSSessionReq,76,"HTTP/1\.1\x20400\x2
SF:0Bad\x20Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nCon
SF:nection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.90 seconds

```

- Port `80`

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.11.161/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt --no-error      
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.161/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 401) [Size: 30]
/api                  (Status: 200) [Size: 20]
```
