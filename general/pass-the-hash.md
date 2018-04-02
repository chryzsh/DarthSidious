## Passing the hash

The first goal of a Windows pentest is to get a user or a shell as a user.

You **CAN** perform Pass-The-Hash attacks with NTLM hashes.

You **CANNOT** perform Pass-The-Hash attacks with Net-NTLM hashes.

You can pass the hash using a metasploit module called [PSExec](https://www.offensive-security.com/metasploit-unleashed/psexec-pass-hash/).

If you want test your newly found hash across multiple machine smb\_login Metasploit module is how it's done. However, trying this with a domain hash will lock the account out of the domain, assuming they have a lockout policy.

Empire also has options for performing pass-the-hash in the credentials/mimikatz/pth module. https://www.powershellempire.com/?page_id=270





#### Ranger

There is an insane tool named Ranger that can interact with Windows based systems in oh so many ways.

https://github.com/funkandwagnalls/ranger

Ranger is a command-line driven attack and penetration testing tool, which as the ability to use an instantiated catapult server to deliver capabilities against Windows Systems. As long as a user has a set of credentials or a hash set \(NTLM, LM, LM:NTLM\) he or she can gain access to systems that are apart of the trust.











