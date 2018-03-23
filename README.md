## Becoming Darth Sidious: Creating an Active Directory domain and hacking it

![](http://assets1.ignimgs.com/2015/05/27/lightsabersjpg-b61171_1280w.jpg)

This is a guide to setting up a windows domain in a virtual lab to practice penetration testing. It is a collaborative effort between three persistent pentesters: chryzsh, bufferov3rride and lkys37en. Wanna get in contact with any of us, write to either one of us on http://netsecfocus.slack.com/

**This guide/tutorial will teach you the following:**

* Creating a virtual Windows domain lab environment with a few hosts and a domain controller
* Credential Replay Attacks
* Domain Privilege Escalation
* Dumping System and Domain Secrets
* Tools like Empire, Bloodhound and ranger
* Cool attacks on Microsoft infrastructure
* Actual pentest experiences


-If you have no idew what you are doing, we recommend reading the mini guide to Windows and then begin setting up a lab. There are currently two lab building guides in this book:

##### Mini lab for children
The first lab guide helps you set up a very small and simple lab consisting of a domain controller and a few workstations. The goal is to practice Responder and other fun tools for credential relaying.

##### Big boy labs for big boys
We have added an article for building a much bigger lab using ESXi and automating a lot of it with Vagrant. If you feel confident, brave and have a lot of time, go for this. It requires fundamental knowledge of setting up Active Directory and configuring Windows machines. It could be used as a companion lab for MCSA and MCSE certifications.


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
* Abusing Cross Domain Trusts
* Delegation Issues
* Persistence Techniques
* Abusing SQL Server Trusts in an AD Environment
* Backdoors and Command and Control
* Forest and domain trusts in AD
* Detecting attack techniques
* Defending an Active Directory Environment
* [Attacking domain trusts](http://www.harmj0y.net/blog/redteaming/a-guide-to-attacking-domain-trusts/)
* LDAP integration with non-Microsoft products