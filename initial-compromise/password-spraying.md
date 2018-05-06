---
description: A brute force technique to gain access to a domain user's credentials
---

# Password spraying

## General approach

The technique called password spraying is basically to attempt logins using a valid username and a list of passwords. This technnique is of course not the most reliable and is prone to lockout, but success scales with the amount of usernames obtained. Logins should therebefore be attempted in a very controlled fashion, and across multiple users to avoid lockouts. Password spraying is a technique that falls under [MITRE ATT&CK technique T1110 - Brute force](https://attack.mitre.org/wiki/Technique/T1110).

Password spraying has certain requirements for work. It doesn't matter if its external or internal, the principles stay the same. Most of these elements can be found without too much work.

### **Requirements:**

* An available service to spray to
* Domain name
* Username / e-mail
* Password

### **Tools:**

* [DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray)
* [MailSniper](https://github.com/dafthack/MailSniper)
* [auxiliary/scanner/http/owa\_login](https://www.rapid7.com/db/modules/auxiliary/scanner/http/owa_login)
* [auxiliary/scanner/http/owa\_ews\_login](https://www.rapid7.com/db/modules/auxiliary/scanner/http/owa_ews_login)

## External password spraying

A domain user is any user in the domain. We of course have to figure out the domain name first. Once you have acquired the domain name, you can try a technique called password spraying. Most enterprises provide numerous ways of authenticating. Common ways are through Outlook Web Access, SMB or other. That means if we can figure out the syntax we can attempt to authenticate with common passwords. However, when spraying passwords beware of lockout tresholds. Many enterprises have a lockout policy after five attempts. BE VERY CAREFUL NOT TO LOCKOUT ACCOUNTS!

### **Step 1 Finding a service to spray**

An OWA portal is very typically located at `mail.example.com` or similar. Any ADFS integrated portal should in theory work since its all the same AD authentication in the back-end. However, certain tools like MailSniper are built for attacking certain parts of the mail portal, so not all functions may work if its not an OWA.

### **Step 2 Obtain the domain name**

This feature of Metasploit owa\_login module can not only brute force, but it also has the ability to leak domain names. Note that this will be one attempt logged to the username if you have a valid username. Metasploit uses NTLM responses to identify the domain name. If you are really lucky, the helpful sysadmins have written the domain name to the OWA page already. Note that in this attempt, you don't necessarily need a valid user.

```text
use auxiliary/scanner/http/owa_login
set rhost <rhost>
set username <username>
set password <password>
exploit

[+] Found target domain: TESTLAB
```

### **Step 3 Obtain a valid username or e-mail address**

We got the domain name, now let's try to enumerate ourselves to a username. If we do not have an e-mail or a username, we need to do OSINT or similar to get a username. The alternative would be brute forcing ourselves to  username, which is not really viable. Let's assume for the sake of example we have gotten an e-mail address through LinkedIn, which is not a far fetched scenario at all. Now there are a number of possible syntaxes in a domain, but most follow a few common schemes.  
Let's say his name is Bill Long Money and his e-mail we found on LinkedIn is`bill.money@testlab.biz`  
- First letter of first name + entire last name = `bmoney`  
- First letter of first and last name \(could also involve middle names\) = `bm`  or  `blm`  
- Prefix \(some character usually\) + First letter of first name + entire last name =  `a-bmoney`  
- Full name = `billmoney` \(less common I believe\)

Anyway, we can try the different schemes with a domain name. Usually the owa\_login module is able to verify the validity of accounts without a correct password.  Let's try a spray with a randomly guessed password

```text
use auxiliary/scanner/http/owa_login
set rhost 10.10.10.10
set domain TESTLAB
set username bmoney
set password ilovemoney
exploit

[*] 10.10.10.10:443 OWA - Trying bmoney : ilovemoney
[+] server type: MX01
[*] 10.10.10.10:443 OWA - FAILED LOGIN, BUT USERNAME IS VALID. 0.224583728 'TESTLAB\bmoney' : 'ilovemoney': SAVING TO CREDS
[*] Auxiliary module execution completed
```

We we're correct about the username syntax and so we got a valid username. Note that this module uses response time to assess the validity of usernames, so you might get false positives here. It can be worth trying a few usernames you know are invalid to verify.

### Step 4: Perform password spraying

Now when both a domain name and a user name has been acquired we can perform the actual password spraying. The key here is to spray in a controlled manner to prevent lockout. That means to record which users were sprayed, exactly when, what password were tried and how many attempts for each user.

```text

```

Alternatively the spraying can be performed with MailSniper, where the username is entered into `userlist.txt`

```text
Invoke-PasswordSprayOWA -ExchHostname mx01.testlab.l
ocal -userlist .\userlist.txt -password winter2018
```

## Domain password spraying

Password spraying is however not only something you can do externally to find domain users. It can be very useful if you have internal access but no domain user, or to get the credentials ofr higher privileged users when nothing else is working.

