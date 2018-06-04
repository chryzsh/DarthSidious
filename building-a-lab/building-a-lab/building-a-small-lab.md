# Building a small lab

## VMware or Virtualbox

* Set up a host-only network with DHCP on the host machine.
* Start whatever DHCP service you are running on your host.
* If you're using ESXI or other types of hypervisors, you probably don't need me telling you what to do.

## Install VMs

For this lab, installing a few Windows hosts and a server is necessary. Windows ISO's can be acquired [here](http://windowsiso.net/) and be run in trial mode. Architechture, service packs and patch levels are not important for this lab, unless you want to create specifically vulnerable machines. This will be added in the future. Kali Linux can be downloaded from [their website](https://www.kali.org/downloads/).

Install the following as VMs:

* **Windows 7**
* **Windows 8 or 8.1**
* **Windows 10**
* **Windows Server 2012 R2**
* **Kali Linux**

Give each machine a minimum of 2 vCPU, 2 GB RAM and enough storage space. Remember to set the network adapter for the VMs to the host-only network you created. Remember to also add an adapter in the Kali VM settings and make sure it gets an IP from the DHCP. You should be able to ping host from Kali and vice versa.

Set the same password on all accounts, as they will be the local administrator on the hosts. Set a stronger password on the Server which will be set up as Domain Admin \(DA\).

## Create a domain on the Domain Controller and add some users

[A good tutorial on setting up a Domain Controller](https://social.technet.microsoft.com/wiki/contents/articles/22622.building-your-first-domain-controller-on-2012-r2.aspx)

* Create a Domain on the Server 2012 R2 machine
* Set the time correctly \(this is actually important\)
* Set it up as a Domain Controller \(DC\)
* [Create three user accounts in the domain and give them simple names and passwords, like User1:Password1](https://msdn.microsoft.com/en-us/library/aa545262.aspx)
* Set a static IP with DNS pointed to 127.0.0.1

## Add hosts and users to the domain

These steps must be performed for all hosts: W7, W8 and W10.

* Log in as the administrator account on all hosts
* Set the time correctly \(this is actually important\)
* Disable Windows Defender and Windows Firewall
* Enable network sharing
* Set DNS settings to point to the IP of the DC
* Change the "computer name" to something easy \(like W7 for the Windows 7 machine\)
* [Add the hosts to the domain](https://technet.microsoft.com/en-us/library/bb456990.aspx)
* Add the "Domain Users" group to the administrators group using `net localgroup administrators /add "DOMAIN\Domain Users"`
* To confirm the group was added, run `net localgroup Administrators` and you should see "Domain Users" added to that group
* Log in to each of the domain users you created earlier on the workstations, except one box of your choosing, where you log in as DA
* Ping all the machines back and forth to make sure everything is working. Ping Kali and the host as well.