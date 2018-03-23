### Sharpen your lightsaber

You need some tools for this guide. Get ready! Make a directory to put all this in, because it can get messy. I won't explain all the tools and how to install them, because the different tools and procedures might change.

* [Impacket](https://github.com/CoreSecurity/impacket) - A python collection for networking. Includes ntlmrelay, which will be useful later.
* [Responder](https://github.com/lgandx/Responder) - Tool for poisoning requests.
* [Empire](https://github.com/EmpireProject/Empire) - A framework almost like Metasploit, but for Windows hacking.
* [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - Includes tools to check for SMB signing.
* [DeathStar](https://github.com/byt3bl33d3r/DeathStar) - An insane tool for automating the process of becoming Domain Admin.
* [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Creating a map of AD. Check /releases on the GitHub page for precompiled binaries!
* [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - Powershell tools for post exploitation, some are included in Empire already.
* [Mimikatz](https://github.com/gentilkiwi/mimikatz) - Dumping credentials.
* [Neo4j](https://neo4j.com/download/) - Database to use with BloodHound.

Most of these should be available through apt. If not, install them by cloning in git and putting them in /opt. Now you may need to  play a litte with installing the tools and their dependencies. A lot of them are written in Python, so familiarize yourself with pip. Make sure you are running tools and scripts with the correct Python version. Certain tools are written for 2.7 and some for 3. How to use the tools are explained in subsequent parts of this tutorial.

