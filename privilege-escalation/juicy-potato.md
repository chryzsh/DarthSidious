---
description: Using the juicy potato exploit for privilege escalation
---

# Juicy Potato

{% embed data="{\"url\":\"https://ohpe.github.io/juicy-potato/\",\"type\":\"link\",\"title\":\"Juicy Potato \(abusing the golden privileges\)\",\"description\":\"A sugared version of RottenPotatoNG, with a bit of juice, i.e. another Local Privilege Escalation tool, from a Windows Service Accounts to NT AUTHORITY\\\\SYSTEM.\"}" %}

Juicy potato is basically a weaponized version of the RottenPotato exploit that exploits the way Microsoft handles tokens. Through this, we achieve privilege escalation. 

### How does it work?

I will admit I am not an expert in Windows internals, but I have tried to understand how this exploit works. A CLSID is a globally unique identifier that identifies a COM class object. The exploit allows us to escalate from service accounts in session 0 to SYSTEM. More to come once I understand it all!

### How to use

As we can see, we are on Windows 10 Enterprise 1709, but the OS shouldn't matter. We need to have a shell as a service account. For demo purposes I used`nt authority\local service` 

The only real requirement however, is that the account has the `SeAssignPrimaryTokenPrivilege` and/or `SeImpersonatePrivilege` which most service accounts do have.

To try this yourself, you can open a shell as the service account using [psexec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) from Microsoft Sysinternals as displayed in the screenshot below. 

`PsExec64.exe -i -u "nt authority\local service" cmd.exe`

We then pick a CLSID from [here](%20https://ohpe.github.io/juicy-potato/CLSID/). Interesting note: Numerous CLSIDs belong to `LOGGED-IN-USER`, so if you select this use this and a domain admin is logged in you can basically escalate directly to DA. However, it will only get the user of the first session \(1\). Finding a way to predict which user that is will require further testing. Either way, SYSTEM level privileges will get you where you want.

Now we run the exploit by specifiying a COM port of 1337, and executing the process cmd.exe trying both techniques `CreateProcessWithTokenW`,  `CreateProcessAsUser` A shell pops as `nt authority\system`

`juicypotato.exe -l 1337 -p c:\windows\system32\cmd.exe -t * -c {F87B28F1-DA9A-4F35-8EC0-800EFCF26B83}`

![](../.gitbook/assets/image%20%2824%29.png)

### Mitigation

This can't simply be patched. It's due to how service accounts needing to impersonate users when [kerberos delegation](https://technet.microsoft.com/en-us/library/cc995228.aspx) is enabled.

According to the creators, the actual solution is to protect sensitive accounts and applications which run under the `* SERVICE` accounts.

