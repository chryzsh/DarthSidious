### Sharpen your knives
You need a lot of tools for this. Get ready! Make a directory to put all this in, because it can get messy. I won't explain all the tools. Some things you gotta ready yourself.

- [Impacket](https://github.com/CoreSecurity/impacket) - Includes ntlmrelay, which will be useful later.
- [Responder](https://github.com/lgandx/Responder) - Poisons requests.
- [Empire](https://github.com/EmpireProject/Empire) - Imagine it as a Metasploit for Windows hacking.
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - Includes tools to check for SMB signing.
- [DeathStar](https://github.com/byt3bl33d3r/DeathStar) - Insanely cool tool for automating the "getting DA" process. Still in development, so may not always be stable.
- [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Creating a map of AD. Check the /releases on the GitHub page for precompiled binaries!
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - Powershell tools, some are included in Empire already.
- [Mimikatz](https://github.com/gentilkiwi/mimikatz) - Dumping credentials
- [Neo4j](https://neo4j.com/download/) - Database to use with BloodHound

Now you may need to trick a bit with installing the dependencies for all these tools. Familiarize yourself with pip. Make sure you are running tools and scripts with the correct Python version. Certain tools are written for 2.7 and some for 3.
