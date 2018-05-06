---
description: >-
  This is a guide to setting up a windows domain in a virtual lab to practice
  penetration testing and hacking it.
---

# Getting started

**This guide/tutorial will teach you the following:**

* Creating a virtual Active Directory domain lab environment
* Credential Replay Attacks
* Domain Privilege Escalation
* Dumping System and Domain Secrets
* Tools like Empire, Bloodhound and ranger
* Actual pentest experiences

If you have no idea what you are doing, we recommend reading the [Mini guide to Windows ](https://github.com/chryzsh/DarthSidious/tree/fdd707cf9dbbc2faf3cf3dbbcd712b06fceeee87/labs/stuff/miniguide.md) and then begin [Building a lab](https://github.com/chryzsh/DarthSidious/tree/fdd707cf9dbbc2faf3cf3dbbcd712b06fceeee87/labs/labs/building-a-lab.md).

If you have an idea what you're doing and/or already have a lab environment I recommend you check out the article [Network access to Domain Admin](https://github.com/chryzsh/DarthSidious/tree/fdd707cf9dbbc2faf3cf3dbbcd712b06fceeee87/labs/general/network-access-to-domain-admin.md) for a general approach to hacking Active Directory domains.

**Obvious disclaimer is obvious**

The tools demonstrated in this book should not be used in an environment without complete authorization from it's legal owner. I.e. don't be stupid and don't run commands you don't know what does.

**Todo list**

* Stealth - improve article
* Introduction to Active Directory
* Kerberos and authentication in AD
* Introduction to PowerShell
* Exploiting MSSQL Servers
* Client Side Attacks
* Domain Enumeration and Information Gathering
* Local Privilege Escalation
* Exchange enumeration and attacks
* Sharepoint enumeration and attack
* Mitigations against common attacks
* General recommendations for securing AD

**Future plans**

* Kerberos Attacks and Defense \(Golden, Silver tickets and more\)
* Delegation Issues
* Persistence Techniques
* Abusing SQL Server Trusts in an AD Environment
* Backdoors and Command and Control
* Forest and domain trusts in AD
* Detecting attack techniques
* Defending an Active Directory Environment
* [Attacking domain trusts](http://www.harmj0y.net/blog/redteaming/a-guide-to-attacking-domain-trusts/)
* LDAP integration with non-Microsoft products

