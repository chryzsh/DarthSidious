---
description: A technique to gain access to a domain user's credentials
---

# Password spraying

## General approach

The technique called password spraying is basically to attempt logins using a valid username and a list of passwords. This technnique is of course not the most reliable and is prone to lockout, but success scales with the amount of usernames obtained. Logins should therebefore be attempted in a very controlled fashion, and across multiple users to avoid lockouts. Password spraying is a technique that falls under [Mitre ATT&CK technique T1110 - Brute force](https://attack.mitre.org/wiki/Technique/T1110).

Password spraying has certain requirements for work. It doesn't matter if its external or internal, the principles stay the same. Most of these elements can be found without too much work.

**Requirements:**

* An available service to spray to
* Domain name
* Username / e-mail
* Password

**Tools:**

* [DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray)
* [MailSniper](https://github.com/dafthack/MailSniper)
* [auxiliary/scanner/http/owa\_login](https://www.rapid7.com/db/modules/auxiliary/scanner/http/owa_login)
* [auxiliary/scanner/http/owa\_ews\_login](https://www.rapid7.com/db/modules/auxiliary/scanner/http/owa_ews_login)

## External password spraying

A domain user is any user in the domain. We of course have to figure out the domain name first. Once you have acquired the domain name, you can try a technique called password spraying. Most enterprises provide numerous ways of authenticating. Common ways are through Outlook Web Access, SMB or other. That means if we can figure out the syntax we can attempt to authenticate with common passwords. However, when spraying passwords beware of lockout tresholds. Many enterprises have a lockout policy after five attempts. BE VERY CAREFUL NOT TO LOCKOUT ACCOUNTS!

**Step 1 Finding a service to spray to**  
An OWA is very typically located at mail.example.com or similar. Any ADFS portal should in theory work since its all the same AD authentication in the backend. However, certain tools like MailSniper are built around there actually being a mail portal, so not all functions may work.

**Step 2 Obtain the domain name  
**This feature of Metasploit owa\_login module can not only brute force, but it also has the ability to leak domain names. Note that this will be one attempt logged to the username if you have a valid username. Metasploit uses NTLM responses to identify the domain name. If you are really lucky, the helpful sysadmins have written the domain name to the OWA page already.

 `auxiliary/scanner/http/owa_login`  




**Step 3: Obtain a valid username or e-mail address  
**

Step 4: Perform password spraying

## Domain password spraying

Password spraying is however not only something you can do externally to find domain users. It can be very useful if you have internal access but no domain user, or to get the credentials ofr higher privileged users when nothing else is working.

