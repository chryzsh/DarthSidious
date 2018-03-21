# From network access to Domain Admin

This article attempts to provide a methodology for domain pentests where the scenario is usually the type where you have network access to a network with one or several domains and nothing more. The goal is to get access as Domain Admin, because DA can do anything. You could say Enterprise Admin is an even bigger target, but to gain access to an EA is trivial once you have DA.

In real engagements you are usually provided with some basics, but to make this is hard as possible I will try to find a manual way to find every piece of information necessary to achieve full compromise.

byt3bl33d3r has made a tool to automate most of the process after gaining domain user credentials. See the DeathStar article in this Gitbook and his own article here: [https://byt3bl33d3r.github.io/automating-the-empire-with-the-death-star-getting-domain-dmin-with-a-push-of-a-button.html](https://byt3bl33d3r.github.io/automating-the-empire-with-the-death-star-getting-domain-dmin-with-a-push-of-a-button.html)

![](/assets/deathstar.png)

## Step by step approach

The general approach to going from network access to domain admin consists of a few steps. Usually things don't work out perfectly linearly like this on a real engagement. There will almost always be forks, twists and turns on the way.

##### Enumerating the network

Since network access is the only thing provided, the first step naturally becomes to enumerate the network to discover computers.

##### Get a domain user

Only when a domain user has been acquired, enumerating the domain itself can begin.

##### Enumerate the domain

Get an overview of the domain. Bloodhound is a key tool for this step.

##### Identify paths to Domain Admin

Bloodhound is also great here.

##### Take steps towards Domain Admin

This depends on the domain size and configuration. If every user has local administrator access on their workstations and there are hundreds of DAs, this step is usually trivial.

##### Hijack Domain Admin access

This step is when you are finally close to a DA user. There are numerous ways to hijack a Domain Admin accounts.

## Enumerate the network

**Goal: Find a domain controller**  
We want to discover as much as possible. The best alternative here is to use a commercial vulnerability scanner. If such a tool is not available, use free tools like Nmap, but remember to stay organized when working in large networks. The output can be enormous.

We're looking for the good stuff here, the domain controllers. In big domains there are usually several.

Other things to look for are port 443 as SSL certificates often contain domain names. In relation to that you should also look for Outlook Web Access portals.

## Get a domain user

**Goal: Get domain user access**

**Method**  
A domain user is any user in the domain. We of course have to figure out the domain name first.

Once you have acquired the domain name, you can try a technique called password spraying. Most enterprises provide numerous ways of authenticating. Common ways are through Outlook Web Access, SMB or other. That means if we can figure out the syntax we can attempt to authenticate with common passwords.

However, when spraying passwords beware of lockout tresholds. Many enterprises have a lockout policy after five attempts.

## Enumerate the domain

**Goal: Discover computers, users and groups**

**Method**

We want to learn as much as possible about the domain. That means identifying what computers are in the domain, which users and who these belong too. These things are all called

**Tools**

* Bloodhound
* PowerView
* Empire

## Identify paths to Domain Admin

**Goal: Make a plan for reaching DA**  
Bloodhound should already have given you this. A default query in BH is "Shortest path do DA" which should give you a few options, granted the size and/or configuration of the domain. If not, however, manual enumeration is needed.

**Tools**

* Bloodhound

## Move towards Domain Admin

**Goal: Reaching a box with Domain Admin credentials on**

Once a path has been laid out, you can start moving laterally in the domain. However, since AD is built the way it is

## Hijack Domain Admin access

**Goal: Getting shell or credentials as DA      
**By this point you have ended up on a box

**Tools**

* Powershell
* PowerView
* 


