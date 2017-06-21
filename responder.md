### Responder attack

Responder does not pick up on FQDN, but it does pick up on NetBIOS and LLMNR, because Windows boxes are VERY chatty! Responder works because the windows box are looking for resources and default to NetBIOS like described in the Windows guide part of this tutorial.

Responder captures NetNTLMv2 hashes. You can not pass the hash with these but you can crack and relay them. This part will only explain how to run Responder and capture hashes.

Run Responder:
```
Responder -I eth0 -wrf
```

 Where eth0 is your interface. The -wrf is optional, but -f is useful as it fingerprints the OS version. The other options are explained with -h.

Open explorer on the Windows machines and write `\\share\` and wait for the credential prompt. This will trigger a name resolution request over SMB to find a resource using the users credentials. That should allow Responder to capture the NetNTLMv2 hash of the user making the request and print them in your terminal.

You can now crack the hashes using your favorite tool, but while that works it's magic you can read on for something even cooler.
