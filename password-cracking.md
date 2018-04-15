# Password cracking and auditing

One of the most fun parts of a pentest! Sit back with a cup of coffee and enjoy passwords flowing across the screen for hours on end. I am a sucker for hashcat so this article is pretty much going to be details for using that. John is a viable alternative and Orphcrack can be used if comparing hashes with rainbow tables.

## Hashcat

**Useful links**

* [FAQ](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_can_i_show_previously_cracked_passwords_and_output_them_in_a_specific_format_eg_emailpassword)
* [Command line options](https://hashcat.net/wiki/doku.php?id=hashcat)
* [Hashcat mode codes](https://hashcat.net/wiki/doku.php?id=example_hashes)
* [Hashview, web front end for hashcat](http://www.hashview.io/screenshots.html)

Hashcat can be used to crack all kinds of hashes with GPU. In our case the most relevant things to crack is NTLM hashes, Kerberos tickets and other things you could potentially stumble upon like Keepass databases. The goal is naturally to crack as many as possible as fast as possible, while being smug about all the shitty passwords you'll see. I highly recommend a good GPU, you'll crack faster and have more fun. Even with my not ideal GTX 1060 3GB I'm still cracking NTLM's like it was nothing.

The most basic hashcat attacks are dictionary based. That means a hash is computed for each entry in the dictionary and compared to the hash you want to crack. The hashcat syntax is very easy to understand, but you need to know the different "modes" hashcat uses and those can be found here:. For fast lookup I have added the most commonly seen ones in AD environments below

| Mode | Hash | Description |
| :--- | :----- | :---------- |
| 1000 | NTLM | Extremely common, used for general domain authentication |
| 1100 | MsCache | Domain cached credentials, old version |
| 2100 | MsCache v2 | Domain cached credentials, new version |
| 3000 | LM | Old, rarely used anymore \(still a part of NTLM\) |
| 5500 | NetNTLMv1 / NetNTLMv1+ESS | NTLM for authentication over the network |
| 5600 | NetNTLMv2 | NTLM for authentication over the network |
| 7500 | Kerberos 5 AS-REQ Pre-Auth etype 23 | AS\_REQ is the initial user authentication request of Kerberoas |
| 13100 | Kerberos 5 TGS-REP etype 23 | TGS\_REP is the reply of the Ticket Granting Server to the previous request |

### Dictionary attack

For dictionary attacks, the quality of your dictionary is the most important factor. It can either be very big, to cover a lot of ground. This can be useful for less expensive hashes like NTLM, but with expensive ones like MsCacheV2 you often want a more curated list based on OSINT and certain assumptions or enumerationi \(like password policy\) and instead apply rules.

Here is a very basic dictionary attack using the world famous rockyou wordlist.
```
hashcat64.exe -a 0 -m 1000 ntlm.txt rockyou.txt
```

The limitation here is as with all wordlist attacks the fact that if the password you are trying to crack is not in the list; you won't be able to crack it. This leads us to the next type of attack, a rule-based attack.


### Rules-based attack

Rules are different modifications on words like cut or extend words, add numbers, add special characters and more or less everything you can think of. Like dictionaries, there are also big lists of rules. A rule-based attack is therefore basically like a dictionary attack, but with a lot of modifications on the words. This naturally increases the amount of hashes we are able to crack.

Hashcat has a few built in rules, like the dive.rule which is huge. However, people have used statistics to try and generate rules that are more efficient at cracking. This article details a ruleset aptly named [One Rule to Rule Them All](https://www.notsosecure.com/one-rule-to-rule-them-all/) and can be downloaded from [their Github](https://github.com/NotSoSecure/password_cracking_rules). I have had great success with this rule, and it's statistically proven to be very good. If you need quicker cracking with fewer rules there are plenty of built-in rules in hashcat like the best64.rule. We could probably generate statistics about what works best, but I find experimenting here a lot of fun and

Run rockyou with the best64 ruleset.

```
hashcat64.exe -a 0 -m 1000 -r ./rules/best64.rule ntlm.txt rockyou.txt
```

You are free to experiment with both lists and rules in this part. Only the sky is the limit \(or your GPU / tolerance for hot computer smell\)

After cracking a good amount of the hashes, output the cracked passwords to a new file. The outfile-format 2 specifies to print the passwords only.

```
hashcat64.exe -a 0 -m 1000 ntlm.txt rockyou.txt --outfile cracked.txt --outfile-format 2

Recovered........: 1100/2278 (48.28%)
```

Proceed to run a round with the cracked passwords as a wordlist with a big rule set to recover a few more. You can iterate this a few times, in case you crack a lot of hashes using this technique.

```
hashcat64.exe -a 0 -m 1000 ntlm.txt cracked.txt -r .\rules\OneRuleToRuleThemAll.rule

Recovered........: 1199/2278 (52.63%)

hashcat64.exe -a 0 -m 1000 ntlm.txt cracked.txt -r .\rules\dive.rule

Recovered........: 1200/2278 (52.68%)
```

**Mega attack using the weakpass\_2a \(90 GB\) wordlist and the oneruletorulethemall rule set**

```
hashcat64.exe -a 0 -m 1000 ntlm.txt weakpass_2a.txt -r .\rules\oneruletorulethem.rule
```



### Mask attack
Try all combinations from a given keyspace just like in Brute-Force attack, but more specific.

```
hashcat64.exe -a 3 -m 1000 ntlm.txt .\masks\8char-1l-1u-1d-1s-compliant.hcmask
```


### Recommendations
* [SecLists](https://github.com/danielmiessler/SecLists) - A huge collection of all kinds of lists, not only for password cracking.
* [Weakpass](https://weakpass.com/) has a lot of both good and small lists with both statistics and a calculator for estimating crack time. I'm listing a few of those and some others you should know about below.
* rockyou.txt - Old, reliable, fast
* norsk.txt - A Norwegian wordlist I made myself from downloading Wikipedia and a lot of Norwegian wordlists and combining them, filtering out duplicates naturally.
* weakpass\_2a - 90 GB wordlist, it's huge
* [Keyboard-Combinations.txt](https://github.com/danielmiessler/SecLists/blob/5c9217fe8e930c41d128aacdc68cbce7ece96e4f/Passwords/Keyboard-Combinations.txt) - This is a so-called keyboard walking list following regular patterns on a QWERTY keyboard layout. See chapter below.

#### Generating your own wordlists

Sometimes a wordlist from the internet just doesn't cut it so you have to make your own. There are several scenarios where I have had to make my own lists.
1. I need a non-english language wordlist
2. I need a keyboard walking wordlist
3. I need a targeted wordlist

##### Non-english wordlist

For the first scenario, my friend @tro shared his trick with me. So we download Wikipedia in any given language and then use a somewhat tricky one-liner to trim it into a lowercase-only list without special characters.

```
wget http://download.wikimedia.org/nowiki/latest/nowiki-latest-pages-articles.xml.bz2

bzcat nowiki-latest-pages-articles.xml.bz2 | grep '^[a-zA-Z]' | sed 's/[-_:.,;#@+?{}()&|§!¤%`<>="\/]/\ /g' | tr ' ' '\n' | sed 's/[0-9]//g' | sed 's/[^A-Za-z0-9]//g' | sed -e 's/./\L\0/g' | sed 's/[^abcdefghijklmnopqrstuvwxyzæøå]//g' | sort -u | pw-inspector -m1 -M20 > nowiki.lst

wc -l nowiki.lst
3567894
```

Excellent, we got a 3.5 million word dictionary for a language in only a few minutes.

Another trick that can be used to get dictionaries for specific languages is using google with a specific site: Github. So do a few Google searches like this and pull what you need.

```
greek wordlist site:github.com
greek dictionary site:github.com
```

One thing I noticed is that some of the lists I pulled which had regional characters like ÆØÅ sometimes get replaced by special characters, so rememebr to quickly review the lists you download and replace characters if necessary.

Once you have downloaded a lot of lists and fixed potential errors, use the Linux command line to concatenate them, trim away special characters and make them all lowercase

```
sed -e 's/[;,()'\'']/ /g;s/ */ /g' list.txt | tr '[:upper:]' '[:lower:]' > newlist.txt
```

You should now have a pretty good working list in a specific language and you should start to understand why learning things like cut, tr, sed, awk, piping and redirection is so damn applicable.

**Bonus**
I discovered that you can find lists with names and places. These are often used for passwords. People love their kids and grandkids and thus use it as password. I found such things on [Github](https://gist.github.com/eiriks/8b028e05d9b53f8de628) by a little Googling.. Now all these were in JSON, but that is not a concern.

Linux magic to the rescue

```
cat *.json | sed 's/,/\n/g' | cut -f '"' -f2 | sort -u > nornames.txt

wc -l nornames.txt
9785
```

So we have added a few more words.

I can now add this to my other Norwegian list and filter duplicates. Put them both in the same file

```
cat norsk.txt nor_names.txt sort -u > norsk.txt

wc norsk.txt -l
2191221
```

Awesome, more than 2 million unique Norwegian words.

##### Keyboard walking wordlist

Keyboard walking is following regular patterns on a QWERTY keyboard layout to make a password that's easily rememberable. Apparently people think this generates secure passwords, but in reality they are highly predictable. Hence, these patterns can be generated from a keymap and wordlists can easily be generated.

Hashcat published a keyboard-walk generator a few years ago called [kwprocessor](https://github.com/hashcat/kwprocessor). You can use this to generate pretty big lists based on a number of patterns and sizes. A quick example of generating a 2-16 character long list.

```
/kw.out -s 1 basechars/full.base keymaps/en.keymap routes/2-to-16-max-3-direction-changes.route -o words.txt
```

Remember it does not necessarily make much sense running rule based attacks on this kind of list.

Another options for keyboard walking, is using the `Keyboard-Combinations.txt` list mentioned above.

##### Target wordlist

Often in pentesting engagements you are in an enterprise with very specific names and details. More than often enough, people set passwords with the name of the company for both service accounts and user accounts. A very simple trick can be to just write a few company related names into a list, but a more effective way is to use the web crawling tool Cewl on the enterprise's public website.

```
cewl -w list.txt -d 5 -m 5 http://example.com
```

We should now have a decently sized wordlist based on words that are relevant for the specific enterprise, like names, locations and a lot of their business lingo.

Another targeted possibility is cracking with the usernames as a wordlist, but note that certain password policies does not allow this.

Also, if you have dumped a database from a domain controller you probably also have access to the full names of employees. So a neat trick would be to make a wordlist with every first and last name and use that for password cracking with rules. That could provide some extra results.


### Useful hashcat options you can play with

* Print hashes that haven't been cracked using `--left`
* Print hashes that haven't been cracked using `--left`
* Print cracked password  in this format `hash:password` using `--show`
* Print cracked password in this format `username:hash:password` using `--show --username`
* Burn your GPU with `-w <number>` where the scale is 1 to 3
* Write cracked hashes to file using `--outfile cracked.txt --outfile-format 2` where 2 is the output format. See `--help` for possible values
* Start hashcat as a session that can be stopped and resumed with `--session <id>` where the id is juts a number. When restoring a session use the same parameter with the same id.

### Online cracking tools

To be honest, I prefer not using these and especially not in pentesting engagements. You do not want to submit something you don't know what contains to an online repository for eternal storage. Odds are it won't ever be detected, but err on the side of caution here. If you decide to submit hashes from a lab or hashes you know the plaintext for already, [Crackstation.net](https://crackstation.net/) is a good choice.

## Domain Password Audit Tool \(DPAT\)

[clr2of8/DPAT](https://github.com/clr2of8/DPAT)  
A python script that will generate password use statistics from password hashes dumped from a domain controller and a password crack file such as hashcat.potfile generated from the Hashcat tool during password cracking. The report is an HTML report with clickable links.

## Other

#### Rainbow tables

Rainbox tables are pre-computed hashes you can use to compare against if hashes are not salted, like NTLM.  
[Free rainbow tables](https://web.archive.org/web/20160402172945/https://www.freerainbowtables.com/en/tables2
)

