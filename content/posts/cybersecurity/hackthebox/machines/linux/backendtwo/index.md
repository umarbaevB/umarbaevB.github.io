---
title: "[HTB] Machine: BackendTwo"
description: "[HTB] Machine: BackendTwo"
date: 2023-11-06
menu:
  sidebar:
    name: "[HTB] Machine: BackendTwo"
    identifier: htb-machine-BackendTwo
    parent: htb-machines-linux
    weight: 10
hero: images/backendtwo.png
tags: ["HTB"]
---

# BackendTwo
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.162 --min-rate 1000                                                                                
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 18:59 GMT
Nmap scan report for 10.10.11.162 (10.10.11.162)
Host is up (0.18s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 76.04 seconds
```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.11.162 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 19:09 GMT
Nmap scan report for 10.10.11.162 (10.10.11.162)
Host is up (0.19s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea:84:21:a3:22:4a:7d:f9:b5:25:51:79:83:a4:f5:f2 (RSA)
|   256 b8:39:9e:f4:88:be:aa:01:73:2d:10:fb:44:7f:84:61 (ECDSA)
|_  256 22:21:e9:f4:85:90:87:45:16:1f:73:36:41:ee:3b:32 (ED25519)
80/tcp open  http    uvicorn
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     content-type: text/plain; charset=utf-8
|     Connection: close
|     Invalid HTTP request received.
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     date: Fri, 10 Nov 2023 19:08:48 GMT
|     server: uvicorn
|     content-length: 22
|     content-type: application/json
|     Connection: close
|     {"detail":"Not Found"}
|   GetRequest: 
|     HTTP/1.1 200 OK
|     date: Fri, 10 Nov 2023 19:08:35 GMT
|     server: uvicorn
|     content-length: 22
|     content-type: application/json
|     Connection: close
|     {"msg":"UHC Api v2.0"}
|   HTTPOptions: 
|     HTTP/1.1 405 Method Not Allowed
|     date: Fri, 10 Nov 2023 19:08:41 GMT
|     server: uvicorn
|     content-length: 31
|     content-type: application/json
|     Connection: close
|_    {"detail":"Method Not Allowed"}
|_http-server-header: uvicorn
|_http-title: Site doesn't have a title (application/json).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.94%I=7%D=11/10%Time=654E7FEE%P=x86_64-pc-linux-gnu%r(Get
SF:Request,A6,"HTTP/1\.1\x20200\x20OK\r\ndate:\x20Fri,\x2010\x20Nov\x20202
SF:3\x2019:08:35\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:\x2022\r\n
SF:content-type:\x20application/json\r\nConnection:\x20close\r\n\r\n{\"msg
SF:\":\"UHC\x20Api\x20v2\.0\"}")%r(HTTPOptions,BF,"HTTP/1\.1\x20405\x20Met
SF:hod\x20Not\x20Allowed\r\ndate:\x20Fri,\x2010\x20Nov\x202023\x2019:08:41
SF:\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:\x2031\r\ncontent-type:
SF:\x20application/json\r\nConnection:\x20close\r\n\r\n{\"detail\":\"Metho
SF:d\x20Not\x20Allowed\"}")%r(RTSPRequest,76,"HTTP/1\.1\x20400\x20Bad\x20R
SF:equest\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\
SF:x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(FourOhFour
SF:Request,AD,"HTTP/1\.1\x20404\x20Not\x20Found\r\ndate:\x20Fri,\x2010\x20
SF:Nov\x202023\x2019:08:48\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:
SF:\x2022\r\ncontent-type:\x20application/json\r\nConnection:\x20close\r\n
SF:\r\n{\"detail\":\"Not\x20Found\"}")%r(GenericLines,76,"HTTP/1\.1\x20400
SF:\x20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\n
SF:Connection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%
SF:r(DNSVersionBindReqTCP,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nconten
SF:t-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n
SF:Invalid\x20HTTP\x20request\x20received\.")%r(DNSStatusRequestTCP,76,"HT
SF:TP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20cha
SF:rset=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x2
SF:0received\.")%r(SSLSessionReq,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\
SF:r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(TerminalServerCooki
SF:e,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20text/plain
SF:;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20re
SF:quest\x20received\.")%r(TLSSessionReq,76,"HTTP/1\.1\x20400\x20Bad\x20Re
SF:quest\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x
SF:20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 83.95 seconds

```

- Port `80`

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.11.162/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --no-error 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.162/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 401) [Size: 30]
/api                  (Status: 200) [Size: 19]
```