---
title: "[HTB] Machine: Remote"
description: "[HTB] Machine: Remote"
date: 2023-06-29
menu:
  sidebar:
    name: "[HTB] Machine: Remote"
    identifier: htb-machine-remote
    parent: htb-machines-windows
    weight: 10
hero: images/remote.png
tags: ["HTB"]
---

# Remote
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV -Pn 10.10.10.180
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-02 18:25 BST
Nmap scan report for 10.10.10.180 (10.10.10.180)
Host is up (0.13s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp  open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2049/tcp open  mountd        1-3 (RPC #100005)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -31s
| smb2-time: 
|   date: 2023-07-02T17:26:07
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 198.64 seconds
```
- Web server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.180 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x asp,aspx --no-error
===============================================================
Gobuster v3.5                      
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)                                                                                                                                                                               
===============================================================
[+] Url:                     http://10.10.10.180
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              asp,aspx
[+] Timeout:                 10s
===============================================================
2023/07/02 19:58:02 Starting gobuster in directory enumeration mode
===============================================================
/.aspx                (Status: 200) [Size: 6693]
/contact              (Status: 200) [Size: 7880]
/contact.aspx         (Status: 200) [Size: 7880]
/blog.aspx            (Status: 200) [Size: 5001]
/home.aspx            (Status: 200) [Size: 6703]
/blog                 (Status: 200) [Size: 5011]
/home                 (Status: 200) [Size: 6703]
/products             (Status: 200) [Size: 5328]
/products.aspx        (Status: 200) [Size: 5328]
/default.aspx         (Status: 200) [Size: 6693]
/people.aspx          (Status: 200) [Size: 6739]
/people               (Status: 200) [Size: 6739]
/product.aspx         (Status: 500) [Size: 3420]
/product              (Status: 500) [Size: 3420]
/Default.aspx         (Status: 200) [Size: 6693]
/Home                 (Status: 200) [Size: 6703]
/Home.aspx            (Status: 200) [Size: 6703]
/Products             (Status: 200) [Size: 5338]
/Products.aspx        (Status: 200) [Size: 5338]
/Contact              (Status: 200) [Size: 7890]
/Contact.aspx         (Status: 200) [Size: 7890]
/install              (Status: 302) [Size: 126] [--> /umbraco/]
/Blog.aspx            (Status: 200) [Size: 5011]
/Blog                 (Status: 200) [Size: 5011]
/People.aspx          (Status: 200) [Size: 6749]
/People               (Status: 200) [Size: 6749]
/Product              (Status: 500) [Size: 3420]
/Product.aspx         (Status: 500) [Size: 3420]
```
- Seems like there is a [UMBRACO CMS](https://umbraco.com/), based on `/install` redirect

![](./images/2.png)

- We have `Remote Code Execution` exploits
  - But I need to find the version and creds to be able to perform them

![](./images/3.png)

- Nothing found on `SMB` and `FTP`

![](./images/4.png)
