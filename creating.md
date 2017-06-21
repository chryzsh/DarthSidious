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
