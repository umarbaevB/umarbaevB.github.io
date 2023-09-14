---
title: "[HTB] Machine: StreamIO"
description: "[HTB] Machine: StreamIO"
date: 2023-09-11
menu:
  sidebar:
    name: "[HTB] Machine: StreamIO"
    identifier: htb-machine-StreamIO
    parent: htb-machines-windows
    weight: 10
hero: images/streamio.png
tags: ["HTB"]
---

# StreamIO
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.158 -T4 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-13 19:18 BST
Nmap scan report for 10.10.11.158 (10.10.11.158)
Host is up (0.10s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49701/tcp open  unknown
55287/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 32.55 seconds

```
```
└─$ nmap -Pn -p53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389 -sC -sV 10.10.11.158 -T4 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-13 19:19 BST
Nmap scan report for 10.10.11.158 (10.10.11.158)
Host is up (0.16s latency).

PORT     STATE    SERVICE          VERSION
53/tcp   open     domain           Simple DNS Plus
80/tcp   open     http             Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp   open     kerberos-sec     Microsoft Windows Kerberos (server time: 2023-09-14 01:19:00Z)
135/tcp  open     msrpc            Microsoft Windows RPC
139/tcp  open     netbios-ssn      Microsoft Windows netbios-ssn
389/tcp  open     ldap             Microsoft Windows Active Directory LDAP (Domain: streamIO.htb0., Site: Default-First-Site-Name)
443/tcp  open     ssl/http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
| ssl-cert: Subject: commonName=streamIO/countryName=EU
| Subject Alternative Name: DNS:streamIO.htb, DNS:watch.streamIO.htb
| Not valid before: 2022-02-22T07:03:28
|_Not valid after:  2022-03-24T07:03:28
|_ssl-date: 2023-09-14T01:19:51+00:00; +6h59m23s from scanner time.
|_http-title: Not Found
| tls-alpn: 
|_  http/1.1
445/tcp  open     microsoft-ds?
464/tcp  open     kpasswd5?
593/tcp  open     ncacn_http       Microsoft Windows RPC over HTTP 1.0
636/tcp  open     tcpwrapped
3268/tcp filtered globalcatLDAP
3269/tcp filtered globalcatLDAPssl
5985/tcp open     http             Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open     mc-nmf           .NET Message Framing
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 6h59m22s, deviation: 0s, median: 6h59m21s
| smb2-time: 
|   date: 2023-09-14T01:19:15
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.52 seconds
```
- `smb`
```
└─$ smbclient -N -L //10.10.11.158
session setup failed: NT_STATUS_ACCESS_DENIED
```
- `dns`
```
└─$ dig @10.10.11.158 streamIO.htb                                                                                          

; <<>> DiG 9.18.16-1-Debian <<>> @10.10.11.158 streamIO.htb
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13035
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;streamIO.htb.                  IN      A

;; ANSWER SECTION:
streamIO.htb.           600     IN      A       10.10.11.158

;; Query time: 100 msec
;; SERVER: 10.10.11.158#53(10.10.11.158) (UDP)
;; WHEN: Wed Sep 13 19:26:32 BST 2023
;; MSG SIZE  rcvd: 57

```
```
└─$ dig axfr @10.10.11.158 streamIO.htb

; <<>> DiG 9.18.16-1-Debian <<>> axfr @10.10.11.158 streamIO.htb
; (1 server found)
;; global options: +cmd
; Transfer failed.
```
- `vhosts`
```
└─$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u https://watch.streamIO.htb -H 'Host: FUZZ.streamIO.htb' --hh 315
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://watch.streamIO.htb/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000003520:   200        78 L     245 W      2829 Ch     "watch"                                                                                                                                                                    
000037212:   400        6 L      26 W       334 Ch      "*"                                                                                                                                                                        

Total time: 1237.048
Processed Requests: 100000
Filtered Requests: 99998
Requests/sec.: 80.83756

```
```
└─$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://watch.streamIO.htb -H 'Host: FUZZ.streamIO.htb' --hh 703
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://watch.streamIO.htb/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000037212:   400        6 L      26 W       334 Ch      "*" 

Total time: 1021.031
Processed Requests: 100000
Filtered Requests: 99999
Requests/sec.: 66.52712
```
- 