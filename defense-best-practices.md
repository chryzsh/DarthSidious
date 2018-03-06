# Defense

[https://thedefensedude.wordpress.com](https://thedefensedude.wordpress.com)

## Defending against Responder

#### Step 1. NBNS/LLMNR/WPAD Broadcasts.

If the admin had disabled WPAD or NBNS broadcasts, hashes wouldn't have been caught. This can be done through group policy.

#### Step 2. Password policy

Had the users been forced into using complex passwords with technical controls, cracking would take extra time, and at least become a speed bump to a pentester.

##### Step 3. Local Admin

Administrators should disable local administrator access to machines within a network by users if possible. Create a separate login to be used when admin access is needed which is a member of the local admin group within Active Directory. \(i.e. Maleus21 is my user, Maleus21\_LA is my local admin user to be used when Windows requires administrative rights to run a program\). Local administrator access can easily be deployed and managed through LAPS, see below.

###### Implementing Local Administrator Password Solution \(LAPS[\) ](https://technet.microsoft.com/en-us/mt227395.aspx)

The "Local Administrator Password Solution" \(LAPS\) provides management of local account passwords of domain joined computers. Passwords are stored in Active Directory \(AD\) and protected by ACL, so only eligible users can read it or request its reset. https://www.microsoft.com/en-us/download/details.aspx?id=46899

[https://technet.microsoft.com/en-us/mt227395.aspx](https://technet.microsoft.com/en-us/mt227395.aspx)

[https://adsecurity.org/?p=3164](https://adsecurity.org/?p=3164)

[https://www.youtube.com/watch?v=WD2cBKRvERc&t=12s](https://www.youtube.com/watch?v=WD2cBKRvERc&t=12s)

#### Step 4. Get rid of Mimikatz

Mimikatz works by wdigest within Windows. A simple registry edit can get rid of wdigest. Additionally, User accounts should not be domain admins, see Step 3.

Had any one of those steps been in place, the attack would have been dead and he would need to look for another vector. Sadly, its rare that any of these are in place, and the above attack is the "go to" method on any internal engagement, because 99% of the time, it works.


###¤ SMB Signing

SMB signing can prevent relaying. It is designed to limit man-in-the-middle attacks. SMB Message signing signs SMB packets to ensure that they cannot be replayed against a different system. SMB Message signing is actually four different settings.

The "Local Administrator Password Solution" \(LAPS\) provides management of local account passwords of domain joined computers. Passwords are stored in Active Directory \(AD\) and protected by ACL, so only eligible users can read it or request its reset. https://www.microsoft.com/en-us/download/details.aspx?id=46899







