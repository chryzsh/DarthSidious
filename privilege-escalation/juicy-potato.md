---
description: Using the juicy potato exploit for privilege escalation
---

# Juicy Potato

{% embed data="{\"url\":\"https://ohpe.github.io/juicy-potato/\",\"type\":\"link\",\"title\":\"Juicy Potato \(abusing the golden privileges\)\",\"description\":\"A sugared version of RottenPotatoNG, with a bit of juice, i.e. another Local Privilege Escalation tool, from a Windows Service Accounts to NT AUTHORITY\\\\SYSTEM.\"}" %}

Juicy potato is basically a weaponized version of the RottenPotato exploit that exploits the way Microsoft handles tokens. Through this, we achieve privilege escalation. 

### How does it work?

 I will admit I am not an expert in Windows internals, but I have tried to understand how this exploit works. 

A CLSID is a globally unique identifier that identifies a COM class object.

### How to use

* As we can see, we are on Windows 10 Enterprise 1709, but the OS shouldn't matter.
* We need to have a shell as a service account. For demo purposes I used`nt authority\local service` 
* The only real requirement however, is that the account has the `SeAssignPrimaryTokenPrivilege` and/or `SeImpersonatePrivilege` which most service accounts do have.
* To try this yourself, you can open a shell as the service account using [psexec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) from Microsoft Sysinternals as displayed in the screenshot below.
* We pick a CLSID from [here](https://ohpe.github.io/juicy-potato/CLSID/Windows_10_Enterprise/).
* Now we run the exploit by specifiying a COM port of 1337, and executing the process cmd.exe trying both techniques `CreateProcessWithTokenW`,  `CreateProcessAsUser`
* A shell pops as `nt authority\system`

![](../.gitbook/assets/image%20%281%29.png)

### Mitigation

According to the creators, the actual solution is to protect sensitive accounts and applications which run under the `* SERVICE` accounts.

