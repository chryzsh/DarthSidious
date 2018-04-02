# Stealth

This chapter is about staying stealthy and opsec safe. That means not getting caught by the blue team on engagements.

## General
These are some key things we must avoid

* Putting files on disk
* RDP in to boxes
* Trigger pop-ups on desktops
* Changing account passwords
* Locking out users
* Changing group membership of accounts
* Changing existing settings and files
* Changing GPOs permanently
* Messing up Kerberos tickets
* Triggering alerts from security products like AV
* Killing processes you don't own
* Any sort of DOS
* Leaving files and tools
* Not cleaning up

## Using DLLs
https://pentestlab.blog/tag/rundll32/

## Obfuscating mimikatz

Any sysadmin with half a brain can now write and something to stop most common ways of executing mimikatz. Since we don't want to get caught we could obfuscate Mimikatz numerous ways.

* Running to memory either through Powershell or through meterpreter \(will probably get you caught\)
* Changing some basic things that will be triggered by signature, see: [https://gist.github.com/imaibou/92feba3455bf173f123fbe50bbe80781](https://gist.github.com/imaibou/92feba3455bf173f123fbe50bbe80781)

## Veil Pillage

Veil Pillage is a post exploitation tool and a part of the Veil framework intended for staying undetected through obfuscation.

[https://github.com/Veil-Framework/Veil-Pillage](https://github.com/Veil-Framework/Veil-Pillage)

