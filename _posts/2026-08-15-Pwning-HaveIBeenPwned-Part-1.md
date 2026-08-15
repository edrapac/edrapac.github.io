---
title: 'Pwning HaveIBeenPwned: Part 1'
date: 2026-08-15
permalink: /posts/2026-08-15-Pwning-HaveIBeenPwned-Part-1/
tags:
  - PasswordCracking
  - Hashcat
  - HaveIBeenPwned
  - Wordlists
  - Rules
  - Research
---

Lets deep dive into the HaveIBeenPwned Dataset! (Obligatory: JP taught me most things I know about password cracking ♥️ )

![HaveIBeenPwned password cracking](/images/hibp-part1/hibp-header.png)

What up everyone. Recently I've developed an obsession for password cracking. Today I've got some cool research to share regarding the HaveIBeenPwned dataset. The motivation for this project was pretty simple: how can I better improve my password cracking methodology, and can we learn something about how people like to create passwords?

In this series, we'll look at several popular as well as novel attack techniques for cracking passwords hashes including things like dictionary attacks, effective character mask generation, character and word level mangling, and finally how to use more advanced tooling like PACK and OMEN to leverage some deterministic as well as machine learning to more effectively crack passwords.

**What we will address in this part:** This post will provide an introduction to the project, as well as cover most effective wordlist and ruleset combinations for cracking passwords via dictionary attacks using hashcat + HaveIBeenPwned dataset.

For those of you unfamiliar with [HaveIBeenPwned](https://haveibeenpwned.com/), its a site ran by [Troy Hunt](https://www.troyhunt.com/bio/) that maintains a massive trove of compromised passwords, and the user's to whom they belong. You can visit the site, type in your email address, and the site will tell you whether or not you've been involved in any databreaches that HaveIBeenPwned currently indexes. It's a super cool service (not to mention Troy Hunt is just a good human being for offering this for free) and interestingly the site provides all of the passwords it indexes for download (free of charge) ! Its important to call out that **the site does not attribute these passwords to usernames or emails**. Thats right — theres no identifiable information to be found here (unless your password is your name... more on that later). What you *can* download however is many gigabytes of passwords, all hashed with the SHA-1 algorithm. When I first decided to do a little exercise into which wordlists and rulesets are best for password cracking, I realized this makes an excellent candidate as these are real passwords and number in the **billions**.

## SHA-1

- - -

Lets talk about the hashes for a moment. SHA-1 is a cryptographic hashing function, a one-way function meant to receive a plaintext input (the hash preimage) and produce a fixed length string (hash digest) as output. Every SHA-1 hash will be the same length regardless of input size, and a specific input's digest will always be the same, every time. I know this is elementary hashing basics but I don't write these blogs only for the veteran pentesters!

SHA-1 itself is considered insecure for several reasons. The first reason being that in 2017 a group of researchers [successfully demonstrated](https://security.googleblog.com/2017/02/announcing-first-sha1-collision.html) generating a collision. If you aren't familiar with collisions or want an even deeper understanding of the SHA-1 algo i'd recommend Aditya Anand's fantastic [blog here](https://infosecwriteups.com/breaking-down-sha-1-algorithm-c152ed353de2)

The other and more applicable reason SHA-1 is considered an insecure choice is it's poor resistance to bruteforcing via things like dictionary attacks. When we use wordlists to attempt to crack a hash, all we are doing is simply taking a word from that wordlist, computing it's SHA-1 hash, and then comparing that to the hash we are trying to crack. If they match, we know we've just successfully "cracked" that hash and recovered it's plaintext value.

When we repeat this process at scale, for example using a wordlist of millions of guesses (candidates) against not just a single password hash but rather thousands or even millions of hashes, we need to consider how "fast" computing the hash becomes. If you use a wordlist with one million candidates, you will end up computing one million SHA-1 hashes **for each password** you're trying to crack!

Thankfully SHA-1 helps us here. A hash's resistance to cracking is mainly measured in two ways — how computationally expensive the hash is to compute, and how much memory a single hash takes when computing. The first one is generally called it's compute cost whereas the latter is referred to as memory hardness. If you want to learn more about either of these terms, I'd again refer you to the blog post about SHA-1 I linked above, suffice to say, SHA-1 is both fast to compute, as well as takes up very little memory when computing, allowing us to compute many, many hash candidates concurrently and with very little memory overhead.

## Rules, and how they affect count

- - -

Last technical intro before we dive in — lets talk about rules for a sec. Rules are an incredibly useful way to boost the cracking ability of your given wordlist. Rules perform various transformations on a candidate before then hashing that candidate. For example, lets say we are trying to crack a hash and we have a wordlist of common, bad passwords. One of the entries in that wordlist is simply `password`. Using rules allows us to not only guess `password` but also perform mutations on that word too, so transforming it to candidates like `p@ssword` and `pa$$word` — see the usefulness here? We've just extended 1 initial candidate into 3. Rulesets define their rules much like our wordlists, each ruleset has one rule per line. When we apply rulesets to wordlists, we apply **each rule to every candidate**

These transformations come with a price though. **For each ruleset you use, you multiply number of candidates x number of rules**

Lets look at an example. We have a wordlist with 2 entries `password` and `root`. We decide to use 2 rulesets as well, **ruleset 1** capitalizes the first letter of each candidate, and it also adds `@` to the end of the candidate. **Ruleset 2** capitalizes the last letter of each candidate, and also adds a `2` to the end of the candidate

```
candidates X ruleset 1 X ruleset 2 = total candidates
2          x    2      x     2     = 8
```

As you can see, with large wordlists and rulesets, the number of total candidates generated becomes very large! For this reason, we will only be using one ruleset per wordlist rather than stacking rulesets.

## Getting the data locally

- - -

Downloading the entire dataset was actually really straightforward, theres a really helpful guide on the official [HaveIBeenPwned github](https://github.com/HaveIBeenPwned/PwnedPasswordsDownloader/issues/79)

Following that guide leads me to the first mistake I made. After successfully downloading all the fragmented files, I foolishly combined them into 1 giant password dump of about 79GB, and **2,068,408,781 passwords (thats 2 billion 68 million!!!)** An unbelievably large number! To put that in context, thats a little less than the entire estimated population of both India and China combined!

```
$ ls -alh
-rw-rw-r--  1 ubuntu ubuntu 79G Jul 10 08:49 hibp_all.txt
```

Hashcat was entirely incapable of dealing with this large of a file, but fortunately it was easy to break it up into 79, 1 GB chunks

```
split -C 1G hibp_all.txt -d piece_
```

That gave me the following results

```
-rw-rw-r-- 1 ubuntu ubuntu  1073741824 Jul 10 13:35 piece_00
-rw-rw-r-- 1 ubuntu ubuntu  1073741824 Jul 10 13:35 piece_01
-rw-rw-r-- 1 ubuntu ubuntu  1073741824 Jul 10 13:35 piece_02
-rw-rw-r-- 1 ubuntu ubuntu  1073741824 Jul 10 13:35 piece_03
...
```

Lets take a look at the contents of one of these badboys

```
$ head -n 5 piece_00
000000005AD76BD555C1D6D771DE417A4B87E4B4
00000000A8DAE4228F821FB418F59826079BF368
00000000DD7F2A1C68A35673713783CA390C9E93
0000000173CE12FC3F90E1F560B14AC3FAE65056
00000001E225B908BAC31C56DB04D892E47536E0
```

The nice thing about this is that these are sorted and de-duplicated SHA-1 hashes, they are entirely random and have 0 influence over one another. By that I mean, while the hashes might look similar, even 2 similar hashes could have entirely different plaintext values. What this means for us is that each 1GB sample is well and truly random!

However...

These 1GB samples also had horrible runtime! Dang. So I decided lets cut it down a bit more. If these are truly random, then taking a sample from *them* should also be a good random representative of the whole that we can draw conclusions from, right? Its sort of like saying we could perform 1 billion coinflips, or 1 million coin flips, both of which would deduce the likelihood of heads to 50%. So, whats a good sample size? I figured 1 million passwords per cracking run were more than sufficient because bro science.

Here's how i'd generate a random sample per hashcat run

```bash
echo "Sampling 1000000 hashes from $HIBP_FILE ..."

MAXOFF=$((FILESIZE - BLOCK))
(( MAXOFF < 0 )) && MAXOFF=0

: > "$SAMPLE"
count=0
while (( count < 1000000 )); do
    off=$(shuf -i 0-"$MAXOFF" -n 1)
    # read one block at a random byte offset, keep only whole 40-hex lines
    # (this drops the leading/trailing partial lines automatically)
    dd if="$HIBP_FILE" bs="$BLOCK" skip="$off" iflag=skip_bytes count=1 status=none \
        | grep -E '^[0-9A-Fa-f]{40}$' >> "$SAMPLE"

    count=$(wc -l < "$SAMPLE")
    printf '\r  collected: %d / %d hashes' "$count" "1000000"
done
echo

# trim to exactly N hashes
head -n "1000000" "$SAMPLE" > "$SAMPLE.tmp"
mv "$SAMPLE.tmp" random_1M_sample.txt
```

## Technical Specs

- - -

So, where to start? [Hashmob](https://hashmob.net/) provides some excellent starting points and the ratings system provided by [Weakpass](https://weakpass.com/), a site that indexes wordlists based on their usefulness for password cracking, is also a good resource. At this point too I should mention the type of cracking rig I'm working with

* **CPU:** Intel(R) Core(TM) i5-6600K CPU @ 3.50GHz
* **RAM:** 16GB DDR5
* **Graphics Card:** NVIDIA GeForce RTX 3060
* **Storage:** 1TB NVME SSD

Modest specs that also dictate what we can and cannot try in the simple interest of time — as a pentester I wanted any of my recommendations to carry an element of feasibility, it sure would be cool if we can get a 100% crack rate but if that takes a week to do so, thats too long for me. The rough rule I held this research to was if the wordlist + ruleset combo took more than 24 hrs to run, I wasn't going to consider it.

## Cracking some hashes!

- - -

Alright, so we've got some idea of wordlists and rulesets to try. Lets get these on the box and start crackin! First the wordlists — all of these can be found on Weakpass with the exception of `hashcracky-main.txt` and `passphrases.txt` which you can find [here](https://hashcracky.com/static/resources/WORDLISTS/hashcracky-main.txt.zst) and [here](https://github.com/initstring/passphrase-wordlist/releases) respectively. These are all the wordlists I chose (these are in no particular order, and some are pretty large so heads up). Also: some of these don't have extensions, others are .txt. They're all text files, I was just too lazy to add the extension if the file extracted without one after I downloaded it 🤷‍♂

```
hashmob.net_2025.found
all_in_one.txt
all-h.txt
weakpass_4.txt
Hashes.org
hashesorg2019
SAWL
cyclone_hk_v2.txt
kaonashi.txt
hashmob.net_2025.large.found
linuxwz.txt
hashmob.net_2025.small.found
ASLM.txt
crackstation.txt
ignis-1M.txt
breach.txt
hashcracky-main.txt
rockyou2024.txt
passphrases.txt
```

Lets get some rulesets now! Caeksec has a [fantastic repo](https://github.com/caeksec/rules/tree/main) of popular rulesets. I also vibe coded a [quick UI](https://edrapac.github.io/tools/hashcat-rules/) overlay since I always forget what rule does what

![Hashcat rules reference UI](/images/hibp-part1/hashcat-rules-ui.png)

Anyhow, now that we've got our rulesets, and wordlists, its time to talk about how I actually went about cracking these hashes since some of the command line flags warrant a little discussion

```
hashcat -a 0 -m 100 -w 4 -O --potfile-disable --loopback \
-r top_5000.rule -o ASLM_top_5000_output \
random_1M_sample.txt ASLM.txt
```

This is one of the many invocations of hashcat I ran, while most of the flags are pretty standard, a few are important callouts

`-O` — Optimized kernels. You'll want to get these running for whichever graphics driver you have (nvidia or opencl)

`-w 4` — nightmare workload profile instructs hashcat to use as much system resources (and power!) as it needs

`--potfile-disable` — since I wanted each cracking run to be independent of the last, I didn't want a previous run's potfile to interfere with the current run. I wanted cracked hashes to be retrieved only from wordlist+rules combos and not because I already had the hash in my potfile. Now, since all these passwords are de-duped, this actually shouldnt be an issue from a cracking perspective, however lets say we are running hashcat a total of 30 times, against a new 1GB chunk each run. Even if hashcat gets a ~50% success rate, by the time the last hashcat run happens, the potfile has now grown to almost 15GB! Hashcat is notoriously slow at parsing potfiles, and in fact, it *needs to* parse the potfile before each run unless instructed not to, so without this disabled, before a run can take place, hashcat would have to first parse an increasingly large potfile. No fun.

`-o` simple output flag, just for record keeping :)

SO what did this end up looking like?

Here's some raw output from a run where I tested the `all-h.txt` wordlist with a personal favorite `yubaba.64` rule against a random 1 million sample. Impressively the entire run took around 7 mins and recovered an incredible 81.9%!

```
hashcat -a 0 -m 100 -w 4 -O --potfile-disable --loopback \
-r yubaba64.rule -o all-h-1mil_output \
random_1M_sample.txt all-h.txt

Session..........: hashcat
Status...........: Exhausted
Hash.Mode........: 100 (SHA1)
Hash.Target......: /home/ubuntu/Desktop/random_1M_sample.txt
Time.Started.....: Sun Jul 19 10:53:55 2026 (7 mins, 32 secs)
Time.Estimated...: Sun Jul 19 11:00:28 2026 (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (/home/ubuntu/Desktop/Wordlists/all-h.txt)
Guess.Mod........: Rules (/home/ubuntu/Desktop/hashcat-7.1.2/rules/kaonashi/yubaba64.rule)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   453.5 MH/s (47.76ms) @ Accel:256 Loops:64 Thr:512 Vec:1
Recovered........: 819045/1000000 (81.90%) Digests (total), 819045/1000000 (81.90%) Digests (new)
Remaining........: 180955 (18.10%) Digests
Recovered/Time...: CUR:104944,N/A,N/A AVG:127707.59,N/A,N/A (Min,Hour,Day)
Progress.........: 174345606464/174345606464 (100.00%)
Rejected.........: 0/174345606464 (0.00%)
Restore.Point....: 2724150101/2724150101 (100.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-64 Iteration:0-64
Candidate.Engine.: Device Generator
Candidates.#1....: $HEX[efbfbdefbfbd323931efbfbdefbfbdefbfbd] -> $HEX[ffffff7a7a3738]
Hardware.Mon.#1..: Temp: 55c Fan: 53% Util: 13% Core:1980MHz Mem:7301MHz Bus:16
```

This seemed a bit too good to be true, but re running that against a larger, 5 Million sample had the same results and an even better runtime of only 13 minutes !

```
hashcat -a 0 -m 100 -w 4 -O --potfile-disable --loopback \
-r yubaba64.rule -o all-h-5mil_output \
random_5M_sample.txt all-h.txt

Session..........: hashcat
Status...........: Exhausted
Hash.Mode........: 100 (SHA1)
Hash.Target......: /home/ubuntu/Desktop/random_5M_sample.txt
Time.Started.....: Sun Jul 19 11:02:08 2026 (13 mins, 17 secs)
Time.Estimated...: Sun Jul 19 11:15:25 2026 (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (/home/ubuntu/Desktop/Wordlists/all-h.txt)
Guess.Mod........: Rules (/home/ubuntu/Desktop/hashcat-7.1.2/rules/kaonashi/yubaba64.rule)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   218.9 MH/s (53.71ms) @ Accel:1024 Loops:64 Thr:128 Vec:1
Recovered........: 4081470/4980791 (81.94%) Digests (total), 4081470/4980791 (81.94%) Digests (new)
Remaining........: 899321 (18.06%) Digests
Recovered/Time...: CUR:274679,N/A,N/A AVG:307304.40,N/A,N/A (Min,Hour,Day)
Progress.........: 174345606464/174345606464 (100.00%)
Rejected.........: 0/174345606464 (0.00%)
Restore.Point....: 2724150101/2724150101 (100.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-64 Iteration:0-64
Candidate.Engine.: Device Generator
Candidates.#1....: $HEX[efbfbdefbfbdefbfbdefbfbdefbfbdefbfbd35363336343837] -> $HEX[ffffff7a7a3738]
Hardware.Mon.#1..: Temp: 54c Fan: 51% Util:  4% Core:1980MHz Mem:7301MHz Bus:16
```

## Results

- - -

This process was repeated over 29 different wordlist+ruleset combos — a random 1 million sample was drawn, and I then ran hashcat. To save time, I'm just going to post the results in a big table here:

<div style="overflow-x: auto;" markdown="1">

| Wordlist + Ruleset | % recovered | Runtime | Wordlist Size (pre ruleset) | Generated Candidates |
|---|---|---|---|---|
| hashmob.net_2025.found + HashMob.10k | 89.50% | 1h 32m 24s | 23.8 GB | 18,714,648,590,000 (238.1 TB) |
| all_in_one.txt + top_500 | 88.63% | 1h 35m 01s | 340.6 GB | 14,815,405,056,500 (170.3 TB) |
| all-h.txt + top_1500 | 88.33% | 21m 05s | 30.6 GB | 4,087,550,095,500 (45.9 TB) |
| all_in_one.txt + yubaba64 | 85.09% | 1h 04m 31s | 340.6 GB | 1,896,369,583,680 (21.8 TB) |
| all-h.txt + yubaba64 | 81.94% | 7m 32s | 30.6 GB | 174,398,047,360 (2.0 TB) |
| hashmob.net_2025.found + yubaba64 | 75.51% | 4m 40s | 23.8 GB | 119,764,794,496 (1.5 TB) |
| weakpass_4.txt + top_5000 | 74.52% | 48m 31s | 24.1 GB | 10,962,230,650,000 (120.6 TB) |
| Hashes.org + top_5000 | 70.64% | 31m 24s | 15.0 GB | 6,989,721,605,000 (75.1 TB) |
| hashesorg2019 + top_5000 | 68.69% | 28m 32s | 13.7 GB | 6,402,079,925,000 (68.7 TB) |
| SAWL + top_5000 | 65.80% | 33m 10s | 18.3 GB | 7,450,638,785,000 (91.4 TB) |
| cyclone_hk_v2.txt + OneRuleToRuleThemStill | 64.02% | 1h 56m 22s | 6.5 GB | 29,508,561,901,392 (314.2 TB) |
| kaonashi.txt + haku34K | 62.98% | 1h 14m 36s | 6.0 GB | 18,539,961,056,829 (204.7 TB) |
| kaonashi.txt + kamaji34K | 62.85% | 1h 14m 16s | 6.0 GB | 18,539,916,077,610 (204.7 TB) |
| hashmob.net_2025.large.found + sapphire_v3 | 59.14% | 3h 10m 01s | 555 MB | 43,466,061,885,057 (421.2 TB) |
| hashmob.net_2025.large.found + buka_400k | 57.34% | 1h 50m 31s | 555 MB | 23,174,097,156,228 (224.5 TB) |
| linuxwz.txt + Robot_CurrentBestRules | 49.03% | 1h 55m 19s | 338 MB | 27,197,985,600,920 (323.0 TB) |
| hashmob.net_2025.small.found + SuperUnicorn | 43.91% | 14m 51s | 21 MB | 3,004,407,751,516 (27.5 TB) |
| ASLM.txt + OneRuleToRuleThemAll | 41.68% | 9m 20s | 416 MB | 2,054,666,640,000 (21.8 TB) |
| ASLM.txt + _NSAKEY.v2.dive | 40.52% | 20m 56s | 416 MB | 4,870,074,666,099 (51.8 TB) |
| crackstation.txt + top_5000 | 38.57% | 26m 26s | 15.7 GB | 6,063,608,760,000 (78.5 TB) |
| ignis-1M.txt + Fordyv3-250k | 33.35% | 3m 14s | 8.8 MB | 333,375,082,978 (3.0 TB) |
| hashmob.net_2025.small.found + CakeV2 | 32.43% | 29m 50s | 21 MB | 5,642,510,848,560 (51.6 TB) |
| ASLM.txt + top_5000 | 30.03% | 2m 06s | 416 MB | 196,981,720,000 (2.1 TB) |
| ignis-1M.txt + clem9669_large | 28.28% | 22m 18s | 8.8 MB | 4,553,800,504,980 (41.1 TB) |
| breach.txt + yubaba64 | 26.64% | 1m 47s | 6.1 GB | 32,593,064,128 (387.4 GB) |
| hashcracky-main.txt + top_5000 | 10.86% | 23m 44s | 17.2 GB | 5,182,717,000,000 (85.8 TB) |
| hashmob.net_2025.small.found + best64 ×2 | 8.47% | 1m 00s | 21 MB | 14,425,494,160 (130.2 GB) |
| rockyou2024.txt + top_5000 | 2.49% | 1m 38s | 6.7 GB | 634,728,165,000 (33.5 TB) |
| passphrases.txt + top_5000 | 0.73% | 1m 27s | 541 MB | 129,827,035,000 (2.7 TB) |

</div>

Some pretty incredible results here! The hashmob wordlists are absolutely dominating, and on a real pentest these are indeed my go-to for quick wins. 89.5% recovery rate with `hashmob.net_2025.found` is just amazing!

At this point, it's worth mentioning that HIBP has since added more breaches to their archives, it's possible these numbers have gone up or down slightly 🤷‍♂

That's all for now though! Stick around for Part 2 as we delve into analyzing the passwords recovered and look at tools like PACK and OMEN to perform character and word level analysis to identify the overall shape and makeup of the passwords we cracked!

*This post also appears on [Medium](https://medium.com/@emrunning/pwning-haveibeenpwned-part-1-69e29430651e).*
