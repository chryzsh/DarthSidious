# Password spraying

**Tools**
https://github.com/dafthack/DomainPasswordSpray

### External password spraying
A domain user is any user in the domain. We of course have to figure out the domain name first. Once you have acquired the domain name, you can try a technique called password spraying. Most enterprises provide numerous ways of authenticating. Common ways are through Outlook Web Access, SMB or other. That means if we can figure out the syntax we can attempt to authenticate with common passwords. However, when spraying passwords beware of lockout tresholds. Many enterprises have a lockout policy after five attempts. BE VERY CAREFUL NOT TO LOCKOUT ACCOUNTS!


### Domain password spraying
Password spraying is however not only something you can do externally to find domain users. It can be very useful if you have internal access but no domain user, or to get the credentials ofr higher privileged users when nothing else is working.

