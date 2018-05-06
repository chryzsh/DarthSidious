# External network access to Domain Admin

#### Intro

This article attempts to provide a methodology for domain pentests. The scenario is that you have been provided internal network access to a network with one or several domains and nothing more. The goal is to get access as Domain Admin.

In certain pentesting engagements some level of access is usually provided. Very often this is internal network access, but that does not mean you should necessarily skip the parts before. There will potentially be things on the internet that will help on the way to domain admin. I try with this article to provide a manual way to find every piece of information necessary to achieve DA access.

#### Step by step approach

The general approach to going from external network access to domain admin consists of numerous steps. On a real domain pentest, this usually isn't possible in such a linear fashion as outlined here. There will always be twists and turns requiring you to take steps in multiple directions. However, these steps should help you structure your approach to domain pentesting.

## Enumerating the external network

**Goal: Acquire a target machine and service** You are on the external network, often this will be the Internet. You should of course have a target in mind already. If you're targeting a lab environment, a set of IP addressed should already be available. The first step will be enumerating a set of IPs or subnets. Once one or more target machines and services have been discovered, they should be investigated thoroughly.

**Techniques**

* Port scanning
* Investigating exposed services

**Things to look out for**

* IIS \(80, 443\)
* RDP \(3389\)
* MSSQL \(1433\)
* RDS \(3389\)
* RDWEB \(3389, navigate to [http://ip/rdweb](http://ip/rdweb)\)
* OWA \(80, navigate to subdomain email.domain.com or http://ip/owa\)

**Tools**

* Nmap
* Firefox

## Acquiring domain user credentials

**Goal: Acquire domain user credentials** A domain user is any user in the domain. We of course have to figure out the domain name first. Once you have acquired the domain name, you can try a technique called password spraying. Most enterprises provide numerous ways of authenticating. Common ways are through Outlook Web Access, SMB or other. That means if we can figure out the syntax we can attempt to authenticate with common passwords. However, when spraying passwords beware of lockout tresholds. Many enterprises have a lockout policy after five attempts.

**Method**

* Figure out the domain name
* Figure out the domain account name syntax
* Acquire a domain user account name
* Acquire a domain account's password

**Technique**

* OSINT
* Investigate content of exposed services
* Password spraying on OWA
* Other means of verifying credentials \(RDP\)

**Tools**

* OSINT
* Burp Intruder
* Mailsniper

### Acquiring a shell on the internal network

**Goal: Getting a shell on a machine in the internal network** The goal here is getting a payload executed on the internal network that connects back to you. This, so you gain a shell on a machine in the internal network.

The most obvious method is like hacking in 2003, you exploit some vulnrability in an externally exposed service. That can be everything from an RCE in wordpress, Sharepoint or an SQL injection somewhere. If this is not possible, there are tools that can help you abuse existing services like Ruler abuses Exchange. The last alternative is phishing, which may gain you command execution, but at the same time is a somewhat loud way to go about.

**Technique**

* Abusing externally exposed services
* Exploiting vulnerabilities
* Phishing

**Tools**

* Ruler
* Phishing with HTA payload
* Metasploit

### Command and Control \(C2\)

**Goal: establishing a communication system to the internal network** A shell gained through for example an SQL injection might not be the most stable channel to work with. We need to set up a better channel for interacting with the system. This is where a command and control infrastructure comes in. It can easily be set up from an environment where you control port openings like at home, or through an AWS box.

**Technique**

* Establish a C2 server
* Open a port and listener on the server
* Connect back from domain
* Establish persistence

**Tools**

* Empire
* dnscat2
* Empire persistence modules

## Enumerate the internal network

**Goal: Find vulnerabilites on machines in the internal network**

**Method** Only when a domain user shell has been acquired and a stable means of communicating and interacting with the internal network has been established, enumerating the domain itself can begin.

We want to discover as much as possible. The best alternative here is to use a commercial vulnerability scanner. If such a tool is not available, use free tools like Nmap, but remember to stay organized when working in large networks. The output can be enormous.

We're looking for the good stuff here, the domain controllers. In big domains there are usually several.

Other things to look for are port 443 as SSL certificates often contain domain names. In relation to that you should also look for Outlook Web Access portals.

Depending on your goal, this step can potentially be skipped in favor of starting to enumerate the domain instead. You might discover domain admin is wonderfully close, and if that's your goal then spending a lot of time scanning for vulnerabilites is essentially a waste. However, you might discover machines in the domain vulnerable to MS17-010 and if they in fact are exploitable, you have very easy SYSTEM access on boxes. Note that port and vulnerability scanning inside a controlled network usually gets picked up by the blue team very fast. So it's not a good option if you're looking to be stealthy.

**Technique**

* Scan for vulnerabilites
* Review results
* Pick a target

**Tools**

* Nessus
* Nmap
* Metasploit

## Automatically enumerating the domain

**Goal: Discover computers, users, groups and GPOs** We want to learn as much as possible about the domain. That means identifying what computers are in the domain, which users and who these belong too. There will also be GPOs applied that can be enumerated. Certain domains may have several trusts too. Bloodhound is a key tool for this step. Bloodhound should already have given you a path now. A default query in BH is "Shortest path do DA" which should give you a few options, granted the size and/or configuration of the domain. If not, however, manual enumeration is needed.

**Technique**

* Run Bloodhound collection
* Look for DA sessions
* Look for local admin privileges for current user
* Identify paths to Domain Admin

**Tools**

* Bloodhound
* PowerView
* Empire

## Manually enumerating the domain

**Goal: Increasing privileges in the domain** Usually if you are a regular user of sorts you won't have privileges to much in the domain. That means you are probably not local administrator anywhere. This is pretty much a key step to be able to use tools like Mimikatz. A typical approach is to go from workstation user -&gt; workstation admin -&gt; server admin -&gt; domain admin.

In a decently patched AD environment, there won't be any point in trying too hard to escalate privileges locally on a box.

**Technique**

* Identify if the user you have is local admin on any machine
* Identify group memberships
* Identify GPOs applied to user, group membership or current machine
* Possibly escalate privileges locally

**Tools**

* Empire
* PowerView

## Move towards Domain Admin

**Goal: Command execution on a box with Domain Admin session on** Once a path has been laid out, you can start moving laterally in the domain. However, since AD is built the way it is you don't necessarily have to pop shells all over the place. Not only does this increase the possibility of detection, but if the environment have the capabilities of remoting in to boxes, then that is a better option. This is why i have written command execution and not shell here. While the difference is only so slight, it can be a very efficient way of getting things done. Also consider you will potentially be doing these kinds of activities over a tunnel, which might add significant delay to your operations. Normally, the deeper you get into the internal network, the tougher it gets.

This step depends a lot on the domain size, security maturity and configuration. If every user has local administrator access on their workstations or there are hundreds of DAs, this step is usually trivial

**Technique**

* Pick a target box where DA has session
* Enumerate privileges on that box
* Enumerate whether remote command execution is possible

**Tools**

* WinRM
* WMI
* PsExec
* Empire lateral movement modules

## Hijack Domain Admin access

**Goal: Command execution as DA or DA credentials**  
By this point you should have access to execute command on a box that hopefully has a domain admin logged in. The goal of this step is then to escalate privileges to DA. If you are local administrator on the box, this is easy. You can simply use mimikatz to dump credentials \(passwords or hashes\) for users. If that is not an option you can use token impersonation to steal the DA's token. However, if no admin privileges at all is possible to gain, you should start enumerating who is local admin on this box and see if you can acqurie said privilege.

**Technique**

* If local admin -&gt; Mimikatz
* Else -&gt; enumerate who is local admin
* Gain user to local admin for said box

**Tools**

* Powershell
* PowerView
* Invoke-TokenImpersonation
* Invoke-InternalMonologue

