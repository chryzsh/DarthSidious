## Mini guide to Windows

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
