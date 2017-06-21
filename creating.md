## Creating a lab

**I strongly you recommend reading every step closely, as missing out on certain stuff can make things not work as it should.**

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
