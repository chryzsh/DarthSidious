# Domain admin in 30 minutes

This is an experience from a real internal domain pentest. Customer name has obviously been removed.

### Null session on domain controller \(DC\)

Did a fast scan with Nessus, could have done other things to find a domain controller like nmap or other windows commands.

Tried to check it the DC supported null sessions using the following command: 

```
something something
```

Domain controller gives us everything we need.

A list of every machine, every user, including admins and domain admins.

Bonus: every group membership and the password policy.

PW policy reveals there is no lockout on regular users and no requirement for complexity.

**It’s a party!**

### Password spraying

Every regular user has three letter usernames. Let’s make a list of all those. We don’t want to block out admin users \(they usually have lockout\). About 500 usernames on this test.

No complexity requirements = shit passwords. This test was performed during christmas 2017 so we make a list of relevant passwords and check it twice:

* Winter2017
* winter2017
* christmas2017
* Christmas2017
* Now we need a place to try the passwords. Trying to authenticate with a lot of failures on SMB may be detected and this stuff is way easier to set up in Burp anyway. We load up a list of usernames and passwords with Burp Intruder and point it to the external OWA e-mail server that really wasn't that hard to guess.

 `www.mail.customer.com`







