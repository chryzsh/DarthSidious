# Password spraying

## General approach

Password spraying is a technique to attempt logins using valid usernames and a list of passwords with the purpose of obtaining a set of valid credentials.

This technnique is naturally somewhat unreliable and is also on the edge of opsec if not carefully executed. Password spraying should therebefore be attempted in a very controlled fashion, and across multiple users to avoid lockouts. Password spraying is a technique that falls under [MITRE ATT&CK technique T1110 - Brute force](https://attack.mitre.org/wiki/Technique/T1110).

The technique has certain requirements that must be fulfilled for it to work. It doesn't matter if its performed externally or internally, the principles stay the same. Most of the required elements can be found without too much effort.

### **Requirements**

* An available service to spray to
* Domain name
* Username or e-mail

### **Tools:**

* [DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray)
* [MailSniper](https://github.com/dafthack/MailSniper)
* [auxiliary/scanner/http/owa\_login](https://www.rapid7.com/db/modules/auxiliary/scanner/http/owa_login)
* [auxiliary/scanner/http/owa\_ews\_login](https://www.rapid7.com/db/modules/auxiliary/scanner/http/owa_ews_login)

## External password spraying

External password spraying is when we perform the attack from outside a domain. Most enterprises provide their employees a way of accessing e-mail from the internet. Therefore, most organizations have a portal for Outlook Web App \(OWA\). It usually looks something like this:

![](../.gitbook/assets/image%20%286%29.png)

That means if we can figure out the syntax we can attempt to authenticate with common passwords. However, when spraying passwords beware of lockout tresholds. Many enterprises have a lockout policy after five or ten attempts, but some have lockouts as low as three attemps. There is also usually a treshold for this lockout set around 30 minutes. But mind you, that these are all educated guesses and may vary. So always err on the side of caution when password spraying.

BE VERY CAREFUL NOT TO LOCKOUT ACCOUNTS!

### **Step 1 Finding a service to spray**

OWA portals are typically located at `mail.testlab.local` ,`testlab.local/owa` or similar. Any ADFS integrated portal should in theory work since its all the same AD authentication in the back-end. However, certain tools like MailSniper are built for attacking certain parts of the mail portal, so not all functions may work if its not an OWA. Subdomain enumeration can be very helpful for discovering such portals. Another alternative is Remote Web Access pages, typically located at `testlab.local/RDweb`

### **Step 2 Obtain the domain name**

The Metasploit owa\_login module can not only brute force, but it also has the ability to leak domain names. Note that this will be one attempt to the username if you have a valid one. Metasploit uses NTLM responses to identify the domain name. If you are really lucky, the helpful sysadmins have written the domain name to the OWA page already. Note that in this attempt, you don't necessarily need a valid user.

```text
use auxiliary/scanner/http/owa_login
set rhost <rhost>
set username <username>
set password <password>
exploit

[+] Found target domain: TESTLAB
```

Alternatively, MailSniper can be used to obtain the domain name using response times.

```text
Invoke-DomainHarvestOWA -ExchHostname testlab.local -DomainList c:\temp\domainlist.txt -OutFile potential-domains.txt

[*] Harvesting Domain Name from the OWA portal at https://testlab.local/owa/
...
[*] Potentially Valid Domain! Domain:TESTLAB
```

### **Step 3 Obtain a valid username or e-mail address**

We got the domain name, now let's try to enumerate ourselves to a username. If we do not have an e-mail or a username, we need to do more recon to get that. The alternative would be brute forcing ourselves to a username, but that's not always a viable option.

Let's assume for the sake of example we have gotten an e-mail address through LinkedIn, which is not far fetched at all. Side note: check out [theHarvester](https://github.com/laramies/theHarvester). Let's say we find an employee for this organization on LinkedIn. His name is `Bill Long Money` and his e-mail is`bill.money@testlab.biz`

Now there are a number of possible syntaxes for domain users, but most follow one of a few common schemes. Let's apply them to our friend:  
- First letter of first name + entire last name = `bmoney`  
- First letter of first and last name \(could also include middle names\) = `bm`  or  `blm`  
- Prefix \(a character usually\) + first letter of first name + entire last name =  `a-bmoney`  
- Full name = `billmoney` \(less common I believe\)

We can try the different schemes with a domain name. Usually the owa\_login module is able to verify the validity of accounts without a correct password.  Let's try a spray with a randomly guessed password with `bmoney` as username.

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

We were correct about the username syntax and so we got a valid username. Note that this module uses response time to assess the validity of usernames, so you might get false positives here. It can be worth trying a few usernames you know are invalid to verify whether .

As before we can also use Mailsniper for this task

```text
Invoke-UsernameHarvestOWA -ExchHostname testlab.corp.local -UserList c:\temp\userlist.txt -Domain TESTLAB -OutFile potential-usernames.txt

[*] Now spraying the OWA portal at https://testlab.local/owa/
...
[*] Potentially Valid! User:TESTLAB\bmoney
```

### Step 4 Perform password spraying

Now when both a domain name and a user name has been acquired we can perform the actual password spraying. As mentioned before, the it's very important to spray in a controlled manner to prevent lockout. That means to record which users were sprayed, exactly when, what passwords were tried and how many attempts for each user.

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

Alternatively the spraying can be performed with MailSniper, where the username is entered into `userlist.txt`

```text
Invoke-PasswordSprayOWA -ExchHostname testlab.local -userlist .\userlist.txt -password winter2018

[*] Now spraying the OWA portal at https://testlab.local/owa/

[*] SUCCESS! User:bmoney:winter2018
```

Awesome! We got a valid password!

Now we did this example with one user only, but don't forget that the approach should be to spray a lot of users using generic passwords like winter2018 and hope they stick. The larger the list of users, the greater the chance of success.

## Internal password spraying

Password spraying is however not only something you can do externally to find domain users. It can be very useful if you have internal access but no domain user, or to get the credentials ofr higher privileged users when nothing else is working.

More coming soon!

