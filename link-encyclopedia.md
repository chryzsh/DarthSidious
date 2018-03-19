# Link encyclopedia
Going to try to keep this updated.
## Microsoft

### Powershell
* [Powershell 101](https://hkh4cks.com/blog/2018/01/01/powershell-101/)
* [Learn Windows PowerShell in a Month of Lunches (Youtube)](https://www.youtube.com/playlist?list=PL6D474E721138865A) - Companion videos to the famous book
* [p3nt4/PowerShdll](https://github.com/p3nt4/PowerShdll) - Run PowerShell with dlls only. Does not require access to powershell.exe as it uses powershell automation dlls.

### Empire
* [Empire 101](http://www.powershellempire.com/?page_id=110) - Empire Introduction from official documentation

### Powerview
* [Powerview repository](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon) - contains documentation and how to use Powerview
* [PowerView-3.0-tricks.ps1](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993) - Powerview tips and tricks from HarmJ0y

### Bloodhound
* [Bloodhound node info](https://github.com/BloodHoundAD/BloodHound/wiki/Users) - Bloodhound Node info explanations
* [Lay of the land with bloodhound](http://threat.tevora.com/lay-of-the-land-with-bloodhound/) - General Bloodhound usage guide article

### Mimikatz
* [Lazykats](https://github.com/bhdresh/lazykatz) -  Mass Mimikatz with AV bypass (questionable)
* [Direct link to Invoke-Mimikatz.ps1](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1)
* [Auto dumping domain credentials](https://blog.netspi.com/auto-dumping-domain-credentials-using-spns-powershell-remoting-and-mimikatz/)
* [eladshamir/Internal-Monologue](https://github.com/eladshamir/Internal-Monologue) - Internal Monologue Attack: Retrieving NTLM Hashes without Touching LSASS

### Enumeration
* [Invoke-Portscan.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/262a260865d408808ab332f972d410d3b861eff1/Recon/Invoke-Portscan.ps1) - Invoke-Portscan is a module from Powersploit that can perform port scans similar to Nmap straight from Powershell.

### Tunneling
* [SShuttle](http://sshuttle.readthedocs.io/en/stable/) - SShuttle creates an SSH tunnel that works almost just like a VPN

### Command and control (C2)
* [SANS Pentest Blog](https://pen-testing.sans.org/blog/2017/12/10/putting-my-zero-cents-in-using-the-free-tier-on-amazon-web-services-ec2) - Using Amazon AWS EC2 for C2
* [lukebaggett/dnscat2-powershell](https://github.com/lukebaggett/dnscat2-powershell) - Powershell implementation of dnscat2 client
* [C2 with dnscat2 and powershell](https://www.blackhillsinfosec.com/powershell-dns-command-control-with-dnscat2-powershell/) - dnscat2 can be used with powershell for working over DNS to hide C2 activity
* [DNS tunneling](https://pentest.blog/data-exfiltration-tunneling-attacks-against-corporate-network/) - How DNS tunneling works

### Exploit
* [SharpShooter](https://github.com/mdsecactivebreach/SharpShooter) - SharpShooter can create payloads for many formats like HTA, JS and VBS
* [DCShadow](https://blog.alsid.eu/dcshadow-explained-4510f52fc19d) - DCShadow, attack technique to create a rogue domain controller

#### Mail
* [Ruler](https://github.com/sensepost/ruler) - Ruler can interact with Exchange servers remotely

### Breaking out of locked down environments
* [Breaking Out of Citrix and other Restricted Desktop Environments](https://www.pentestpartners.com/security-blog/breaking-out-of-citrix-and-other-restricted-desktop-environments/)
* [Applocker Case study](https://oddvar.moe/2017/12/21/applocker-case-study-how-insecure-is-it-really-part-2/) - Breaking out of Applocker using advanced techniques
* [Bypass Applocker](https://github.com/api0cradle/UltimateAppLockerByPassList) - List of most known Applocker bypass techniques
* [Babushka Dolls or How To Bypass Application Whitelisting and Constrained Powershell](https://improsec.com/blog/babushka-dolls-or-how-to-bypass-application-whitelisting-and-constrained-powershell)


## Defense
* [MS - Securing privileged access](https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material) - Reference material for securing admin access in AD
* [MS - What is AD Red Forest](https://social.technet.microsoft.com/wiki/contents/articles/37509.what-is-active-directory-red-forest-design.aspx) - Red forest design is building an administrative AD environement built with security in mind
* [Managing Applocker with Powershell](https://4sysops.com/archives/managing-applocker-with-powershell/)

## Lab building
* [Automatedlab/Automatedlab](https://github.com/AutomatedLab/AutomatedLab) - Automatedlab is a project for building a lab environment automatically using Powershell.
* [Building a lab with ESXI and Vagrant](building-a-lab-with-esxi-and-vagrant.md) - Big article from this book about building a lab using ESXi
* [Mini lab](creating.md) - Small article from this book about creating a small lab for practicing things like Responder

## Other
* [OSCP Survival Guide archived](http://web.archive.org/web/20171014213457/https://github.com/frizb/OSCP-Survival-Guide) - contains a ton of useful commands for enumeration and exploitation