# From network access to Domain Admin

This article attempts to provide a methodology for domain pentests where the scenario is usually the type where you have network access to a network with one or several domains and nothing more. The goal is to get access as Domain Admin, because DA can do anything. You could say Enterprise Admin is an even bigger target, but to gain access to an EA is trivial once you have DA.

In real engagements you are usually provided with some basics, but to make this is hard as possible I will try to find a manual way to find every piece of information necessary to achieve full compromise.

## Step by step approach
The general approach to going from network access to domain admin consists of a few steps. Usually things don't work out perfectly linear like this on a real engagement. There will be forks, twists and turns on the way.

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

## Get a domain user
**Goal: get domain user access**
A domain user is any user in the domain. We of course have to figure out the domain name first.

## Enumerate the domain
Goal: discover computers, users and groups
* Bloodhound

## Identify paths to Domain Admin


## Move towards Domain Admin


## Hijack Domain Admin access