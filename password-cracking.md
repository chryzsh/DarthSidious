# Password cracking and auditing

One of the most fun parts of a pentest! Sit back with a cup of coffee and enjoy passwords flowing across the screen for hours on end. I myself am a sucker for hashcat so this article is pretty much going to be about that

## Hashcat

* [FAQ](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword)
* [Command line options](https://hashcat.net/wiki/doku.php?id=hashcat)
* [Hashcat mode codes](https://hashcat.net/wiki/doku.php?id=example_hashes)
* [Hashview, web front end for hashcat](http://www.hashview.io/screenshots.html)

Hashcat can be used to crack all kinds of hashes with GPU. In our case the most relevant things to crack is NTLM hashes, Kerberos tickets and other things you could potentially stumble upon like Keepass databases. The goal is naturally to crack as many as possible as fast as possible, while being smug about all the shitty passwords you'll see. I highly recommend a good GPU, you'll crack faster and have more fun. Even with my not ideal GTX 1060 3GB I'm still cracking NTLM's like it was nothing.

The most basic hashcat attacks are dictionary based. That means a hash is computed for each entry in the dictionary and compared to the hash you want to crack. The hashcat syntax is very easy to understand, but you need to know the different "modes" hashcat uses and those can be found here:. For fast lookup I have added the most commonly seen ones in AD environments below

| Mode | Hash | Description |
| --- | --- | --- |
| 1000 | NTLM | Extremely common, used for general domain authentication |
| 1100 | MsCache | Domain cached credentials, old version |
| 2100 | MsCache v2 | Domain cached credentials, new version |
| 3000 | LM | Old, rarely used anymore \(still a part of NTLM\) |
| 5500 | NetNTLMv1 / NetNTLMv1+ESS | NTLM for authentication over the network |
| 5600 | NetNTLMv2 | NTLM for authentication over the network |
| 7500 | Kerberos 5 AS-REQ Pre-Auth etype 23 | AS_REQ is the initial user authentication request of Kerberoas |
| 13100 | Kerberos 5 TGS-REP etype 23  | TGS_REP is the reply of the Ticket Granting Server to the previous request |


### Dictionary attack

For dictionary attacks, the quality of your dictionary is the most important factor. It can either be very big, to cover a lot of ground. This can be useful for less expensive hashes like NTLM, but with expensive ones like MsCacheV2 you often want a more curated list based on OSINT and certain assumptions (like password policy) and instead apply rules.

#### Dictionary recommendations
[Weakpass](https://weakpass.com/) has a lot of both good and small lists with both statistics and a calculator. A few I've used from there is 
* rockyou.txt - 


Here is a very basic dictionary attack using the world famous rockyou wordlist. 
```
hashcat64.exe -a 0 -m 1000 ntlm.txt rockyou.txt
```
The limitation here is as with all wordlist attacks the fact that if the password you are trying to crack is not in the list; you won't be able to crack it. This leads us to the next type of attack.


### Rules-based attack
Rules are different modifications on words like cut or extend words, add numbers, add special characters and more or less everything you can think of. Like dictionaries, there are also big lists of rules. A rule-based attack is therefore basically like a dictionary attack, but with a lot of modifications on the words. This naturally increases the amount of hashes we are able to crack.

##### Rule recommendations
Hashcat has a few built in rules, like the dive.rule which is huge. However, people have used statistics to try and generate rules that are more efficient at cracking. This article details a ruleset aptly named [One Rule to Rule Them All](https://www.notsosecure.com/one-rule-to-rule-them-all/) and can be downloaded from [their Github](https://github.com/NotSoSecure/password_cracking_rules). I have had great success with this rule, and it's statistically proven to be very good.

The next step is to start adding rules. That means each word in the wordlist is slightly changed.

```
hashcat64.exe -m 1000 -r ./rules/best64.rule ntlm.txt rockyou.txt
```

```
.\hashcat64.exe -a 0 -m 1000 ntlm.txt rockyou.txt  -potfile .\hashcat.potfile --session 1 --separator ":" -w 2
```

After cracking a good amount of the hashes, output the cracked passwords to a new file.

```
.\hashcat64.exe -a 0 -m 1000 ..\onlyntlm.txt .\rockyou -potfile ..\hashcat.potfile --show
 --separator ":" --outfile cracked.txt --outfile-format 2
```

Proceed to run a round with the new found passwords and recover a few more.

```
.\hashcat64.exe -a 0 -m 1000 ..\onlyntlm.txt .\cracked.txt -potfile ..\hashcat.potfile --
separator ":" -r .\rules\OneRuleToRuleThemAll.rule

Recovered........: 1199/2278 (52.63%)

.\hashcat64.exe -a 0 -m 1000 ..\onlyntlm.txt .\cracked.txt -potfile ..\hashcat.potfile --
separator ":" -r .\rules\dive.rule

Recovered........: 1200/2278 (52.68%)
```

## 

## 

## Domain Password Audit Tool \(DPAT\)

[https://github.com/clr2of8/DPAT](https://github.com/clr2of8/DPAT)

