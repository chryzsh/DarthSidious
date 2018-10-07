---
description: >-
  BloodHound is a tool to graphically map Active Directory and discover attack
  paths.
---

# BloodHound

## Mapping AD with BloodHound

One of the glorious design features of AD is that everyone in the domain needs to know where everything is. So when you get user credentials and/or a shell, you can basically map the entire domain without breaking any rules. Any user can query Active Directory for computers, domain controllers, groups and sessions.

Now we can use this brilliant feature to collect a ton of information and create a cool GUI map of the entire AD which can be queried using BloodHound. There are two software requirements, you need `BloodHound` and a database to store the data in. The recommended choice is `neo4j`, see below for further instructions.

![Example picture](../.gitbook/assets/image%20%2831%29.png)

## Installing neo4j

#### Linux

* [Install neo4j](https://neo4j.com/developer/kb/how-do-i-enable-remote-https-access-with-neo4j-30x/) [Community Edition](https://neo4j.com/download/community-edition/) manually from their [website](https://neo4j.com/download/?ref=hro) , not through apt.
* [http://neo4j.com/download/other-releases/\#releases](http://neo4j.com/download/other-releases/#releases)
* [A guide on installing can be found here](https://stealingthe.network/quick-guide-to-installing-bloodhound-in-kali-rolling/)
* Run neo4j with `/opt/neo4j-community-3.3.0/bin/neo4j start`
* Navigate to `localhost:7474` in your browser
* Log in with username and password `neo4j` 
* Set a new password for the neo4j account
* Open the file `neo4j.conf`  from the neo4j installation directory and set the following parameters to make any host be able to access the database.

  ```text
  dbms.connector.http.enabled=true
  dbms.connector.http.listen_address=0.0.0.0:7474
  ```

* Restart neo4j with  `/opt/neo4j-community-3.1.1/bin/neo4j restart`
* Access neo4j in the browser at `http://0.0.0.0:7474/browser/`

#### Windows

Neo4j can be started with powershell on windows.

* Spawn an administrative powershell with -bypass exec
* Navigate to the neo4j/bin directory
* `Import-Module .\Neo4j-Management.psd1`
* `Invoke-Neo4j Console`
* Likewise to Linux, log in to `localhost:7474` from your browser and change the password.

#### MacOS

Similar procedure as linux. Neo4j does not support Java 9, so Java SDK must be version 8 and not 9. Install java 8 with cask in Homebrew:

* `brew update`
* `brew cask install java8`

## Getting started with Bloodhound

* Install BloodHound according to instructions on the [Github page](https://github.com/BloodHoundAD/BloodHound/wiki/Getting-started)
* Launch BloodHound and log in to the neo4j database with credentials previously set

## Data collection

To collect data in a format Bloodhound can read is called ingestion. There are several ways of doing this and different types of collection methods. The most useable is the C\# ingestor called SharpHound and a Powershell ingestor called Invoke-BloodHound. Both are bundled with the latest release.

From Bloodhound [version 1.5](https://github.com/BloodHoundAD/BloodHound/releases/tag/1.5): the container update, you can use the new "All" collection open. See the blogpost from [Specter Ops](https://posts.specterops.io/bloodhound-1-5-the-container-update-fdf1ed2ad9da) for details.

Bloodhound is now in [version 2.0](https://github.com/BloodHoundAD/BloodHound/releases/tag/2.0.3.1), so make sure to grab the latest version of the ingestor. For Windows you can use the [SharpHound exe](https://github.com/BloodHoundAD/BloodHound/blob/master/Ingestors/SharpHound.exe).

#### Powershell ingestion

What I recommend doing if you have internal network access is to run Bloodhound using `runas /netonly` from your own machine and not from a host you are not in the control of. This way you're not cluttering a domain joined machine with files, you will not trigger antivirus and you don't have to exfiltrate the data either, so its generally less noisy.

`runas /netonly /FQDN\user\<username> powershell`

**Example with the domain** `testlab.local` **and a username** `testuser`

`runas /netonly /testlab.local\user\testuser powershell` 

Type in the password of testuser when prompted. This should spawn a new Powershell window. This window will use the local DNS settings to find the nearest domain controller and perform the various LDAP queries Bloodhound performs. First, from a powershell shell with execution policy set to bypass, import the powershell module `Import-module SharpHound.ps1` 

Then, start collecting data. This command specifies to collect all kinds of information, compress it into a ZIP and remove stray CSV files generated during ingestion.

`Invoke-BloodHound -CollectionMethod All -CompressData -RemoveCSV`

You will now have get a ZIP file containing CSV files in the directory where you ran the ingestor from. This ZIP file can from v2.0 be dragged and dropped straight into the BloodHound interface.

You should immediately see data being populated into the database and the interface.

You can now play with BloodHound to create really some really cool maps. You can also perform queries to show the shortest path to DA, etc. See the default queries and SpectreOps blog posts for inspiration.

![Example picture](../.gitbook/assets/image%20%289%29.png)

#### Python ingestion from Kali

If you have a Kali box on the local network you can use the [ Bloodhound.py ingestor](https://github.com/fox-it/BloodHound.py).

## More Bloodhound stuff

Explanations of things found under Node info [https://github.com/BloodHoundAD/BloodHound/wiki/Users](https://github.com/BloodHoundAD/BloodHound/wiki/Users) [https://posts.specterops.io/sharphound-target-selection-and-api-usage-bba517b9e69b](https://posts.specterops.io/sharphound-target-selection-and-api-usage-bba517b9e69b) [https://github.com/porterhau5/BloodHound](https://github.com/porterhau5/BloodHound) [https://porterhau5.com/blog/representing-password-reuse-in-bloodhound/](https://porterhau5.com/blog/representing-password-reuse-in-bloodhound/) [https://porterhau5.com/blog/extending-bloodhound-track-and-visualize-your-compromise/](https://porterhau5.com/blog/extending-bloodhound-track-and-visualize-your-compromise/) [http://threat.tevora.com/lay-of-the-land-with-bloodhound/](http://threat.tevora.com/lay-of-the-land-with-bloodhound/)

