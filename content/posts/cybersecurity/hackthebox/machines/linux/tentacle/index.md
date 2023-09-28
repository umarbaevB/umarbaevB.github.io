---
title: "[HTB] Machine: Tentacle"
description: "[HTB] Machine: Tentacle"
date: 2023-09-27
menu:
  sidebar:
    name: "[HTB] Machine: Tentacle"
    identifier: htb-machine-Tentacle
    parent: htb-machines-linux
    weight: 10
hero: images/tentacle.png
tags: ["HTB"]
---

# Tentacle
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.224 -T4 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-26 20:09 BST
Nmap scan report for 10.10.10.224 (10.10.10.224)
Host is up (0.093s latency).
Not shown: 65495 filtered tcp ports (no-response), 36 filtered tcp ports (host-unreach)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
88/tcp   open  kerberos-sec
3128/tcp open  squid-http

Nmap done: 1 IP address (1 host up) scanned in 39.58 seconds

```
```
└─$ nmap -Pn -p22,53,88,3128 -sC -sV 10.10.10.224 -T4 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-26 20:12 BST
Nmap scan report for 10.10.10.224 (10.10.10.224)
Host is up (0.12s latency).

PORT     STATE SERVICE      VERSION
22/tcp   open  ssh          OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 8d:dd:18:10:e5:7b:b0:da:a3:fa:14:37:a7:52:7a:9c (RSA)
|   256 f6:a9:2e:57:f8:18:b6:f4:ee:03:41:27:1e:1f:93:99 (ECDSA)
|_  256 04:74:dd:68:79:f4:22:78:d8:ce:dd:8b:3e:8c:76:3b (ED25519)
53/tcp   open  domain       ISC BIND 9.11.20 (RedHat Enterprise Linux 8)
| dns-nsid: 
|_  bind.version: 9.11.20-RedHat-9.11.20-5.el8
88/tcp   open  kerberos-sec MIT Kerberos (server time: 2023-09-26 19:11:56Z)
3128/tcp open  http-proxy   Squid http proxy 4.11
|_http-server-header: squid/4.11
|_http-title: ERROR: The requested URL could not be retrieved
Service Info: Host: REALCORP.HTB; OS: Linux; CPE: cpe:/o:redhat:enterprise_linux:8

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.44 seconds

```
```
└─$ sudo nmap -Pn -sU 10.10.10.224 -T4 --min-rate 5000
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-26 20:13 BST
Nmap scan report for 10.10.10.224 (10.10.10.224)
Host is up (0.10s latency).
Not shown: 991 open|filtered udp ports (no-response)
PORT      STATE    SERVICE
53/udp    open     domain
88/udp    open     kerberos-sec
123/udp   open     ntp
17787/udp filtered unknown
20560/udp filtered unknown
21016/udp filtered unknown
40019/udp filtered unknown
44185/udp filtered unknown
49179/udp filtered unknown

```
- `dns`
```
└─$ dig axfr @10.10.10.224
;; communications error to 10.10.10.224#53: timed out
;; communications error to 10.10.10.224#53: timed out

; <<>> DiG 9.18.16-1-Debian <<>> axfr @10.10.10.224
; (1 server found)
;; global options: +cmd
;; Query time: 175 msec
;; SERVER: 10.10.10.224#53(10.10.10.224) (UDP)
;; WHEN: Wed Sep 27 17:09:26 BST 2023
;; MSG SIZE  rcvd: 56

```
```
└─$ dig realcorp.htb @10.10.10.224

; <<>> DiG 9.18.16-1-Debian <<>> realcorp.htb @10.10.10.224
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20726
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 1d3b122485d1c4e6706e605e651453ed994c22dca6111a8b (good)
;; QUESTION SECTION:
;realcorp.htb.                  IN      A

;; AUTHORITY SECTION:
realcorp.htb.           86400   IN      SOA     realcorp.htb. root.realcorp.htb. 199609206 28800 7200 2419200 86400

;; Query time: 91 msec
;; SERVER: 10.10.10.224#53(10.10.10.224) (UDP)
;; WHEN: Wed Sep 27 17:11:02 BST 2023
;; MSG SIZE  rcvd: 110

```
```
└─$ dig axfr realcorp.htb @10.10.10.224

; <<>> DiG 9.18.16-1-Debian <<>> axfr realcorp.htb @10.10.10.224
;; global options: +cmd
; Transfer failed.
```
- `dnsenum`
```
└─$ dnsenum --dnsserver 10.10.10.224 -f /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt realcorp.htb
dnsenum VERSION:1.2.6

-----   realcorp.htb   -----                                                                                                                                                                                                                

Host's addresses:                                                                                                                                                                                                                           
__________________                                                                                                                                                                                                                          
Name Servers:                                                                                                                                                                                                                               
______________                                                                                                                                                                                                                              

ns.realcorp.htb.                         259200   IN    A        10.197.243.77

Mail (MX) Servers:                                                                                                                                                                                                                          
___________________                                                                                                                                                                                                                           
Trying Zone Transfers and getting Bind Versions:                                                                                                                                                                                            
_________________________________________________                                                                                                                                                                                           

unresolvable name: ns.realcorp.htb at /usr/bin/dnsenum line 900.                                                                                                                                                                            
        
Trying Zone Transfer for realcorp.htb on ns.realcorp.htb ... 
AXFR record query failed: no nameservers
    
Brute forcing with /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt:                                                                                                                                                     
________________________________________________________________________________________                                                                                                                                         
ns.realcorp.htb.                         259200   IN    A        10.197.243.77
proxy.realcorp.htb.                      259200   IN    CNAME    ns.realcorp.htb.
ns.realcorp.htb.                         259200   IN    A        10.197.243.77
wpad.realcorp.htb.                       259200   IN    A        10.197.243.31
```

- `kerbrute`
```
└─$ ./kerbrute userenum -d realcorp.htb --dc realcorp.htb /usr/share/seclists/Usernames/cirt-default-usernames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 09/27/23 - Ronnie Flathers @ropnop

2023/09/27 17:24:12 >  Using KDC(s):
2023/09/27 17:24:12 >   realcorp.htb:88

2023/09/27 17:24:27 >  Done! Tested 828 usernames (0 valid) in 15.460 seconds
```

- `ntp`
```
└─$ sudo nmap -Pn --script "ntp*" -sU -sV -p123 realcorp.htb
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-27 17:27 BST
Nmap scan report for realcorp.htb (10.10.10.224)
Host is up (0.13s latency).

PORT    STATE SERVICE VERSION
123/udp open  ntp     NTP v4 (secondary server)
| ntp-info: 
|_  receive time stamp: 2023-09-27T16:26:39

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.67 seconds
```
```
└─$ date; ntpdate -q 10.10.10.224
Wed Sep 27 05:30:03 PM BST 2023
2023-09-27 17:29:22.601161 (+0100) -40.729112 +/- 0.083173 10.10.10.224 s10 no-leap
```

- `squid` version `4.11`
  - `srv01.realcorp.htb`
  - user: `j.nakazawa@realcorp.htb` 

![](./images/1.png)

## Foothold
- Let's check if `j.nakazawa` is `AS-REP-Roastable` 
```
└─$ impacket-GetNPUsers -no-pass -dc-ip 10.10.10.224 realcorp.htb/j.nakazawa  
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Getting TGT for j.nakazawa
$krb5asrep$18$j.nakazawa$REALCORP.HTB$0e2ddaebe3ed76c6aa85e8bc$7aa604e7c0c0c585c6605f846cfce4f16b40d26cac17809f054ba04226ca479e19e10c81f1e62a7fe1e50311adb70d48699b9ce5f92d147affb275ee2978fd6bd3ad79c392a034d5b1befa2543592e0855195eec320da94782b579a0e6be3d4e3c63b95bb511216e84e22ebe6bab9fbb82c6ec7a92580215d47f9a5e7b65924d358d06f16c6a55aa940f859302c5fa51cd8a50d949e0b2168f0a9bf676cf850efb5fcae6b71eaa3c10948dd357aa887b2d69581f54200448a949e3a5b47c76f0274092f244f8be56f50ba7a3af76d36450aae88e8daa3c4d77f943e19f
```
- Can't crack it
```
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash         
Using default input encoding: UTF-8
Loaded 1 password hash (HMAC-SHA256 [password is key, SHA256 256/256 AVX2 8x])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:01 DONE (2023-09-27 17:48) 0g/s 7924Kp/s 7924Kc/s 7924KC/s !SkicA!..*7¡Vamos!
Session completed.
```

- We saw [squid-proxy](http://www.squid-cache.org/) on port `3128 `
  - https://book.hacktricks.xyz/network-services-pentesting/3128-pentesting-squid
  - Let's try enumerating it
  - Let's first create a config
```
strict_chain
proxy_dns
[ProxyList]
http 10.10.10.224 3128
```

- Now let's `nmap` 
  - First I tried `10.197.243.77` and `10.197.243.31` (ips from `dnsenum`), but it failed
  - Then I tried `127.0.0.1` and had the results

```
└─$ sudo proxychains -q -f proxy-to-squid.conf nmap --top-ports 1000 -sT -Pn 127.0.0.1
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-28 16:45 BST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.32s latency).
Not shown: 994 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
88/tcp   open  kerberos-sec
464/tcp  open  kpasswd5
749/tcp  open  kerberos-adm
3128/tcp open  squid-http

Nmap done: 1 IP address (1 host up) scanned in 305.98 seconds
```

- Nothing interesting, except the same `squid` proxy
  - Let's chain the proxies
```
strict_chain
proxy_dns
[ProxyList]
http 10.10.10.224 3128
http 127.0.0.1 3128
```

- Let's `nmap` again
  - `10.197.243.77` works
  - But `10.197.243.31` fails
```
└─$ sudo proxychains -q -f proxy-to-squid.conf nmap --top-ports 1000 -sT -Pn 10.197.243.77
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-28 16:54 BST
Nmap scan report for 10.197.243.77 (10.197.243.77)
Host is up (0.54s latency).
Not shown: 994 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
88/tcp   open  kerberos-sec
464/tcp  open  kpasswd5
749/tcp  open  kerberos-adm
3128/tcp open  squid-http

Nmap done: 1 IP address (1 host up) scanned in 555.24 seconds
```
- The same process again
  - Add another proxy in our config
    - `http 10.197.243.77 3128`
  - So current chain is `10.10.10.224 –> 127.0.0.1 –> 10.197.243.77`
  - And run `nmap` again on `10.197.243.31`
```
└─$ sudo proxychains -q -f proxy-to-squid.conf nmap --top-ports 1000 -sT -Pn 10.197.243.31
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-28 17:07 BST
Nmap scan report for 10.197.243.31 (10.197.243.31)
Host is up (0.65s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
80/tcp   open  http
88/tcp   open  kerberos-sec
464/tcp  open  kpasswd5
749/tcp  open  kerberos-adm
3128/tcp open  squid-http

Nmap done: 1 IP address (1 host up) scanned in 654.09 seconds

```
- Now we see port `80`
  - I opened `firefox` as `proxychains -f proxy-to-squid.conf firefox`
  - Visiting http://10.197.243.31/ shows default page
  - While http://wpad.realcorp.htb/ results in `403`

![](./images/2.png)

![](./images/3.png)

- But if we check for http://wpad.realcorp.htb/wpad.dat
  - https://book.hacktricks.xyz/generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks#wpad

```
function FindProxyForURL(url, host) {
    if (dnsDomainIs(host, "realcorp.htb"))
        return "DIRECT";
    if (isInNet(dnsResolve(host), "10.197.243.0", "255.255.255.0"))
        return "DIRECT"; 
    if (isInNet(dnsResolve(host), "10.241.251.0", "255.255.255.0"))
        return "DIRECT"; 
 
    return "PROXY proxy.realcorp.htb:3128";
}
```

- We have another subnet
  - Let's scan it 
```
└─$ for i in {1..254}; do (dig +noall +answer @10.10.10.224 -x 10.241.251.$i &); done
113.251.241.10.in-addr.arpa. 259200 IN  PTR     srvpod01.realcorp.htb.
```
- We have another host
  - Let's `nmap` it
```
└─$ sudo proxychains -q -f proxy-to-squid.conf nmap --top-ports 1000 -sT -Pn 10.241.251.113
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-28 17:36 BST
Nmap scan report for srvpod01.realcorp.htb (10.241.251.113)
Host is up (0.63s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
25/tcp open  smtp

Nmap done: 1 IP address (1 host up) scanned in 646.00 seconds
```
```
└─$ sudo proxychains -q -f proxy-to-squid.conf nmap -p25 -sC -sV -sT -Pn 10.241.251.113
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-28 17:47 BST
Nmap scan report for srvpod01.realcorp.htb (10.241.251.113)
Host is up (1.2s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    OpenSMTPD
| smtp-commands: smtp.realcorp.htb Hello srvpod01.realcorp.htb [10.241.251.1], pleased to meet you, 8BITMIME, ENHANCEDSTATUSCODES, SIZE 36700160, DSN, HELP
|_ 2.0.0 This is OpenSMTPD 2.0.0 To report bugs in the implementation, please contact bugs@openbsd.org 2.0.0 with full details 2.0.0 End of HELP info
Service Info: Host: smtp.realcorp.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.62 seconds

```


## User

## Root