## Becoming Darth Sidious: Creating a Windows domain and hacking it

This is a guide to setting up a windows domain in a virtual lab to practice penetration testing. This guide/tutorial will teach you the following:

- Creating a basic windows domain with a few hosts and a domain controller.
- Credential Replay Attacks
- Domain Privilege Escalation
- Dumping System and Domain Secrets
- Using a light saber (and a blaster) to take control of your Empire.

![](http://assets1.ignimgs.com/2015/05/27/lightsabersjpg-b61171_1280w.jpg)
<br><br>
**Obvious disclaimer is obvious**

The tools demonstrated in this guide should not be used in an environment without complete authorization from it's legal owner.

**Credits**

_Credz to **lkys37en** from the netsecfocus slack channel for getting me into this stuff, and eventually making this guide possible.

**Future plans**

- Introduction to Active Directory and Kerberos
- Introduction to PowerShell
- Exploiting MSSQL Servers
- Client Side Attacks
- Domain Enumeration and Information Gathering
- Local Privilege Escalation

**More advanced stuff for the future**
- Kerberos Attacks and Defense (Golden, Silver tickets and more)
- Abusing Cross Domain Trusts
- Delegation Issues
- Persistence Techniques
- Abusing SQL Server Trusts in an AD Environment
- Backdoors and Command and Control
- Other trusts in AD
- Detecting attack techniques
- Defending an Active Directory Environment