### Passing the hash
The first goal of a Windows pentest is to get a user or a shell as a user.

You **CAN** perform Pass-The-Hash attacks with NTLM hashes.

You **CANNOT** perform Pass-The-Hash attacks with Net-NTLM hashes.

You can pass the hash using a metasploit module called [PSExec](https://www.offensive-security.com/metasploit-unleashed/psexec-pass-hash/).
