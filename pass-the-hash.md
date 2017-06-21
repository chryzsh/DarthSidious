### Passing the hash
The first goal of a Windows pentest is to get a user or a shell as a user.

In a windows AD environment 9 times out of 10 all the workstations share the same local administrator password. You might be really lucky and both the workstations and servers share the same local administrator password as well. At that point it becomes a discovery mission for where the domain admins are logged in.

You **CAN** perform Pass-The-Hash attacks with NTLM hashes.

You **CANNOT** perform Pass-The-Hash attacks with Net-NTLM hashes.

You can pass the hash using a metasploit module called [PSExec](https://www.offensive-security.com/metasploit-unleashed/psexec-pass-hash/).
