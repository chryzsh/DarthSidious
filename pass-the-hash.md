### Passing the hash

The first goal of a Windows pentest is to get a user or a shell as a user.

You **CAN** perform Pass-The-Hash attacks with NTLM hashes.

You **CANNOT** perform Pass-The-Hash attacks with Net-NTLM hashes.

You can pass the hash using a metasploit module called [PSExec](https://www.offensive-security.com/metasploit-unleashed/psexec-pass-hash/).

If you want test your newly found hash across multiple machine smb\_login module is how it's done. However, trying this with a domain hash will lock the account out of the domain, assuming they have a lockout policy.



There is a metasploit module to do this as well.





