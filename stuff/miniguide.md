## Mini guide to Windows and Active Directory

This should cover the basics necessary to follow the attacks performed in this guide.

https://medium.com/@petergombos/lm-ntlm-net-ntlmv2-oh-my-a9b235c58ed4

### Windows hashes

There are a few different types of hashes in Windows and they can be very confusing. Some explanations can be found [here](http://www.adshotgyan.com/2012/02/lm-hash-and-nt-hash.html), but read this first:

No Windows hashes are salted, so two identical hashes will yield the same plaintext. Windows hashes are broken down into two hashes. LMhash and NTLMhash. This is an example: `testuser:29418:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71`

Broken down:  
`username : unique_identifier : LMhash : NThash`

* **LM** - The LM hash is used for storing passwords. It is disabled in W7 and above. However, LM is enabled in memory if the password is less than 15 characters. That's why all recommendations for admin accounts are 15+ chars. LM is old, based on MD4 and easy to crack. The reason is that Windows domains require speed, but that also makes for shit security.
* **NT** -  The NT hash calculates the hash based on the entire password the user entered. The LM hash splits the password into two 7-character chunks, padding as necessary.
* **NTLM** - The NTLM hash is used for local authentication on hosts in the domain. It is a combination of the LM and NT hash as seen above.
* **NetNTLMv1/2** - Hash for authentication on the network \(SMB\). Sometimes called NTLMv2, but don't get confused; it is not the same as an NTLM hash.

In windows the hashes are stored in memory for single sign-on purposes. Everytime a user clicks on a  network share the creds are passed across the network. We can exploit this by grabbing those credentials while in transit or on the machine itself. The alternative is to always ask the user for credentials, which will rarely happen in a windows environment.

NTLM hash is just as good as plaintext creds when authenticating to windows machines so it's not that big of a deal if you can't grab plaintext credentials. By good I mean it is possible to just pass the hash to authenticated, you don't need the password itself.

Cracking hashes can be a lot of fun, and since most user passwords are shitty / not complex they can easily be cracked in the manner of seconds or minutes. Imagine if the domain has a password reset policy of 90 days. If you crack a user's credentials in two hours it's a big fail for them. If you can crack a domain admin's creds in two hours or even a few days it's game over for them. But instead of cracking hashes, we can reuse them by relaying.


### Authentication in Windows
There are numerous ways of proving identity in Windows systems.
* **Passwords** - Passwords are
* **Hashes** - Windows can use hashes for authentication. It is possible to leverage attacks like pass-the-hash to prove identity with a compromised user, completely without the account password.
* **Tokens** - the concept of token is identity.  When a user or service logs in to a system, the system validates their identity once, and mints a token, which is handed to that user/service and serves as their identity. The system then doesn't need to validate identity every time a program opens a file, for example. This basically ensures a clean separation between authentication (proving a user/service is who they say they are) and authorization (determining whether a user/service can access some resource).
* **Tickets **- usually refers to Kerberos tickets, see below.


### Kerberos

Kerberos is an authentication protocol in Windows based on \_tickets \_to allow machines communicating over non-secure networks to provide their identity to one another in a secure manner. Kerberos builds on symmetric key cryptography and requires a trusted third party, and optionally may use public-key cryptography during certain phases of authentication. Kerberos uses UDP port 88 by default.

There have been discovered multiple exploits for Kerberos over the years:

* [MS14-068](https://www.exploit-db.com/exploits/35474/)

### Windows name resolution

There are a few different name resolution protocols and names in Windows:

* **FQDN** - Fully Qualified Domain Name
* **WINS** - Windows Internet Name Service
* **NBT-NS** - \(NetBIOS Name Service\) - commonly referred to as **NetBIOS**
* **LLMNR** - Link-Local Multicast Name Resolution
* **WPAD** - Web Proxy Auto-Discovery Protocol

If the name is a **FQDN**, which means a full name including domain name like `test.lab.local` it queries the hosts file, and then a DNS-server for name resolution.

If the name is an unqualified name like `\\fileshare`, the following name resolutions are attempted to find the that fileshare:  
1. **LLMNR** - uses multicast to perform name resolution for the names of neighboring computers without requiring a DNS server.  
2. **NetBIOS** - queries a WINS-server for resolution if present. If not, it uses broadcast to resolve the name from neighboring computers.

Because FQDN lookup is not common for fileshares and isn't enabled by default it checks LLMNR and then NetBIOS. In the corporate world a DNS server is available to look up resources, in a home environment it's less likely so if you want to share content between two hosts, LLMNR and NetBIOS is how it's done. However, users don't usually type in `share.hacklab.net` in the address field in  explorer, so the name resolution resorts to LLMNR and NetBIOS.

