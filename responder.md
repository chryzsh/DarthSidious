### Responder attack

Responder does not pick up on FQDN.

Responder captures NetNTLMv2 hashes. You can not pass the hash with these but you can relay them.

Responder picks up on NetBIOS and LLMNR, because Windows boxes are VERY chatty!
Responder works because the windows box are looking for resources and default to NetBIOS.

Open explorer on the Windows machines and write `\\share\` and wait for the credential prompt. This will trigger a request over SMB to find a resource using the users credentials. That should allow Responder to capture the NetNTLMv2 hash of the user and print them in your terminal.

You can now crack the hashes using your favorite tool, or you can read on for something even cooler.
