---
description: Enumerating Azure services
---

# Azure enumeration

## Common misconfigurations

_Or, what Microsoft refers to as default._

Azure is by default open to every user in the organization. That means clients who for instance have Office 365 most likely haven't set up a conditional access policy to prevent users from logging in to [portal.azure.com](http://portal.azure.com/) and retrieving every user, role and group. Odds are that if they haven't done that, they don't monitor what the users do there to closely either. So why ask the on-premise domain controller and get detected if you can get what you need right from Azure? Procedures for doing the magic with Powershell below.

## Procedures and tools

### **Useful articles**

* [https://www.blackhillsinfosec.com/red-teaming-microsoft-part-1-active-directory-leaks-via-azure/](https://www.blackhillsinfosec.com/red-teaming-microsoft-part-1-active-directory-leaks-via-azure/)
* [https://blog.netspi.com/enumerating-azure-services/](https://blog.netspi.com/enumerating-azure-services/)
* [https://blog.netspi.com/get-azurepasswords](https://blog.netspi.com/get-azurepasswords/)

### **Tools**

* [https://github.com/mwrlabs/Azurite](https://github.com/mwrlabs/Azurite) - Azurite was developed to assist penetration testers and auditors during the enumeration and reconnaisance activities within the Microsof Azure public Cloud environment.
* [https://github.com/nccgroup/azucar](https://github.com/nccgroup/azucar) - Azucar automatically gathers a variety of configuration data and analyses all data relating to a particular subscription in order to determine security risks.
* [https://github.com/NetSPI/MicroBurst](https://github.com/NetSPI/MicroBurst) - MicroBurst includes functions and scripts that support Azure Services discovery, weak configuration auditing, and post exploitation actions such as credential dumping. It is intended to be used during penetration tests where Azure is in use.
* [https://github.com/chrismaddalena/SharpCloud](https://github.com/chrismaddalena/SharpCloud) - SharpCloud is a simple C\# utility for checking for the existence of credential files related to Amazon Web Services, Microsoft Azure, and Google Compute.   

### **Procedures**

A little code block with some common procedures for enumerating Azure AD.



```text
### Azure AD enumeration

#Connect to Azure AD using Powershell
install-module azuread
import-module azuread
get-module azuread
connect-azuread

# Get list of users with role global admins# Note that role =! group
$role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Company Administrator'}
Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId

# Get all groups and an example using filter
Get-AzureADGroup
Get-AzureADGroup -Filter "DisplayName eq 'Intune Administrators'"

# Get Azure AD policy
Get-AzureADPolicy

# Get Azure AD roles with some examples
Get-AzureADDirectoryRole
Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Security Reader'}
Get-AzureADDirectoryRoleTemplate

#Get Azure AD SPNs
Get-AzureADServicePrincipal

# Log in using Azure CLI (this is not powershell)
az login --allow-no-subscriptions

#Get member list using Azure CLI
az ad group member list --output=json --query='[].{Created:createdDateTime,UPN:userPrincipalName,Name:displayName,Title:jobTitle,Department:department,Email:mail,UserId:mailNickname,Phone:telephoneNumber,Mobile:mobile,Enabled:accountEnabled}' --group='Company Administrators'

#Get user list
az ad user list --output=json --query='[].{Created:createdDateTime,UPN:userPrincipalName,Name:displayName,Title:jobTitle,Department:department,Email:mail,UserId:mailNickname,Phone:telephoneNumber,Mobile:mobile,Enabled:accountEnabled}' --upn='username@domain.com'

#PS script to get array of users / roles
$roleUsers = @() 
$roles=Get-AzureADDirectoryRole
 
ForEach($role in $roles) {
  $users=Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId
  ForEach($user in $users) {
    write-host $role.DisplayName,$user.DisplayName
    $obj = New-Object PSCustomObject
    $obj | Add-Member -type NoteProperty -name RoleName -value ""
    $obj | Add-Member -type NoteProperty -name UserDisplayName -value ""
    $obj | Add-Member -type NoteProperty -name IsAdSynced -value false
    $obj.RoleName=$role.DisplayName
    $obj.UserDisplayName=$user.DisplayName
    $obj.IsAdSynced=$user.DirSyncEnabled -eq $true
    $roleUsers+=$obj
  }
}
$roleUsers

### Enumeration using Microburst
https://github.com/NetSPI/MicroBurst

Import-Module .\MicroBurst.psm1

#Anonymous enumeration
Invoke-EnumerateAzureBlobs -Base company
Invoke-EnumerateAzureSubDomains -base company -verbose

#Authencticated enumeration
Get-AzureDomainInfo -folder MicroBurst -VerboseGet-MSOLDomainInfo
Get-MSOLDomainInfo
```

```text

```

