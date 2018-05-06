---
description: Enumeration tool for Active Directory
---

# PowerView

[PowerView ](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon)is a great tool for domain enumeration part of the PowerSploit collection. Almost every function is available in Empire too. Although each function is pretty self-explainable and you should explore it yourself I'll provide some hints here. I can also highly recommend reading the source code for these kinds of Powershell based hacking tools, because there are usually tons of tips and examples bundled with them.

Harmj0y has tips and tricks on his Github too [https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)

## User functions

`Invoke-EnumerateLocalAdmin` Enumerates members of the local Administrators groups across all machines in the domain.

## Group functions

`Get-NetLocalGroup -ComputerName MX01 -GroupName "Remote Management Users"` Gets members of users who can use WinRM on a specific machine.

`Get-NetLocalGroup -ComputerName MX01 -GroupName "Remote Desktop Users"` Gets members of users who can RDP to a specific machine.

## GPO functions

`Get-NetGPOGroup -ResolveMemberSIDs` Gets all GPOs in a domain that set "Restricted Groups" on on target machines, and resolve the SIDs of the member groups or users.

`Find-GPOLocation -UserName testuser -LocalGroup RDP` Takes a speicifc user or group and checks where the user has access to through GPO enumeration. Here we can see what boxes testuser can RDP into.

