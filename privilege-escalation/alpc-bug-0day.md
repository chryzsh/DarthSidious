# ALPC bug 0day

[https://github.com/SandboxEscaper/randomrepo/blob/master/PoC-LPE.rar](https://github.com/SandboxEscaper/randomrepo/blob/master/PoC-LPE.rar)

{% embed data="{\"url\":\"https://www.theregister.co.uk/2018/08/28/windows\_0day\_pops\_up\_out\_of\_span\_classstrikenowherespan\_twitter/\",\"type\":\"link\",\"title\":\"Windows 0-day pops up out of nowhere Twitter\",\"description\":\"Local privilege escalation in procedure calls\",\"icon\":{\"type\":\"icon\",\"url\":\"https://www.theregister.co.uk/Design/graphics/icons/vulture\_black\_60\_trans.png\",\"aspectRatio\":0},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://regmedia.co.uk/2017/05/03/panic\_shutterstock.jpg?x=1200&y=794\",\"width\":1200,\"height\":794,\"aspectRatio\":0.6616666666666666}}" %}

## How to use

A 0day for a local priv esc for Windows was published August 28th on Twitter by @sandboxescaper, whose account was pulled quickly. The PoC is on [Github](https://github.com/SandboxEscaper/randomrepo). The video posted with the PoC wasn't evident so I made a quick reproduction to verify whether it works, and it certainly does.

* As Administrator, open [Process Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer) - right click, "Run as administrator"
* As a regular user, launch notepad. If you opened it from cmd, you get a subprocess notepad inside cmd. This thread runs with the user context you launched it with. Note that the PID of the notepad process is `3872`

![](../.gitbook/assets/image%20%285%29.png)

![](../.gitbook/assets/image%20%288%29.png)

* If you need to see username and  integrity level in Process Explorer you can go to View -&gt; Select columns and check 

![](../.gitbook/assets/image%20%2810%29.png)

Now, have a look at the process spoolsv.exe which is basically where the actions is going to happen. Nothing much here yet.

![](../.gitbook/assets/image%20%2812%29.png)

Now fire the exploit off and see what happens \(this is demonstrated in the PoC video\). We use the PID of the notepad process we spawned earlier `3872`

![](../.gitbook/assets/image%20%282%29.png)

Now, it appears that nothing is happening, but take a look at spoolsv in Process Explorer again.

![](../.gitbook/assets/image%20%281%29.png)

Bham! cmd.exe with subprocesses conhost and notepad has spawned as SYSTEM! 

### Windows 10 - works!

0day priv esc confirmed on Windows 10 1803. No patch has been released by MS yet \(28.08.2018\)

![](../.gitbook/assets/image%20%283%29.png)

This could probably be tweaked to open an actual cmd window as SYSTEM instead of a windowless process in the background.

Edit on the above: @plaintext notified me that the processes spawn in session 0 which is why they won't be visible to the user which operations in session 1. If you toggle Session in the columns panel in ProcExplorer you can see that very clearly.

![](../.gitbook/assets/image%20%284%29.png)

###  Server 2016 - works

![](../.gitbook/assets/image%20%2811%29.png)

### Windows 7 - Nothing happens

![](../.gitbook/assets/image%20%2813%29.png)

\*\*\*\*

\*\*\*\*

**Bonus**: when you kill the spawned cmd.exe process, a popup asking you to save the print output spawns.

![](../.gitbook/assets/image%20%286%29.png)

