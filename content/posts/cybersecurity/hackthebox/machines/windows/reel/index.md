---
title: "[HTB] Machine: Reel"
description: "[HTB] Machine: Reel"
date: 2023-09-11
menu:
  sidebar:
    name: "[HTB] Machine: Reel"
    identifier: htb-machine-Reel
    parent: htb-machines-windows
    weight: 10
hero: images/reel.png
tags: ["HTB"]
---

# Reel
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.77 -T4                                                                
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-13 18:39 BST
Nmap scan report for 10.10.10.77 (10.10.10.77)
Host is up (0.093s latency).
Not shown: 65527 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
25/tcp    open  smtp
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
49159/tcp open  unknown
```
```
└─$ nmap -Pn -p21,22,25,135,139,445,593 -sC -sV 10.10.10.77 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-13 18:43 BST
Stats: 0:03:29 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.90% done; ETC: 18:46 (0:00:00 remaining)
Nmap scan report for 10.10.10.77 (10.10.10.77)
Host is up (0.14s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_05-29-18  12:19AM       <DIR>          documents
| ftp-syst: 
|_  SYST: Windows_NT
22/tcp  open  ssh         OpenSSH 7.6 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:20:c3:bd:16:cb:a2:9c:88:87:1d:6c:15:59:ed:ed (RSA)
|   256 23:2b:b8:0a:8c:1c:f4:4d:8d:7e:5e:64:58:80:33:45 (ECDSA)
|_  256 ac:8b:de:25:1d:b7:d8:38:38:9b:9c:16:bf:f6:3f:ed (ED25519)
25/tcp  open  smtp?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, X11Probe: 
|     220 Mail Service ready
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|   Hello: 
|     220 Mail Service ready
|     EHLO Invalid domain address.
|   Help: 
|     220 Mail Service ready
|     DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
|   SIPOptions: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|   TerminalServerCookie: 
|     220 Mail Service ready
|_    sequence of commands
| smtp-commands: REEL, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
135/tcp open  msrpc       Microsoft Windows RPC
139/tcp open  netbios-ssn Microsoft Windows netbios-ssn
445/tcp open             Windows Server 2012 R2 Standard 9600 microsoft-ds (workgroup: HTB)
593/tcp open  ncacn_http  Microsoft Windows RPC over HTTP 1.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port25-TCP:V=7.94%I=7%D=9/13%Time=6501F4AB%P=x86_64-pc-linux-gnu%r(NULL
SF:,18,"220\x20Mail\x20Service\x20ready\r\n")%r(Hello,3A,"220\x20Mail\x20S
SF:ervice\x20ready\r\n501\x20EHLO\x20Invalid\x20domain\x20address\.\r\n")%
SF:r(Help,54,"220\x20Mail\x20Service\x20ready\r\n211\x20DATA\x20HELO\x20EH
SF:LO\x20MAIL\x20NOOP\x20QUIT\x20RCPT\x20RSET\x20SAML\x20TURN\x20VRFY\r\n"
SF:)%r(GenericLines,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad\x20s
SF:equence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\r
SF:\n")%r(GetRequest,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad\x20
SF:sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\
SF:r\n")%r(HTTPOptions,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad\x
SF:20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20command
SF:s\r\n")%r(RTSPRequest,54,"220\x20Mail\x20Service\x20ready\r\n503\x20Bad
SF:\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20comma
SF:nds\r\n")%r(RPCCheck,18,"220\x20Mail\x20Service\x20ready\r\n")%r(DNSVer
SF:sionBindReqTCP,18,"220\x20Mail\x20Service\x20ready\r\n")%r(DNSStatusReq
SF:uestTCP,18,"220\x20Mail\x20Service\x20ready\r\n")%r(SSLSessionReq,18,"2
SF:20\x20Mail\x20Service\x20ready\r\n")%r(TerminalServerCookie,36,"220\x20
SF:Mail\x20Service\x20ready\r\n503\x20Bad\x20sequence\x20of\x20commands\r\
SF:n")%r(TLSSessionReq,18,"220\x20Mail\x20Service\x20ready\r\n")%r(Kerbero
SF:s,18,"220\x20Mail\x20Service\x20ready\r\n")%r(SMBProgNeg,18,"220\x20Mai
SF:l\x20Service\x20ready\r\n")%r(X11Probe,18,"220\x20Mail\x20Service\x20re
SF:ady\r\n")%r(FourOhFourRequest,54,"220\x20Mail\x20Service\x20ready\r\n50
SF:3\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\
SF:x20commands\r\n")%r(LPDString,18,"220\x20Mail\x20Service\x20ready\r\n")
SF:%r(LDAPSearchReq,18,"220\x20Mail\x20Service\x20ready\r\n")%r(LDAPBindRe
SF:q,18,"220\x20Mail\x20Service\x20ready\r\n")%r(SIPOptions,162,"220\x20Ma
SF:il\x20Service\x20ready\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n5
SF:03\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of
SF:\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\
SF:x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20comman
SF:ds\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequenc
SF:e\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x20commands\r\n503\
SF:x20Bad\x20sequence\x20of\x20commands\r\n503\x20Bad\x20sequence\x20of\x2
SF:0commands\r\n");
Service Info: Host: REEL; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb-os-discovery: 
|   OS: Windows Server 2012 R2 Standard 9600 (Windows Server 2012 R2 Standard 6.3)
|   OS CPE: cpe:/o:microsoft:windows_server_2012::-
|   Computer name: REEL
|   NetBIOS computer name: REEL\x00
|   Domain name: HTB.LOCAL
|   Forest name: HTB.LOCAL
|   FQDN: REEL.HTB.LOCAL
|_  System time: 2023-09-13T18:45:17+01:00
| smb2-time: 
|   date: 2023-09-13T17:45:18
|_  start_date: 2023-09-13T17:37:37
| smb2-security-mode: 
|   3:0:2: 
|_    Message signing enabled and required
|_clock-skew: mean: -20m36s, deviation: 34m36s, median: -38s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 212.37 seconds

```
- `smb`
```
└─$ smbclient -N -L //10.10.10.77
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.77 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
- `ftp`
```
└─$ ftp anonymous@10.10.10.77  
Connected to 10.10.10.77.
220 Microsoft FTP Service
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||41001|)
125 Data connection already open; Transfer starting.
05-29-18  12:19AM       <DIR>          documents
226 Transfer complete.
ftp> cd Documents
250 CWD command successful.
ftp> ls
229 Entering Extended Passive Mode (|||41002|)
125 Data connection already open; Transfer starting.
05-29-18  12:19AM                 2047 AppLocker.docx
05-28-18  02:01PM                  124 readme.txt
10-31-17  10:13PM                14581 Windows Event Forwarding.docx
ftp> prompt
Interactive mode off.
ftp> mget *
...
```
```
└─$ cat readme.txt                                                         
please email me any rtf format procedures - I'll review and convert.

new format / converted documents will be saved here.
```
```
└─$ exiftool *.docx                                                        
======== AppLocker.docx
ExifTool Version Number         : 12.64
File Name                       : AppLocker.docx
Directory                       : .
File Size                       : 2.0 kB
File Modification Date/Time     : 2018:05:29 00:19:48+01:00
File Access Date/Time           : 2023:09:13 18:49:30+01:00
File Inode Change Date/Time     : 2023:09:13 18:49:30+01:00
File Permissions                : -rw-r--r--
File Type                       : DOCX
File Type Extension             : docx
MIME Type                       : application/vnd.openxmlformats-officedocument.wordprocessingml.document
Zip Required Version            : 20
Zip Bit Flag                    : 0x0008
Zip Compression                 : Deflated
Zip Modify Date                 : 2018:05:29 00:19:50
Zip CRC                         : 0x3cdd8b4f
Zip Compressed Size             : 166
Zip Uncompressed Size           : 284
Zip File Name                   : _rels/.rels
======== Windows Event Forwarding.docx
ExifTool Version Number         : 12.64
File Name                       : Windows Event Forwarding.docx
Directory                       : .
File Size                       : 15 kB
File Modification Date/Time     : 2017:10:31 21:13:23+00:00
File Access Date/Time           : 2023:09:13 18:49:32+01:00
File Inode Change Date/Time     : 2023:09:13 18:49:32+01:00
File Permissions                : -rw-r--r--
File Type                       : DOCX
File Type Extension             : docx
MIME Type                       : application/vnd.openxmlformats-officedocument.wordprocessingml.document
Zip Required Version            : 20
Zip Bit Flag                    : 0x0006
Zip Compression                 : Deflated
Zip Modify Date                 : 1980:01:01 00:00:00
Zip CRC                         : 0x82872409
Zip Compressed Size             : 385
Zip Uncompressed Size           : 1422
Zip File Name                   : [Content_Types].xml
Creator                         : nico@megabank.com
Revision Number                 : 4
Create Date                     : 2017:10:31 18:42:00Z
Modify Date                     : 2017:10:31 18:51:00Z
Template                        : Normal.dotm
Total Edit Time                 : 5 minutes
Pages                           : 2
Words                           : 299
Characters                      : 1709
Application                     : Microsoft Office Word
Doc Security                    : None
Lines                           : 14
Paragraphs                      : 4
Scale Crop                      : No
Heading Pairs                   : Title, 1
Titles Of Parts                 : 
Company                         : 
Links Up To Date                : No
Characters With Spaces          : 2004
Shared Doc                      : No
Hyperlinks Changed              : No
App Version                     : 14.0000
    2 image files read
```

![](./images/1.png)

- `smtp`
  - We can do it via `nmap`
  - Or `smtp-user-enum`
```
└─$ nmap -Pn --script smtp-enum* -p25 10.10.10.77
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-13 18:51 BST
Nmap scan report for 10.10.10.77 (10.10.10.77)
Host is up (0.092s latency).

PORT   STATE SERVICE
25/tcp open  smtp
| smtp-enum-users: 
|   root
|   admin
|   administrator
|   webadmin
|   sysadmin
|   netadmin
|   guest
|   user
|   web
|_  test

```

## Foothold


## Root