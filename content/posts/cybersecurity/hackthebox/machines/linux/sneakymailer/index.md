---
title: "[HTB] Machine: SneakyMailer"
description: "[HTB] Machine: SneakyMailer"
date: 2023-10-28
menu:
  sidebar:
    name: "[HTB] Machine: SneakyMailer"
    identifier: htb-machine-SneakyMailer
    parent: htb-machines-linux
    weight: 10
hero: images/sneakymailer.png
tags: ["HTB"]
---

# SneakyMailer
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.197 --min-rate 5000                 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-30 17:04 GMT
Warning: 10.10.10.197 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.197 (10.10.10.197)
Host is up (0.17s latency).
Not shown: 43759 filtered tcp ports (no-response), 21769 closed tcp ports (conn-refused)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
143/tcp  open  imap
993/tcp  open  imaps
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 119.98 seconds

```
```
└─$ nmap -Pn -p21,22,25,80,143,993,8080 -sC -sV 10.10.10.197 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-30 17:07 GMT
Nmap scan report for 10.10.10.197 (10.10.10.197)
Host is up (0.28s latency).

PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
22/tcp   open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 57:c9:00:35:36:56:e6:6f:f6:de:86:40:b2:ee:3e:fd (RSA)
|   256 d8:21:23:28:1d:b8:30:46:e2:67:2d:59:65:f0:0a:05 (ECDSA)
|_  256 5e:4f:23:4e:d4:90:8e:e9:5e:89:74:b3:19:0c:fc:1a (ED25519)
25/tcp   open  smtp     Postfix smtpd
|_smtp-commands: debian, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
80/tcp   open  http     nginx 1.14.2
|_http-title: Did not follow redirect to http://sneakycorp.htb
|_http-server-header: nginx/1.14.2
143/tcp  open  imap     Courier Imapd (released 2018)
|_imap-capabilities: QUOTA CAPABILITY completed UIDPLUS UTF8=ACCEPTA0001 ENABLE SORT IMAP4rev1 THREAD=ORDEREDSUBJECT STARTTLS CHILDREN THREAD=REFERENCES ACL2=UNION IDLE NAMESPACE OK ACL
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-05-14T17:14:21
|_Not valid after:  2021-05-14T17:14:21
|_ssl-date: TLS randomness does not represent time
993/tcp  open  ssl/imap Courier Imapd (released 2018)
|_imap-capabilities: QUOTA CAPABILITY completed UIDPLUS UTF8=ACCEPTA0001 ENABLE AUTH=PLAIN SORT IMAP4rev1 THREAD=ORDEREDSUBJECT CHILDREN THREAD=REFERENCES ACL2=UNION IDLE NAMESPACE OK ACL
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
| Subject Alternative Name: email:postmaster@example.com
| Not valid before: 2020-05-14T17:14:21
|_Not valid after:  2021-05-14T17:14:21
|_ssl-date: TLS randomness does not represent time
8080/tcp open  http     nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome to nginx!
|_http-open-proxy: Proxy might be redirecting requests
Service Info: Host:  debian; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 58.51 seconds

```

- Web server

![](./images/1.png)

- Port `8080`

![](./images/2.png)

- `gobuster`
```
└─$ gobuster dir -u http://sneakycorp.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js --no-error
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://sneakycorp.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 185] [--> http://sneakycorp.htb/img/]
/css                  (Status: 301) [Size: 185] [--> http://sneakycorp.htb/css/]
/js                   (Status: 301) [Size: 185] [--> http://sneakycorp.htb/js/]
/vendor               (Status: 301) [Size: 185] [--> http://sneakycorp.htb/vendor/]
/pypi                 (Status: 301) [Size: 185] [--> http://sneakycorp.htb/pypi/]
```
```
└─$ gobuster dir -u http://sneakycorp.htb:8080/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js --no-error 

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://sneakycorp.htb:8080/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 612]

```

- `vhosts`
```
└─$ wfuzz -u http://sneakycorp.htb/ -H 'Host: FUZZ.sneakycorp.htb' -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --hw 12
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://sneakycorp.htb/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000019:   200        340 L    989 W      13737 Ch    "dev"    
```

- `ftp`
```
└─$ ftp anonymous@sneakycorp.htb
Connected to sneakycorp.htb.
220 (vsFTPd 3.0.3)
530 Permission denied.
ftp: Login failed
```