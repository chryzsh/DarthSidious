---
description: >-
  A truly awesome tool to graphically map out an entire AD forest and all its
  objects
---

# BloodHound

## Mapping AD with BloodHound

**Update march 2018:** Bloodhound has been released in version 1.5 which now includes GPO enumeration. More to come regarding this.

One of the glorious design features of AD is that everyone in the domain needs to know where everything is. So when you get user credentials and/or a shell, you can basically map the entire domain without breaking any rules. Any user can query Active Directory for computers, domain controllers, groups and sessions.

Now we can use this brilliant feature to collect a ton of information and create a cool GUI map of the entire AD which can be queried using BloodHound. There are two software requirements, you need Bloodhound and a database to store. The recommended choice is neo4j, see below.

## Installing neo4j

* [Install neo4j](https://neo4j.com/developer/kb/how-do-i-enable-remote-https-access-with-neo4j-30x/) [Community Edition](https://neo4j.com/download/community-edition/) manually from their [website](https://neo4j.com/download/?ref=hro) , not through apt.
* [http://neo4j.com/download/other-releases/\#releases](http://neo4j.com/download/other-releases/#releases)
* [A guide on installing can be found here](https://stealingthe.network/quick-guide-to-installing-bloodhound-in-kali-rolling/)
* Run neo4j from `/opt/neo4j-community-3.3.0/bin/neo4j start`
* Navigate to localhost:7474 in your browser.
* Set a new password for the neo4j account.
* Edit neo4j.conf and set the following parameters to make any host be able to access the database.

  ```text
  dbms.connector.http.enabled=true
  dbms.connector.http.listen_address=0.0.0.0:7474
  ```

* Restart neo4j with  `/opt/neo4j-community-3.1.1/bin/neo4j restart`
* Access neo4j in the browser at `http://0.0.0.0:7474/browser/`

#### Neo4j on Windows

Neo4j can be started with powershell on windows.

* Spawn an administrative powershell with -bypass exec
* Navigate to the neo4j/bin directory
* `Import-Module .\Neo4j-Management.psd1`
* `Invoke-Neo4j Console`
* Likewise to Linux, log in to localhost:7474 and change the password.

#### Neo4j on Mac

Similar procedure as linux. Neo4j does not support Java 9, so Java SDK must be version 8 and not 9 \(neo4j not suppported\). So install java 8 with Homebrew

* `brew update`
* `brew cask install java8`

### Installing Bloodhound

* Install BloodHound according to instructions on the [Github page](https://github.com/BloodHoundAD/BloodHound/wiki/Getting-started)
* Launch BloodHound and log in to the neo4j database with credentials from before

### Ingestion

To collect data in a format Bloodhound can read is called ingestion. There are several ways of doing this and different types of collection methods. The most useable is the Powershell ingestor called SharpHound, it's bundled with the latest release.

From Bloodhound [version 1.5](https://github.com/BloodHoundAD/BloodHound/releases/tag/1.5): the container update, you can use the new "All" collection open. See the blogpost from [Specter Ops](https://posts.specterops.io/bloodhound-1-5-the-container-update-fdf1ed2ad9da) for details.

### Powershell ingestion

What I recommend doing if you have internal network access is to run Bloodhound using runas /netonly from your own local box. This way you're not cluttering a domain joined machine with files and you don't have to extract them either, so its generally more covert. `runas /netonly /FQDN\user\<username> powershell` Type in password when prompted. This should spawn a new window. This window will use the local DNS settings to find the nearest domain controller and perform the various LDAP queries Bloodhound performs. First, import the powershell module `Import-module SharpHound.ps1` `Invoke-BloodHound -CollectionMethod All -CompressData -RemoveCSV`

You should now see data being populated into the database and you can play with BloodHound to create really some really cool maps. You can also perform queries to show the shortest path to DA, etc.

### Python ingestion from Kali

If you have a Kali box on the local network you can use the [ Bloodhound.py ingestor](https://github.com/fox-it/BloodHound.py).

## More Bloodhound stuff

Explanations of things found under Node info [https://github.com/BloodHoundAD/BloodHound/wiki/Users](https://github.com/BloodHoundAD/BloodHound/wiki/Users) [https://posts.specterops.io/sharphound-target-selection-and-api-usage-bba517b9e69b](https://posts.specterops.io/sharphound-target-selection-and-api-usage-bba517b9e69b) [https://github.com/porterhau5/BloodHound](https://github.com/porterhau5/BloodHound) [https://porterhau5.com/blog/representing-password-reuse-in-bloodhound/](https://porterhau5.com/blog/representing-password-reuse-in-bloodhound/) [https://porterhau5.com/blog/extending-bloodhound-track-and-visualize-your-compromise/](https://porterhau5.com/blog/extending-bloodhound-track-and-visualize-your-compromise/) [http://threat.tevora.com/lay-of-the-land-with-bloodhound/](http://threat.tevora.com/lay-of-the-land-with-bloodhound/)

